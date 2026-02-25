# JavaScript Components with Store Pattern

## Introduction
In this exercise, you will build a todo CRUD application using a **component-based architecture** with **centralized state management**. Instead of writing procedural code that directly manipulates the DOM, you'll create reusable components that automatically update when the application state changes.

### What You'll Learn
By completing this exercise, you will:
- Understand the **component pattern** for organizing UI code
- Implement a **centralized store** for state management
- Use the **observer pattern** to make components reactive
- Build reusable, self-contained UI components
- Structure a modern JavaScript application without a framework

### Why Components and Store?
Traditional DOM manipulation code becomes hard to maintain:
```js
// Traditional approach - scattered state and updates
let todos = [];
function addTodo() {
    todos.push(newTodo);
    // Manual DOM update in multiple places
    updateTable();
    updateForm();
}
```

With components and a store:
```js
// Component approach - centralized state
store.addTodo(newTodo);
// All subscribed components automatically update!
```

**Benefits:**
- **Single source of truth** - all state lives in one place
- **Automatic updates** - components subscribe to state changes
- **Reusable components** - easy to use the same component multiple times
- **Easier debugging** - track all state changes in one location
- **Testable** - components and store can be tested independently

### Application Architecture
```
┌─────────────────────────────────────────┐
│              Store                       │
│  - State (todos, isLoading, error)      │
│  - Methods (subscribe, addTodo, etc.)    │
│  - Notifies all subscribers on change    │
└──────────────┬──────────────────────────┘
               │ subscribes
       ┌───────┴────────┐
       │                │
  ┌────▼─────┐   ┌─────▼────┐
  │TodoForm  │   │TodoTable │
  │Component │   │Component │
  └──────────┘   └──────────┘
```

### Project Structure
```
project/
├── index.html
├── todo.store.js      # Centralized state management (factory function)
├── todo.api.js        # API communication
├── todo.form.js       # Form component for adding/editing todos
├── todo.table.js      # Table component for displaying todos
├── app.js             # Application initialization
└── details/
    └── index.html     # Todo details page
```

### Key Architectural Decisions
- **Factory functions** instead of classes for simplicity
- **Separate API module** for clean separation of concerns
- **Closure-based encapsulation** for private state
- **Optimistic updates** for better user experience
- **Observer pattern** for reactive updates
- **Event delegation** for efficient event handling
- **Explicit initialization** with `init()` for clear component lifecycle
- **Bootstrap** for styling instead of custom CSS
- **Form-based editing** instead of inline editing
- **Navigation to details page** for viewing todo details


## Part 1: Understanding the Store Pattern

The **store** holds all application state and notifies components when state changes.

### Core Concepts

**1. Centralized State**
All data lives in one object:
```js
const state = {
    todos: [],
    isLoading: false,
    error: null
};
```

**2. State is Read-Only (via Closure)**
Components can't directly modify state. They must call store methods:
```js
// ❌ Bad - can't do this! State is private in closure
// todos.push(newTodo);  // Not accessible!

// ✅ Good - use store methods
store.addTodo(newTodo);
```

**3. Observer Pattern**
Components subscribe to state changes:
```js
const unsubscribe = store.subscribe(render);
// This function will be called whenever state changes

// Component can unsubscribe when destroyed
store.unsubscribe(render);
// Or use the returned cleanup function
unsubscribe();
```

### Benefits of This Pattern
- **Predictable state changes** - only store methods can modify state
- **Easy debugging** - log all state changes in one place
- **Time-travel debugging** - store history of all state changes
- **Undo/Redo** - easier to implement with centralized state


## Part 2: Create the Store

Let's build the store using a factory function pattern with closures for data encapsulation.

### Step 1: Create the Store Module

Create `todo.store.js`:

