# JavaScript Fetch API - CRUD exercises

## Introduction
In this exercise you will make a full CRUD (Create, Read, Update, Delete) application using the Fetch API to interact with a RESTful API. You will practice making GET, POST, PUT/PATCH, and DELETE requests to manage resources on the server. We will use the JSONPlaceholder API (https://jsonplaceholder.typicode.com/) for testing our requests.

> **Note**: The JSONPlaceholder API is a fake online REST API for testing and prototyping. It does not actually create, update, or delete data on the server, but it will respond as if it did.

## Starter code

**`index.html`:**

Notice that we use Bootstrap for styling, which gives us a nice layout and pre-styled components.
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
    <style>
        body {
            padding: 2rem;
        }
    </style>
    <title>Fetch example</title>
</head>

<body>
    <h1 class="text-center">TODO CRUD</h1>
    <form id="todoForm">
        <input type="hidden" id="todoId" name="id">
        <input type="text" id="todoTitle" name="title" placeholder="Enter a todo">
        <input type="text" id="userId" name="userId" placeholder="Enter user ID">
        <label for="completed">Completed</label>
        <input type="checkbox" id="completed" name="completed">

        <button type="submit" class="btn btn-primary">Add Todo</button>
    </form>

    <table class="table">
        <thead>
            <tr>
                <th scope="col">Title</th>
                <th scope="col">User ID</th>
                <th scope="col">Completed</th>
                <th scope="col">Actions</th>
            </tr>
        </thead>
        <tbody id="todoTableBody">
        </tbody>
    </table>

    <script src="app.js"></script>
</body>

</html>
```

**`app.js`:**
```js
document.addEventListener("DOMContentLoaded", initApp);

const BASE_URL_TODOS = "https://jsonplaceholder.typicode.com/todos";

function initApp() {}
```

## Exercise 1: Fetching and displaying todos

1. Create an async function called `fetchTodos` that fetches the list of todos from the JSONPlaceholder API:
    ```js
    async function fetchTodos() {
        const response = await fetch(BASE_URL_TODOS);
        const todos = await response.json();
        return todos;
    }
    ```
2. Make the `fetchTodos` use `try...catch` to handle any potential errors that may occur during the fetch operation, and check the response status to ensure it was successful.
3. Call the `fetchTodos` function inside `initApp`. We need to make the `initApp` function async to use `await` (try logging the todos to the console to verify that we are getting the data correctly):
    ```js
    async function initApp() {
        const todos = await fetchTodos();
    }
    ```
4. Now we have the list of todos in the console. Next, we need to display them in the table. Create a `displayTodos(todos)` function that takes the list of todos as an argument and renders them in the table body (`#todoTableBody`). It should delegate the rendering of each todo to a separate function called `renderTodoRow(todo)` that creates a table row for each todo and appends it to the table body:
```js
function displayTodos(todos) {
    const tableBody = document.getElementById("todoTableBody");
    tableBody.innerHTML = ""; // Clear existing rows
    for (const todo of todos) {
        renderTodoRow(todo);
    }
}

function renderTodoRow(todo) {
    const tableBody = document.getElementById("todoTableBody");
    const row = document.createElement("tr");
    row.setAttribute("data-id", todo.id);
    row.innerHTML = `
        <td>${todo.title}</td>
        <td>${todo.userId}</td>
        <td>${todo.completed ? "Yes" : "No"}</td>
        <td>
            <button class="btn btn-warning" data-action="edit">Edit</button>
            <button class="btn btn-danger" data-action="delete">Delete</button>
        </td>
    `;
    tableBody.appendChild(row);
}
```
5. Rewrite renderTodoRow to use `createElement` and `appendChild` instead of `innerHTML` to create the table row and its cells. This is a more secure way to create DOM elements.
    > Notice that we set a `data-id` attribute on the row to store the ID of the todo. And that we use `data-action` attributes on the buttons to identify their actions (edit or delete).

6. Finally, call the `displayTodos` function inside `initApp` after fetching the todos to render them in the table:
```js
async function initApp() {
    const todos = await fetchTodos();
    displayTodos(todos);
}
```

## Exercise 2: Adding a new todo
1. Create an async function called `addTodo` that takes a todo object as an argument and sends a POST request to the JSONPlaceholder API to create a new todo:
```js
async function addTodo(todo) {
    const response = await fetch(BASE_URL_TODOS, {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify(todo)
    });
    const newTodo = await response.json();
    return newTodo;
}
```
2. Add `try...catch` to the `addTodo` function to handle any potential errors that may occur during the fetch operation, and check the response status to ensure it was successful. And and check the response status to ensure it was successful.
3. Create a `handleFormSubmit` function that will be called when the form is submitted.
```js
async function handleFormSubmit(event) {
    event.preventDefault();
    const form = new FormData(event.target);
    const title = form.get("title");
    const userId = Number(form.get("userId"));
    const completed = form.get("completed") === "on"; // Checkbox value
    
    const newTodo = {
        title,
        userId,
        completed
    };
    const createdTodo = await addTodo(newTodo);
    console.log("Created todo:", createdTodo);
    // Optionally, you can add the new todo to the table without refetching all todos
    // renderTodoRow(createdTodo);

    event.target.reset(); // Clear the form
}
```
4. Add an event listener inside the `initApp` function to the form to handle the submit event and call the `handleFormSubmit` function:
```js
async function initApp() {
    const todos = await fetchTodos();
    displayTodos(todos);

    document.querySelector("#todoForm").addEventListener("submit", handleFormSubmit);
}
```

## Exercise 3: Deleting a todo
1. Create an async function called `deleteTodo` that takes a todo ID as an argument and sends a DELETE request to the JSONPlaceholder API to delete the todo:
```js
async function deleteTodo(id) {
    const response = await fetch(`${BASE_URL_TODOS}/${id}`, {
        method: "DELETE"
    });
    return true;
}
```
2. Add `try...catch` to the `deleteTodo` function to handle any potential errors that may occur during the fetch operation, and check the response status to ensure it was successful.
3. Create a function `handleTableClick` that will be called when any button in the table is clicked. This function should check if the clicked element is a delete button, and if so, call the `deleteTodo` function with the corresponding todo ID:
```js
async function handleTableClick(event) {
    const action = event.target.getAttribute("data-action");
    const row = event.target.closest("tr");
    const id = row.getAttribute("data-id");
    if (action === "delete") {
        const response = await deleteTodo(id);
        // Optionally, you can remove the row from the table without refetching all todos
        // if (response) {
        //     row.remove();
        // }
    }
}
```
4. Add an event listener to the table body to handle click events and call the `handleTableClick` function:
```js
async function initApp() {
    const todos = await fetchTodos();
    displayTodos(todos);

    document.querySelector("#todoForm").addEventListener("submit", handleFormSubmit);
    document.querySelector("#todoTableBody").addEventListener("click", handleTableClick);
}
```

## Exercise 4: Updating a todo
1. Create an async function called `updateTodo` that takes a todo ID and an updated todo object as arguments and sends a PUT request to the JSONPlaceholder API to update the todo:
```js
async function updateTodo(id, updatedTodo) {
    const response = await fetch(`${BASE_URL_TODOS}/${id}`, {
        method: "PUT",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify(updatedTodo)
    });
    const updatedData = await response.json();
    return updatedData;
}
```
2. Add `try...catch` to the `updateTodo` function to handle any potential errors that may occur during the fetch operation, and check the response status to ensure it was successful.
3. Modify the `handleTableClick` function to also handle edit button clicks. When an edit button is clicked, populate the form with the existing todo data and change the form submission to update the todo instead of creating a new one:
```js
async function handleTableClick(event) {
    const action = event.target.getAttribute("data-action");
    const row = event.target.closest("tr");
    const id = row.getAttribute("data-id");
    if (action === "delete") {
        await deleteTodo(id);
        // Optionally, remove the row from the table
        // row.remove();
    } else if (action === "edit") {
        // Populate form with existing todo data
        const title = row.children[0].textContent;
        const userId = row.children[1].textContent;
        const completed = row.children[2].textContent === "Yes";

        // Use .value for inputs and .checked for checkbox
        document.querySelector("#todoId").value = id; // hidden input to store the ID of the todo being edited
        document.querySelector("#todoTitle").value = title;
        document.querySelector("#userId").value = userId;
        document.querySelector("#completed").checked = completed;

    }
}
```
4. Modify the `handleFormSubmit` function to check if we are editing an existing todo (by checking if the hidden input `#todoId` has a value). If it does, call the `updateTodo` function instead of `addTodo`:
```js
async function handleFormSubmit(event) {
    event.preventDefault();
    const form = new FormData(event.target);
    const id = form.get("id");
    const title = form.get("title");
    const userId = Number(form.get("userId"));
    const completed = form.get("completed") === "on"; // Checkbox value
    
    const todoData = {
        title,
        userId,
        completed
    };

    if (id) {
        // Update existing todo
        const updatedTodo = await updateTodo(id, todoData);
        console.log("Updated todo:", updatedTodo);
        // Optionally, update the row in the table without refetching all todos
        // You would need to find the row with the corresponding ID and update its cells
    } else {
        // Add new todo
        const createdTodo = await addTodo(todoData);
        console.log("Created todo:", createdTodo);
        // Optionally, add the new todo to the table without refetching all todos
        // renderTodoRow(createdTodo);
    }
    event.target.reset(); // Clear the form
    document.querySelector("#todoId").value = ""; // Clear the hidden input
}
```





