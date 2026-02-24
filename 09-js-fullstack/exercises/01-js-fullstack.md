# JavaScript - Table Sorting Exercises

## Introduction
In this exercise, you will build on top of the previous exercise where you built a CRUD application that fetches data from the JSONPlaceholder API. You will implement **client-side sorting** functionality for the todo table. When you click on a table header (Title or User ID), the table will sort by that column, and an indicator (▲ or ▼) will show the current sort direction.


### What You'll Build
Starting from your CRUD app, you'll add:
1. Clickable table headers for Title and User ID columns
2. A reusable `sortBy` function that handles both string and number comparisons
3. State management to track which column is sorted and in which direction
4. Visual indicators (▲ ▼) showing the active sort column and direction
5. Persistent sort state that maintains after CRUD operations

## Exercise 1: Update the HTML table headers

First, we need to make the table headers interactive and prepare them for displaying sort indicators.

1. Open your `index.html` file and locate the `<thead>` section of your table.

2. Add `data-sort-key` attributes to the Title and User ID headers. These attributes will identify which property to sort by:

```html
<thead>
    <tr>
        <th data-sort-key="title" style="cursor: pointer;">Title <span id="sort-title"></span></th>
        <th data-sort-key="userId" style="cursor: pointer;">User ID <span id="sort-userId"></span></th>
        <th>Completed</th>
        <th>Actions</th>
    </tr>
</thead>
```

**Explanation:**
- `data-sort-key`: Custom data attribute that stores the property name to sort by
- `<span id="sort-title"></span>`: Empty span where we'll display the sort indicator (▲ or ▼)

## Exercise 2: Build the sortBy function

The `sortBy` function is a **higher-order function** that returns a comparator function. This comparator can be used with JavaScript's `Array.sort()` method to sort an array of objects by a specific property.

### Step 1: Understand the function signature

Add this function skeleton to your `app.js` file:

```js
function sortBy(key, isAsc) {
    // This function returns another function (a comparator)
    return (a, b) => {
        // Comparator logic will go here
    };
}
```

**Parameters:**
- `key`: The property name to sort by (e.g., "title", "userId")
- `isAsc`: Boolean indicating ascending (true) or descending (false) order

**Returns:**
- A comparator function that can be passed to `Array.sort()`

### Step 2: Extract the values to compare

Inside the comparator function, extract the values from both objects:

```js
function sortBy(key, isAsc) {
    return (a, b) => {
        const valA = a[key];
        const valB = b[key];
        
        // More logic will go here
    };
}
```

### Step 3: Handle undefined values

If either value is undefined, we can't compare them meaningfully, so return 0 (no change in order):

```js
function sortBy(key, isAsc) {
    return (a, b) => {
        const valA = a[key];
        const valB = b[key];

        if (valA === undefined || valB === undefined) {
            return 0;
        }
        
        // More logic will go here
    };
}
```

### Step 4: Implement string comparison

For strings, use `localeCompare()` which handles alphabetical sorting correctly (including special characters and different locales):

```js
function sortBy(key, isAsc) {
    return (a, b) => {
        const valA = a[key];
        const valB = b[key];

        if (valA === undefined || valB === undefined) {
            return 0;
        }

        // Sort by string
        if (typeof valA === "string" && typeof valB === "string") {
            return isAsc ? valA.localeCompare(valB) : valB.localeCompare(valA);
        }
        
        // More logic will go here
    };
}
```

**Explanation:**
- `valA.localeCompare(valB)` returns negative if `valA` comes before `valB`, positive if after, 0 if equal
- When `isAsc` is true, we compare `valA` to `valB` (ascending order)
- When `isAsc` is false, we compare `valB` to `valA` (descending order)

### Step 5: Implement numeric comparison

For numbers, subtract one from the other. The difference determines the sort order:

```js
function sortBy(key, isAsc) {
    return (a, b) => {
        const valA = a[key];
        const valB = b[key];

        if (valA === undefined || valB === undefined) {
            return 0;
        }

        // Sort by string
        if (typeof valA === "string" && typeof valB === "string") {
            return isAsc ? valA.localeCompare(valB) : valB.localeCompare(valA);
        }

        // Sort by number
        if (typeof valA === "number" && typeof valB === "number") {
            return isAsc ? valA - valB : valB - valA;
        }
        
        // More logic will go here
    };
}
```

**Explanation:**
- If `valA - valB` is negative, `valA` is smaller and should come first (ascending)
- If positive, `valB` is smaller and should come first
- When `isAsc` is false, we reverse the subtraction order

### Step 6: Add default return

If neither string nor number comparison applies, return 0 (no change):