```js
// todo.store.js
import { fetchTodos, createTodo, updateTodo, deleteTodo } from "./todo.api.js";

export function TodoStore() {
    // Private state - enclosed in closure
    let todos = [];
    let isLoading = false;
    let error = null;
    const subscribers = [];

    // Subscribe to state changes - returns unsubscribe function
    const subscribe = (callback) => {
        subscribers.push(callback);
        // Immediately call with current state
        callback({ todos, isLoading, error });
        
        // Return unsubscribe function
        return () => {
            const index = subscribers.indexOf(callback);
            if (index > -1) {
                subscribers.splice(index, 1);
            }
        };
    };

    // Unsubscribe from state changes
    const unsubscribe = (callback) => {
        const index = subscribers.indexOf(callback);
        if (index > -1) {
            subscribers.splice(index, 1);
        }
    };

    // Notify all subscribers of state changes
    const notify = () => {
        subscribers.forEach(callback => {
            callback({ todos, isLoading, error });
        });
    };

    // Load todos from API
    async function loadTodos(isSilent = false) {
        error = null;
        if (!isSilent) {
            isLoading = true;
            notify();
        }
        try {
            todos = await fetchTodos();
        } catch (err) {
            error = {
                message: "Failed to fetch todos. Please try again.",
                type: "FetchError"
            };
        } finally {
            isLoading = false;
        }
        notify();
    }

    // Add new todo with optimistic update
    async function addTodo(todo) {
        error = null;
        // Create temporary ID for optimistic update
        const tempId = -Date.now();
        const optimisticTodo = { ...todo, id: tempId };
        todos = [...todos, optimisticTodo];
        notify();
        
        try {
            const newTodo = await createTodo(todo);
            // Replace temp todo with real one from server
            todos = todos.map(t => t.id === tempId ? newTodo : t);
            error = null;
        } catch (err) {
            error = {
                message: "Failed to create todo. Please try again.",
                type: "CreateError"
            };
            // Rollback on error
            todos = todos.filter(t => t.id !== tempId);
        }
        notify();
    }

    // Update existing todo with optimistic update
    async function changeTodo(id, todo) {
        error = null;
        const previousTodos = [...todos];
        // Optimistically update UI
        todos = todos.map(t => 
            t.id === Number(id) ? { ...todo, id: Number(id) } : t
        );
        notify();
        
        try {
            await updateTodo(id, todo);
            error = null;
        } catch (err) {
            error = {
                message: "Failed to update todo. Please try again.",
                type: "UpdateError"
            };
            // Rollback on error
            todos = previousTodos;
        }
        notify();
    }

    // Remove todo with optimistic update
    async function removeTodo(id) {
        error = null;
        const previousTodos = [...todos];
        // Optimistically update UI
        todos = todos.filter(todo => todo.id !== Number(id));
        notify();
        
        try {
            await deleteTodo(id);
            error = null;
        } catch (err) {
            error = {
                message: "Failed to delete todo. Please try again.",
                type: "DeleteError"
            };
            // Rollback on error
            todos = previousTodos;
        }
        notify();
    }

    // Get todo by ID
    const getTodoById = (id) => {
        return todos.find(todo => todo.id === Number(id));
    };

    // Initialize store
    const init = async () => {
        await loadTodos();
    };

    // Public API
    return {
        subscribe,
        unsubscribe,
        init,
        addTodo,
        changeTodo,
        removeTodo,
        getTodoById
    };
}
```

### Step 2: Understanding the Store Code

**Factory Function Pattern**
- Returns an object with public methods
- Uses closures to keep state private
- No `new` keyword needed - just call `TodoStore()`

**Key Concepts:**

**`subscribe(callback)`**
- Adds a listener function to be called on state changes
- Immediately calls callback with current state
- **Returns an unsubscribe function** for easy cleanup
- Similar to `addEventListener` but for state

**`unsubscribe(callback)`**
- Removes a listener function from subscribers
- Pass the same callback reference used in subscribe
- Important for cleanup when components are destroyed

**`notify()`**
- Calls all subscribed listeners with current state
- Passes state as a plain object: `{ todos, isLoading, error }`
- This is what makes components reactive!

**`init()`**
- Initializes the store by loading data from API
- Called once when the application starts
- Returns a promise that resolves when data is loaded

**`getTodoById(id)`**
- Helper method to find a todo by its ID
- Used by components that need to access a specific todo
- Returns the todo object or undefined if not found

**Optimistic Updates**
- UI updates immediately (optimistic)
- If API call fails, changes are rolled back
- Provides better user experience
- Example: `addTodo` adds temp todo, then replaces with real one

**Error Handling**
- Each method catches errors and sets error state
- Errors are included in state notifications
- Components can display error messages
- Previous state is restored on failure



## Part 3: Understanding Components

A **component** is a self-contained piece of UI. Each component:
1. Has a root DOM element (container)
2. Subscribes to the store for reactive updates
3. Renders itself when state changes
4. Handles its own events
5. Returns public methods (like `destroy`)

### Component Pattern (Factory Function)

