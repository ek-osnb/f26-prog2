# JavaScript Intro - Building a Todo App

## Goal

Learn JavaScript fundamentals by building a simple todo application. You'll work with variables, arrays, objects, functions, DOM manipulation, and event handling—all the core concepts needed for frontend development.

## What You'll Build

A functional todo application where users can:
- See a list of initial todos
- Add new todos by typing and clicking a button
- Toggle todos as done/undone by clicking on them
- See a status counter showing the number of todos

### What You'll Learn

- **Variables and Data Structures**: Working with arrays and objects
- **DOM Manipulation**: Selecting and modifying HTML elements
- **Functions**: Creating reusable code blocks
- **Event Handling**: Responding to user interactions (clicks)
- **Dynamic Rendering**: Updating the UI based on data changes

## Starter Code

Create a new folder for this project and create a file called `index.html`. Copy the following code into that file:

```html
<!doctype html>
<html lang="en">

<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Todo app</title>
    <style>
        body {
            font-family: system-ui, sans-serif;
            max-width: 720px;
            margin: 2rem auto;
            padding: 0 1rem;
        }

        .row {
            display: flex;
            gap: .5rem;
        }

        input,
        button {
            padding: .6rem;
        }

        input {
            flex: 1;
        }

        #status {
            opacity: .7;
        }

        li {
            margin: .35rem 0;
        }

        .done {
            text-decoration: line-through;
            opacity: .7;
            color: gray;
        }

        li:hover {
            cursor: pointer;
            opacity: .7;
        }
    </style>
</head>

<body>
    <h1>Todo (Day 1)</h1>

    <div class="row">
        <input id="todo-input" placeholder="What needs doing?" />
        <button id="add-btn" type="button" class="add-btn">Add</button>
    </div>

    <p class="muted" id="status">Status: TODO</p>

    <ul id="todo-list">
        <li class="muted">Your todos will appear here…</li>
    </ul>
    <script>
        // Your JavaScript code will go here
    </script>
</body>
</html>
```
## How to Run
Run the live server in VS Code (click on **Go Live** in the bottom right corner) to serve the `index.html` file. This will open the file in your default web browser and allow you to see the changes you make in real-time as you edit the code.

The page will open at `http://localhost:5500` (the url for the live server).

## Step-by-Step Instructions

Work through the TODOs in the code in order. Each TODO builds on the previous one.

### Step 1: Create the Todos Array

Create an array called `todos` that holds your initial todo items. Each todo should be an object with two properties:
- `text`: A string describing the todo
- `done`: A boolean indicating if it's completed (start with `false`)

Try logging it to the console to see the structure:
```javascript
console.log(todos);
```

### Step 2: Get DOM Element References

Use `document.querySelector()` to get references to the HTML elements you'll need to manipulate:
- `inputEl`: The input field (`#todo-input`)
- `buttonEl`: The add button (`#add-btn`)
- `listEl`: The todo list (`#todo-list`)
- `statusEl`: The status paragraph (`#status`)

**Tip:** The `#` symbol selects elements by their `id` attribute.

**Checkpoint:** You now have references to the DOM elements, but still nothing visible changes.

### Step 3: Create the render() Function

Create a function called `render()` that displays all todos in the browser. This function should:

1. **Clear the list**: Set `listEl.innerHTML = ""` to remove all existing items
2. **Loop through todos**: Use a `for` loop to iterate over the `todos` array
3. **Create list items**: For each todo:
   - Create a new `<li>` element by using `document.createElement("li")`
   - Set its text content to the todo text
   - If the todo is done, add the CSS class `"done"` to it
   - Append the `<li>` to `listEl`
4. **Update status**: Set `statusEl.textContent` to show the count (e.g., "Status: 2 todos"). You can see the length of the array with `todos.length`.

After creating the function, **call it once** at the bottom to show the initial todos:
```javascript
render();
```

**Checkpoint:**  You should now see your initial todos displayed as a list, and the status should show "Status: 2 todos" (or however many you created).

### Step 4: Create the addTodo() Function

Create a function called `addTodo(text)` that adds a new todo to the array. This function should:

1. **Trim the text**: Use `text.trim()` to remove whitespace from the beginning and end
2. **Validate**: If the trimmed text is empty, return `false`
3. **Add todo**: Push a new todo object to the `todos` array with `done: false`
4. **Return true** if the todo was added successfully.

**Checkpoint:** The function is defined but not used yet, so nothing changes in the browser.

### Step 5: Create the addTodoClicked() Function

Create a function called `addTodoClicked()` that handles the button click. This function should:

1. **Get the input value**: Read the text from `inputEl.value`
2. **Call addTodo**: Pass the text to the `addTodo()` function (adds it to the array if valid)
3. **If successful** (addTodo returns `true`):
   - Clear the input field: `inputEl.value = ""`
   - Focus the input: `inputEl.focus()` (so the user can immediately type another todo)
   - Call `render()` to update the display

**Checkpoint:** Function is defined but still not connected to the button, so clicking doesn't work yet.

### Step 6: Wire the Event Handler

Connect the `addTodoClicked` function to the button's click event using `addEventListener`:

```javascript
buttonEl.addEventListener("click", addTodoClicked);
```

**Checkpoint:** Save and refresh. Now when you type a todo and click "Add", it should:
- Add the new todo to the list
- Clear the input field
- Keep the input focused
- Update the status counter

### Step 7: Update UI When Toggling

When a todo item is clicked, we want to toggle its "done" status. To do this, we can add a click event listener to the `#todo-list` element that listens for clicks on the `<li>` items.
```javascript
listEl.addEventListener("click", (event) => {
        // Find the index of the clicked <li> in the list
        const items = Array.from(listEl.children);
        const index = items.indexOf(event.target);

        if (index == -1) {
            return; // Clicked outside of a todo item
        }
        // Toggle the "done" status of the corresponding todo
        console.log("Clicked todo index:", index);
        todos[index].done = !todos[index].done;
        // Update the class of the clicked item based on the new status
        if (todos[index].done) {
            event.target.classList.add("done");
        } else {
            event.target.classList.remove("done");
        }
        render(); // Update the UI to reflect the change
    });
```

**Checkpoint:** Save and refresh. Now when you click on any todo in the list:
- It should toggle between normal and strike-through (done) style
- The styling is handled by the CSS class "done"

### Step 8: Refactor into Separate Files
Create a new file called `app.js` and move all your JavaScript code there. Then include it in your `index.html` by adding the following line just before the closing `</body>` tag:
```html
<script src="app.js"></script>
```

Create a new file called `style.css` and move all your CSS code there. Then include it in your `index.html` by adding the following line in the `<head>` section:
```html
<link rel="stylesheet" href="style.css" />
```

**Checkpoint:** Save all files and refresh. The app should work exactly the same, but now your code is organized into separate files for HTML, CSS, and JavaScript.