```js
function sortBy(key, isAsc) {
    return (a, b) => {
        const valA = a[key];
        const valB = b[key];

        if (valA === undefined || valB === undefined) {
            return 0;
        }

        // Sort by string
        if (typeof valA === "string" && typeof valB === "string") {
            return isAsc ? valA.localeCompare(valB) : valB.localeCompare(valA);
        }

        // Sort by number
        if (typeof valA === "number" && typeof valB === "number") {
            return isAsc ? valA - valB : valB - valA;
        }

        return 0;
    };
}
```

### Complete sortBy function

Here's the complete function for reference:

```js
function sortBy(key, isAsc) {
    return (a, b) => {
        const valA = a[key];
        const valB = b[key];

        if (valA === undefined || valB === undefined) {
            return 0;
        }

        // Sort by string
        if (typeof valA === "string" && typeof valB === "string") {
            return isAsc ? valA.localeCompare(valB) : valB.localeCompare(valA);
        }

        // Sort by number
        if (typeof valA === "number" && typeof valB === "number") {
            return isAsc ? valA - valB : valB - valA;
        }

        return 0;
    };
}
```

## Exercise 3: Track sort state

To remember which column is currently sorted and in which direction, we need to maintain state.

1. Add a `sortState` object at the top of your `app.js` file (before the `initApp` function):

```js
// Sort state management
let sortState = {
    key: "title",      // Default sort by title
    isAsc: true        // Default ascending order
};

// Global todos array to avoid refetching
let todosData = [];
```

**Explanation:**
- `sortState`: Stores the current sort column (`key`) and direction (`isAsc`)
- `todosData`: We'll store all todos here so we can re-sort without refetching from the API
- Default values: Sort by title in ascending order when the app loads

## Exercise 4: Add click handlers to table headers

Now we'll create a function that handles clicks on the sortable table headers.

1. Create a `handleHeaderClick` function:

```js
function handleHeaderClick(event) {
    const clickedHeader = event.target.closest("th");
    
    // Check if the clicked element is a sortable header
    if (!clickedHeader || !clickedHeader.hasAttribute("data-sort-key")) {
        return; // Not a sortable header, do nothing
    }

    const clickedKey = clickedHeader.getAttribute("data-sort-key");
    
    // If clicking the same column, toggle sort direction
    if (sortState.key === clickedKey) {
        sortState.isAsc = !sortState.isAsc;
    } else {
        // If clicking a different column, sort by that column in ascending order
        sortState.key = clickedKey;
        sortState.isAsc = true;
    }
    
    // Sort and display the todos with the new sort state
    sortAndDisplayTodos();
}
```