```js
export function MyComponent(container, store) {
    
    const resetContainer = () => {
        container.innerHTML = '';
    };

    const destroy = () => {
        resetContainer();
        store.unsubscribe(render);
        container.removeEventListener('click', onClick);
    };
    
    // Render function - updates the DOM
    const render = (state) => {
        resetContainer();
        container.innerHTML = `
            <div>State: ${JSON.stringify(state)}</div>
        `;
    };
    
    // Event handler using event delegation
    const onClick = (event) => {
        // Handle clicks on child elements
        const action = event.target.dataset.action;
        if (action === 'doSomething') {
            store.doSomething();
        }
    };
    
    // Initialize component
    const init = () => {
        store.subscribe(render);
        container.addEventListener('click', onClick);
    };
    
    init();
    
    // Return public API
    return {
        destroy
    };
}
```

**Key Points:**
- **Factory function** returns an object with public methods
- **Event delegation** - single listener on container instead of many on children
- **Explicit init()** - clear initialization sequence
- **store.unsubscribe(render)** - pass the render function to unsubscribe
- **Render** is called automatically by store when state changes


## Part 4: Create the API Module

Let's create a separate module for all API communication. This keeps network logic separate from state management.

Create `todo.api.js`:

```js
// todo.api.js

// export const BASE_URL = "https://jsonplaceholder.typicode.com";
export const BASE_URL = "http://localhost:8080/api";

const BASE_URL_TODOS = `${BASE_URL}/todos`;

/**
 * Fetch todos from the API
 * @returns {Promise<Array>} Array of todo objects
 */
export async function fetchTodos() {
    const response = await fetch(`${BASE_URL_TODOS}`);
    if (!response.ok) {
        throw new Error('Failed to fetch todos');
    }
    return response.json();
}

/**
 * Create a new todo
 * @param {Object} todo - Todo object to create
 * @returns {Promise<Object>} Created todo with id from server
 */
export async function createTodo(todo) {
    const response = await fetch(`${BASE_URL_TODOS}`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(todo),
    });
    if (!response.ok) {
        throw new Error('Failed to create todo');
    }
    return response.json();
}

/**
 * Update an existing todo
 * @param {number} id - Todo ID
 * @param {Object} todo - Todo object with updated fields
 * @returns {Promise<Object>} Updated todo
 */
export async function updateTodo(id, todo) {
    const response = await fetch(`${BASE_URL_TODOS}/${id}`, {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(todo),
    });
    if (!response.ok) {
        throw new Error('Failed to update todo');
    }
    return response.json();
}

/**
 * Delete a todo
 * @param {number} id - Todo ID to delete
 * @returns {Promise<void>}
 */
export async function deleteTodo(id) {
    const response = await fetch(`${BASE_URL_TODOS}/${id}`, {
        method: 'DELETE',
    });
    if (!response.ok) {
        throw new Error('Failed to delete todo');
    }
}
```

### Understanding the API Module

**Base URL Configuration**
- Primary: `http://localhost:8080/api` for local backend
- Fallback: JSONPlaceholder (commented) for testing without backend
- Easy to switch between local and remote APIs

**Named Exports**
- Each function is exported individually
- Import with: `import { fetchTodos, createTodo } from './todo.api.js'`
- Clear, descriptive function names

**Why Separate Functions?**
- Easy to import only what you need
- Simple to test individual functions
- No object wrapper needed
- Follows functional programming principles

**Key Points:**
- All API logic is in one place (separation of concerns)
- Each function is async and returns a Promise
- Errors are thrown and handled by the store
- Uses RESTful conventions (GET, POST, PUT, DELETE)


## Part 5: Build the TodoForm Component

This component handles both adding new todos and editing existing ones. It uses the same form for both operations.

Create `todo.form.js`:

```js
// todo.form.js
export function TodoForm(element, store) {
    
    const resetForm = () => {
        element.reset();
        element.querySelector("input[name='id']").value = "";
        document.querySelector("#cancelEditBtn").classList.add("hidden");
        document.querySelector("[type='submit']").textContent = "Add Todo";
    };

    const destroy = () => {
        element.removeEventListener("submit", onSubmit);
        element.querySelector("#cancelEditBtn").removeEventListener("click", resetForm);
        store.unsubscribe(handleStoreChange);
    };

    const onSubmit = async (event) => {
        event.preventDefault();
        
        const formData = new FormData(element);
        const title = formData.get("title");
        const userId = formData.get("userId");
        const completed = formData.get("completed") === "on";
        const id = formData.get("id");

        const todo = {
            title,
            userId: Number(userId),
            completed
        };

        if (id) {
            // Edit mode - update existing todo
            todo.id = Number(id);
            store.changeTodo(id, todo);
        } else {
            // Create mode - add new todo
            store.addTodo(todo);
        }
        
        resetForm();
    };

    const setLoading = (loading) => {
        const submitBtn = element.querySelector("[type='submit']");
        submitBtn.disabled = loading;
    };

    const handleStoreChange = (data) => {
        setLoading(data.isLoading);
        
        if (data.error && (data.error.type === "CreateError" || data.error.type === "UpdateError")) {
            alert(`Error: ${data.error?.message}`);
            const submitBtn = element.querySelector("[type='submit']");
            submitBtn.disabled = false;
        }
    };

    const fillForm = (todo) => {
        element.querySelector("input[name='title']").value = todo.title;
        element.querySelector("input[name='userId']").value = todo.userId;
        element.querySelector("input[name='completed']").checked = todo.completed;
        element.querySelector("input[name='id']").value = todo.id;

        document.querySelector("#cancelEditBtn").classList.remove("hidden");
        document.querySelector("[type='submit']").textContent = "Update Todo";
    };

    const init = () => {
        element.addEventListener("submit", onSubmit);
        element.querySelector("#cancelEditBtn").addEventListener("click", resetForm);
        store.subscribe(handleStoreChange);
    };

    init();

    return {
        destroy,
        fillForm
    };
}
```

### Understanding TodoForm

**Dual Mode Form**
- **Create mode**: Default state, adds new todos
- **Edit mode**: Triggered by `fillForm()`, updates existing todo
- Hidden ID field tracks which mode we're in
- Cancel button appears in edit mode

**Key Features:**

**`resetForm()`**
- Clears all form fields
- Switches back to create mode
- Hides cancel button
- Called after successful submission or cancel

**`fillForm(todo)`**
- Public method exposed to other components
- Populates form with todo data
- Switches to edit mode
- Shows cancel button and changes submit text

**`onSubmit(event)`**
- Handles both create and update operations
- Checks if ID exists to determine mode
- Calls appropriate store method
- Resets form on success

**`handleStoreChange(data)`**
- Subscribes to store updates
- Disables submit button during loading
- Shows error alerts if create/update fails
- Component reacts to store state

**Event Delegation Pattern**
- Single submit listener on form element
- Single click listener on cancel button
- Listeners removed in destroy for proper cleanup

**Why This Pattern?**
- Single form for both create and update (DRY)
- No separate edit modal needed
- Clear visual feedback (button text changes)
- Easy to cancel editing
- Store handles all API calls and loading states

**Key Points:**
- Form doesn't display store data (todos list)
- Only sends data TO the store and reacts to loading/errors
- `fillForm` is called by table component when edit button clicked
- Clean separation: form handles UI, store handles data


## Part 6: Build the TodoTable Component

This component displays todos in a table and handles editing and deletion. Clicking on a row navigates to the details page.

Create `todo.table.js`:

```js
// todo.table.js
export function TodoTable(element, store, onEdit) {

    const resetTable = () => {
        element.innerHTML = "";
    };

    const destroy = () => {
        resetTable();
        store.unsubscribe(render);
        element.removeEventListener("click", onClick);
        element.removeEventListener("change", onChange);
    };

    const render = ({ todos, isLoading, error }) => {
        resetTable();
        
        if (error && error.type === "FetchError") {
            element.innerHTML = `<tr><td colspan="4">${error.message}</td></tr>`;
            return;
        }

        if (error && error.type === "DeleteError") {
            alert(`Error: ${error.message}`);
        }

        if (isLoading) {
            element.innerHTML = `<tr><td colspan="4">Loading...</td></tr>`;
            return;
        }
        
        todos.forEach(todo => {
            const tr = document.createElement("tr");
            tr.setAttribute("data-id", todo.id);
            tr.innerHTML = /*html*/ `
                <td>${todo.id}</td>
                <td>${todo.title}</td>
                <td>${todo.userId}</td>
                <td>
                    <input 
                        type="checkbox" 
                        ${todo.completed ? 'checked' : ''}
                        data-action="toggle"
                        class="completed-checkbox"
                    >
                </td>
                <td>
                    <div class="gap-2 flex">
                        <button data-action="edit" class="btn btn-warning">Edit</button>
                        <button data-action="delete" class="btn btn-danger">Delete</button>
                    </div>
                </td>
            `;
            element.appendChild(tr);
        });
    };

    const onClick = async (event) => {
        const id = event.target.closest("tr")?.dataset.id;
        if (id === undefined) return;

        if (event.target.getAttribute("data-action") === "delete") {
            if (!confirm('Are you sure you want to delete this todo?')) {
                return;
            }
            await store.removeTodo(id);
            return;
        }

        if (event.target.getAttribute("data-action") === "edit") {
            const todo = store.getTodoById(id);
            onEdit(todo);
            return;
        }
        
        // Navigate to details page
        window.location.href = `details/?id=${id}`;
    };

    const onChange = async (event) => {
        if (event.target.getAttribute("data-action") === "toggle") {
            const row = event.target.closest("tr");
            if (!row || !row.dataset.id) return;
            const id = parseInt(row.dataset.id);
            const completed = event.target.checked;
            
            // Get current todo and update with all fields
            const todo = store.getTodoById(id);
            await store.changeTodo(id, { ...todo, completed });
        }
    };

    const init = () => {
        store.subscribe(render);
        element.addEventListener("click", onClick);
        element.addEventListener("change", onChange);
    }

    init();

    return {
        destroy
    };
}
```