**Explanation:**
- `event.target.closest("th")`: Finds the closest `<th>` ancestor (handles clicks on the th or span inside)
- Check for `data-sort-key` attribute to ensure it's a sortable column
- Toggle `isAsc` if clicking the same column again
- Reset to ascending when clicking a different column
- Call `sortAndDisplayTodos()` to apply the sort (we'll create this next)

## Exercise 5: Sort and re-render the table

Create a function that sorts the todos array and re-displays the table:

```js
function sortAndDisplayTodos() {
    // Sort the todosData array using our sortBy function
    todosData.sort(sortBy(sortState.key, sortState.isAsc));
    
    // Re-display the sorted todos
    displayTodos(todosData);
    
    // Update the visual indicators
    updateSortIndicators();
}
```

**Explanation:**
- `todosData.sort()`: Sorts the array in-place using our custom comparator
- `displayTodos(todosData)`: Re-renders the table with the sorted data
- `updateSortIndicators()`: Updates the arrow indicators (we'll implement this next)

## Exercise 6: Visual indicators for sort direction

Create a function to show which column is sorted and in which direction:

```js
function updateSortIndicators() {
    // Clear all existing indicators
    document.querySelectorAll("th[data-sort-key] span").forEach(span => {
        span.textContent = "";
    });
    
    // Add indicator to the active sort column
    const activeIndicator = document.querySelector(`#sort-${sortState.key}`);
    if (activeIndicator) {
        activeIndicator.textContent = sortState.isAsc ? " ▲" : " ▼";
    }
}
```

**Explanation:**
- First, clear all span elements inside sortable headers
- Then, add ▲ (ascending) or ▼ (descending) to the active column's span
- The `#sort-${sortState.key}` selector targets the correct span (e.g., `#sort-title`)

## Exercise 7: Initialize sorting functionality

Update your `initApp` function to set up event listeners and store the initial todos data:

1. Modify your `fetchTodos` function to also store the data globally (if you're not already doing this):

```js
async function initApp() {
    // Fetch and display todos
    todosData = await fetchTodos();
    displayTodos(todosData);
    
    // Add event listener for form submission
    document.querySelector("#todoForm").addEventListener("submit", handleFormSubmit);
    
    // Add event listener for table clicks (delete, edit)
    document.querySelector("#todoTableBody").addEventListener("click", handleTableClick);
    
    // Add event listener for table header clicks (sorting)
    document.querySelector("thead").addEventListener("click", handleHeaderClick);
    
    // Initialize sort indicators
    updateSortIndicators();
}
```

**Explanation:**
- Store fetched todos in `todosData` for later sorting
- Add event listener to `<thead>` for header clicks (event delegation)
- Call `updateSortIndicators()` to show the default sort indicator (▲ on Title)

## Exercise 8: Maintain sort after CRUD operations

To ensure the sort persists after adding, updating, or deleting todos, update your CRUD functions:

### After adding a new todo:

```js
async function handleFormSubmit(event) {
    event.preventDefault();
    const form = new FormData(event.target);
    const id = form.get("id");
    const title = form.get("title");
    const userId = Number(form.get("userId"));
    const completed = form.get("completed") === "on";
    
    const todoData = { title, userId, completed };

    if (id) {
        // Update existing todo
        const updatedTodo = await updateTodo(id, todoData);
        // Find and update the todo in todosData
        const index = todosData.findIndex(todo => todo.id === Number(id));
        if (index !== -1) {
            todosData[index] = updatedTodo;
        }
    } else {
        // Add new todo
        const createdTodo = await addTodo(todoData);
        // Add to todosData
        todosData.push(createdTodo);
    }
    
    // Re-sort and display with current sort settings
    sortAndDisplayTodos();
    
    event.target.reset();
    document.querySelector("#todoId").value = "";
}
```

### After deleting a todo:

```js
async function handleTableClick(event) {
    const action = event.target.getAttribute("data-action");
    const row = event.target.closest("tr");
    const id = Number(row.getAttribute("data-id"));
    
    if (action === "delete") {
        await deleteTodo(id);
        // Remove from todosData
        todosData = todosData.filter(todo => todo.id !== id);
        // Re-sort and display
        sortAndDisplayTodos();
    } else if (action === "edit") {
        // Populate form with existing todo data
        const title = row.children[0].textContent;
        const userId = row.children[1].textContent;
        const completed = row.children[2].textContent === "Yes";

        document.querySelector("#todoId").value = id;
        document.querySelector("#todoTitle").value = title;
        document.querySelector("#userId").value = userId;
        document.querySelector("#completed").checked = completed;
    }
}
```

## Exercise 9: Add cancel to edit functionality
To allow users to cancel editing a todo, we can add a "Cancel" button to the form. When clicked, it will reset the form and clear the hidden `id` field.
1. Update your `index.html` form to include a Cancel button:

```html
<form id="todoForm">
    <!-- INPUT FIELDS ... -->
     <button type="button" id="cancelEditBtn" class="btn btn-secondary hidden">Cancel Edit</button>
    <button type="submit" class="btn btn-primary">Save</button>
</form>
```

2. Add some CSS to hide the Cancel button by default:

```css
.hidden {
    display: none;
}
```

3. Make the Cancel button visible when editing:

```js
async function handleTableClick(event) {
    // ... existing code ...
    
    if (action === "delete") {
        // ... existing delete logic ...
    } else if (action === "edit") {
        // ... existing edit logic ...
        // Show the Cancel button
        document.querySelector("#cancelEditBtn").classList.remove("hidden");
    }
}
```

4. Add an event listener inside the `initApp` function for the Cancel button to reset the form and hide itself:

```js
document.querySelector("#cancelEditBtn").addEventListener("click", () => {
    document.querySelector("#todoForm").reset();
    document.querySelector("#todoId").value = "";
    document.querySelector("#cancelEditBtn").classList.add("hidden");
});
```

5. When the form is submitted (either adding or updating), hide the Cancel button again:

```js
async function handleFormSubmit(event) {
    // ... existing code ...

    // Hide the Cancel button after submission
    document.querySelector("#cancelEditBtn").classList.add("hidden");
}
```

6. Now, when you click "Edit" on a todo, the form will populate with the existing data and show the Cancel button. If you click "Cancel", it will reset the form and hide the Cancel button, allowing you to exit edit mode without making changes.


## Exercise 10: Implement the backend API to replace JSONPlaceholder
In the next exercise, you will replace the JSONPlaceholder API with your own Spring Boot backend. This will allow you to have full control over the data and implement additional features as needed. You will create RESTful endpoints for managing todos and connect your frontend to this new backend.

You will need to:
1. Set up a Spring Boot application with the necessary dependencies (Spring Web, Spring Data JPA, H2 Database)
2. Create a `Todo` entity class that maps to a database table
3. Implement a `TodoRepository` interface for database operations
4. Create a `TodoService` class to handle business logic
4. Create a `TodoController` class with REST endpoints for CRUD operations (`GET`, `POST`, `PUT`, `DELETE`).

Once your backend is ready, update your frontend `fetch` calls to point to your new API endpoints instead of JSONPlaceholder. Because of CORS you are not able to call your Spring Boot API directly from the frontend. **To solve this, add CORS configuration to your Spring Boot application that allows requests from your frontend origin (e.g., `http://localhost:5500`)**.