### Understanding TodoTable

**Callback Pattern**
- Accepts `onEdit` callback as third parameter
- Called when edit button is clicked
- Passes todo object to the callback
- This allows form component to fill itself with todo data

**Reactive Component**
- Subscribes to store on initialization
- Automatically re-renders when state changes
- All state comes from the store

**Event Delegation Pattern**
- Single `click` listener on container instead of many on individual buttons
- Single `change` listener for checkboxes
- Much more efficient - no need to re-attach listeners after each render
- Listeners check `event.target` to determine which element was clicked

**Three Click Actions:**
1. **Edit button**: Calls `onEdit(todo)` to fill form
2. **Delete button**: Confirms and calls `store.removeTodo()`
3. **Row click**: Navigates to `details/?id={id}` page

**Navigation**
- Clicking anywhere on row (except buttons) navigates to details
- Uses query parameter to pass todo ID
- Simple page navigation with `window.location.href`

**Checkbox Toggle**
- Updates todo's completed status
- Gets full todo object and spreads it to preserve all fields
- Store handles optimistic update

**Factory Function Benefits**
- All functions in closure scope
- No `this` binding issues
- Clean event handler references
- Uses `store.unsubscribe(render)` in destroy

**Key Improvements**
- `resetTable()` helper for cleaning up container
- Explicit `init()` for clear initialization
- Event delegation reduces memory usage and improves performance
- No need to re-attach listeners after render
- Form-based editing instead of inline editing (simpler, clearer UX)


## Part 7: Initialize the Application

Create `app.js` to wire everything together:

```js
// app.js
import { TodoForm } from "./todo.form.js";
import { TodoStore } from "./todo.store.js";
import { TodoTable } from "./todo.table.js";

document.addEventListener("DOMContentLoaded", () => {
    const form = document.querySelector("#todoForm");
    const table = document.querySelector("#todoTableBody");
    const todoStore = TodoStore();

    const todoForm = TodoForm(form, todoStore);
    const todoTable = TodoTable(table, todoStore, (todo) => todoForm.fillForm(todo));

    todoStore.init();
});
```

### Understanding the App Module

**DOMContentLoaded Event**
- Waits for HTML to be fully loaded before running
- Ensures all DOM elements exist before querying them
- Standard pattern for vanilla JS apps

**Initialization Flow**
1. Query form and table body elements from DOM
2. Create store instance with `TodoStore()`
3. Create form component, passing form element and store
4. Create table component, passing:
   - Table body element
   - Store
   - **Callback function** `(todo) => todoForm.fillForm(todo)`
5. Call `todoStore.init()` to load data from API

**Callback Pattern**
- Table receives `todoForm.fillForm` as `onEdit` callback
- When edit button clicked, table calls: `onEdit(todo)`
- This calls `todoForm.fillForm(todo)`, populating the form
- Clean component communication without tight coupling

**Component Lifecycle**
- Each component's `init()` runs automatically in constructor
- Components subscribe to store in their `init()`
- Store's `init()` loads data, triggering all component renders

**Why This Pattern?**
- Simpler than factory function wrapping
- Direct, straightforward initialization
- Easy to understand flow
- Store passed explicitly (dependency injection)
- Components stay decoupled via callback


## Part 8: HTML Structure

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
    <style>
        body {
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        form {
            padding: 20px;
            border: 1px solid #ccc;
            width: 400px;
            display: flex;
            flex-direction: column;
            gap: 1em;
            margin-bottom: 2rem;
        }

        .hidden {
            display: none;
        }

        tbody tr {
            cursor: pointer;
        }

        tbody tr td button {
            pointer-events: all;
        }
    </style>
    <title>Todo CRUD with Components & Store</title>
</head>

<body>
    <h1 class="text-center">Todo CRUD</h1>
    
    <form id="todoForm">
        <input type="hidden" id="todoId" name="id">
        <input type="text" id="todoTitle" name="title" placeholder="Enter todo title" required>
        <input type="number" id="userId" name="userId" placeholder="Enter user ID" required>
        <div class="form-check">
            <input type="checkbox" id="todoCompleted" name="completed" class="form-check-input">
            <label class="form-check-label" for="todoCompleted">Completed</label>
        </div>
        <button type="button" id="cancelEditBtn" class="btn btn-secondary hidden">Cancel Edit</button>
        <button type="submit" class="btn btn-primary">Add Todo</button>
    </form>

    <table class="table">
        <thead>
            <tr>
                <th scope="col">ID</th>
                <th scope="col">Title</th>
                <th scope="col">User ID</th>
                <th scope="col">Completed</th>
                <th scope="col">Actions</th>
            </tr>
        </thead>
        <tbody id="todoTableBody">
        </tbody>
    </table>

    <script type="module" src="./app.js"></script>
</body>

</html>
```
<!-- 
### Understanding the HTML

**Bootstrap Integration**
- Uses Bootstrap 5 CDN for styling
- Bootstrap classes: `table`, `btn`, `btn-primary`, `btn-warning`, `btn-danger`
- Minimal custom CSS needed

**Form Structure**
- **Hidden ID field**: Tracks whether we're in create or edit mode
- **Title input**: Text field for todo title
- **UserID input**: Number field for user ID
- **Completed checkbox**: Boolean field for completion status
- **Cancel button**: Hidden by default, shows during edit mode
- **Submit button**: Text changes between "Add Todo" and "Update Todo"

**Table Structure**
- Simple Bootstrap table
- Headers: ID, Title, User ID, Completed, Actions
- Empty tbody - populated by TodoTable component
- **Important**: Pass `tbody` element to component, not the whole table

**Custom Styles**
- `.hidden`: Utility class to hide cancel button
- `tbody tr`: Clickable rows for navigation
- Button `pointer-events`: Ensures buttons work despite row click handler

**Module Script**
- `type="module"` enables ES6 imports
- Loads `app.js` which initializes everything


## Part 9: Create the Details Page

When a user clicks on a todo row, they're navigated to a details page that displays full todo information.

Create `details/index.html`:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
    <title>Todo Details</title>
    <style>
        body {
            padding: 20px;
        }
        .card {
            max-width: 600px;
            margin: 2rem auto;
        }
    </style>
</head>

<body>
    <div class="container">
        <h1 class="text-center">Todo Details</h1>
        <div id="content"></div>
        <div class="text-center mt-4">
            <a href="../" class="btn btn-secondary">← Back to List</a>
        </div>
    </div>

    <script type="module">
        import { TodoStore } from "../todo.store.js";
        
        // Get todo ID from URL query parameter
        const todoId = new URLSearchParams(window.location.search).get("id");
        
        const todoStore = TodoStore();
        const content = document.getElementById("content");
        
        todoStore.subscribe(({ todos, isLoading, error }) => {
            if (error) {
                content.innerHTML = `
                    <div class="alert alert-danger" role="alert">
                        <h4>Error</h4>
                        <p>${error.message}</p>
                    </div>
                `;
                return;
            }
            
            if (isLoading) {
                content.innerHTML = `
                    <div class="text-center">
                        <div class="spinner-border" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                    </div>
                `;
                return;
            }

            const todo = todos.find(t => t.id === parseInt(todoId));
            
            if (todo) {
                content.innerHTML = `
                    <div class="card">
                        <div class="card-header">
                            <h2>Todo #${todo.id}</h2>
                        </div>
                        <div class="card-body">
                            <h5 class="card-title">${todo.title}</h5>
                            <p class="card-text">
                                <strong>User ID:</strong> ${todo.userId}<br>
                                <strong>Status:</strong> 
                                <span class="badge ${todo.completed ? 'bg-success' : 'bg-warning'}">
                                    ${todo.completed ? 'Completed' : 'Pending'}
                                </span>
                            </p>
                        </div>
                    </div>
                `;
            } else {
                content.innerHTML = `
                    <div class="alert alert-warning" role="alert">
                        Todo not found
                    </div>
                `;
            }
        });
        
        todoStore.init();
    </script>

</body>

</html>
```

### Understanding the Details Page

**URL Query Parameters**
- Uses `URLSearchParams` to parse `?id=123` from URL
- Extracts the todo ID to look up
- Standard web pattern for passing data in URLs

**Store Reuse**
- Imports and uses the same `TodoStore`
- Subscribes to get todos data
- Benefits from the same optimistic updates and error handling

**Inline Script Module**
- Small page with simple logic, no separate file needed
- Uses `type="module"` to import store
- All logic contained in the HTML file

**Reactive Updates**
- Subscribes to store changes
- Shows loading spinner while fetching
- Displays error if fetch fails
- Finds and displays the specific todo

**Bootstrap Components**
- Card for displaying todo details
- Badge for status (completed/pending)
- Alert for errors and "not found" message
- Spinner for loading state

**Navigation**
- Back button uses relative path `../` to return to main page
- Simple, clean navigation pattern

**Why This Pattern?**
- Simple details page doesn't need full component architecture
- Store provides all the data management
- Inline script keeps it simple for a single-use page
- Easy to enhance later if needed


## Part 10: Styling (Optional Custom CSS)

The application uses Bootstrap for most styling, but you can add custom styles if desired. The inline styles in `index.html` and `details/index.html` provide basic layout.

If you want additional custom styling, create `styles.css`:

```css
/* Optional custom styles - Bootstrap handles most styling */

body {
    background-color: #f8f9fa;
}

form {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.table {
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

tbody tr:hover {
    background-color: #f8f9fa;
}

.hidden {
    display: none !important;
}

.gap-2 {
    gap: 0.5rem;
}

.flex {
    display: flex;
}
```

### Styling Notes

**Bootstrap First**
- Use Bootstrap classes for buttons, forms, tables
- No need for extensive custom CSS
- Consistent, professional look out of the box

**Minimal Custom CSS**
- Only add custom styles for specific needs
- `.hidden` class for toggling visibility
- `.gap-2` and `.flex` for button layouts
- Light background and shadows for depth

**Inline Styles**
- Small style blocks in HTML `<style>` tags
- Acceptable for simple, page-specific styles
- Keeps styling close to the HTML

**When to Add More CSS**
- Custom branding/colors
- Specific layout requirements
- Animations/transitions
- Responsive tweaks beyond Bootstrap


## Part 11: Testing Your Application

1. Open `index.html` in your browser (use Live Server or similar)
2. Open the browser console (F12)
3. Try these interactions:

**Test Form - Create Mode:**
- Enter a todo title, user ID
- Check/uncheck the completed checkbox
- Click "Add Todo"
- Watch the table update automatically
- Form should reset after submission

**Test Form - Edit Mode:**
- Click "Edit" button on any todo row
- Form should populate with todo data
- Cancel button should appear
- Submit button should say "Update Todo"
- Modify the data and submit
- Watch the table update
- Click "Cancel Edit" to return to create mode

**Test Table:**
- Click delete button on a todo
- Confirm deletion dialog
- Watch todo disappear from table
- Toggle completed checkbox
- Watch checkbox state persist

**Test Navigation:**
- Click anywhere on a todo row (not on buttons)
- Should navigate to `details/?id={id}` page
- Details page should display full todo information
- Click "Back to List" to return

**Test Error Handling:**
- Stop your backend server (if using local API)
- Try to add, edit, or delete a todo
- Should see error alerts
- Original state should be preserved (rollback)

**Debug with Console:**
```js
// The store isn't exposed globally in this version
// But you can add console.logs in your code to debug
```


## Part 12: Understanding the Benefits

### 1. Automatic Updates
When you add a todo:
```js
store.addTodo(newTodo);
```

This automatically:
1. Updates the store's state
2. Notifies all subscribed components
3. TodoTable re-renders with new todo
4. Form resets and stays in create mode

No manual DOM manipulation needed!

### 2. Single Source of Truth
```js
// State lives in the store
// All components get data from the same source
store.subscribe(({ todos, isLoading, error }) => {
    console.log(`Total todos: ${todos.length}`);
});
```

### 3. Event Delegation Benefits
Instead of hundreds of event listeners:
```js
// ❌ Bad - re-attaching listeners after each render
todos.forEach(todo => {
    const btn = document.querySelector(`#todo-${todo.id}`);
    btn.addEventListener('click', handleDelete);
});

// ✅ Good - single listener using delegation
container.addEventListener('click', onClick);
```

**Why Event Delegation is Better:**
- **Performance** - fewer event listeners = less memory
- **No re-attachment** - listeners survive DOM updates
- **Dynamic content** - works for elements added later
- **Simpler cleanup** - only remove container listener

### 4. Component Communication via Callbacks
Table and Form communicate without tight coupling:
```js
const todoForm = TodoForm(form, todoStore);
const todoTable = TodoTable(table, todoStore, (todo) => todoForm.fillForm(todo));
```

- Table doesn't know about form implementation
- Form exposes public `fillForm` method
- Clean, testable interface

### 5. Form-Based Editing Pattern
- Single form for create and update
- Clear visual feedback (button text, cancel button)
- No complex inline editing logic
- Better UX on mobile devices

### 6. Easy Testing
```js
// Test store in isolation
const testStore = TodoStore();
await testStore.addTodo({ title: 'Test', userId: 1, completed: false });
// Check that todo was added via subscription

// Test component with mock store
const mockStore = TodoStore();
const container = document.createElement('tbody');
const component = TodoTable(container, mockStore, () => {});
// Component automatically renders with mockStore's data
```


## Part 13: Challenges

### Challenge 1: Add Stats Component
Create a `todo.stats.js` component that:
- Displays total, completed, and pending count
- Shows completion percentage
- Updates automatically when todos change
- Place it above the form

**Hint:** Subscribe to store and calculate stats from todos array.

### Challenge 2: Add Bulk Actions
Add functionality to:
- "Mark all as complete" button
- "Mark all as pending" button
- "Delete completed" button

**Hint:** Add methods to store that loop through todos.

### Challenge 3: Persist State to LocalStorage
Save todos to localStorage:
- Save state on every change
- Load state on app init if available
- Add "Clear All Data" button

**Hint:** Subscribe to store and call `localStorage.setItem()`.

### Challenge 4: Enhance Details Page
Improve `details/index.html` to:
- Show formatted creation date (you'll need to add a timestamp)
- Add comments section for each todo
- Allow editing todo from details page

**Hint:** You can reuse the form logic.

### Challenge 5: Add Search/Filter
Create a search component that:
- Has a search input above the table
- Filters todos by title as you type
- Add filter buttons: All, Completed, Pending

**Hint:** Add `searchTerm` and `filter` to store state.

### Challenge 6: Add Undo/Redo
Enhance the store to:
- Keep a history of the last 10 state changes
- Add `undo()` and `redo()` methods
- Add undo/redo buttons in UI with keyboard shortcuts

**Hint:** Store array of previous states, track current index.


## Part 14: Comparison with Frameworks

What you've built is similar to:
- **React**: Components + Redux/Context
- **Vue**: Components + Vuex/Pinia
- **Angular**: Components + Services
- **Svelte**: Components + Stores

**Key Similarities:**
- Component-based architecture
- Centralized state management
- Reactive updates
- Unidirectional data flow
- Event delegation patterns

**What frameworks add:**
- Virtual DOM (efficient re-rendering)
- JSX/Templates (declarative UI)
- Built-in routing
- Dev tools and debugging
- Larger ecosystem and libraries
- TypeScript integration
- Build optimization

**Your vanilla JS solution is perfect for:**
- Learning core concepts deeply
- Small to medium projects
- No build step needed
- Full control over code
- Understanding what frameworks do under the hood

**When to use a framework:**
- Large, complex applications
- Team collaboration with established patterns
- Need for extensive third-party integrations
- Advanced features like SSR, code splitting
- Long-term maintenance with many developers


## Conclusion

You've built a modern, maintainable todo application using:
- ✅ Component-based architecture
- ✅ Centralized store pattern
- ✅ Observer pattern for reactivity
- ✅ Separation of concerns (API, Store, Components)
- ✅ Modular, reusable code
- ✅ Form-based editing with dual mode
- ✅ Navigation between pages
- ✅ Bootstrap for professional styling
- ✅ Optimistic updates with error rollback
- ✅ Event delegation for performance

This architecture scales well and prepares you for modern frameworks!

### Next Steps
1. Complete the challenges above
2. Add your own features (authentication, due dates, priorities)
3. Connect to a real backend API
4. Try building other apps with this pattern (blog, notes, shopping list)
5. Learn a framework (React, Vue, or Angular) and notice the similarities! -->
