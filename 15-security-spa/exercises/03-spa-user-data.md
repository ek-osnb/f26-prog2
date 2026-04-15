# Showing users specific data in a SPA with Spring Security (DRAFT)

## Exercise 1 - Implementing user persistence with Spring Data JPA

### Instructions
1. Add the following dependencies to your `pom.xml` file to include `Spring Data JPA` and the `MySQL driver`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```
2. Go through the previous exercise on **user persistence** with Spring Data JPA to create the `AppUser` and `Role` entities, as well as the corresponding repositories. This will set up the database structure for storing user information and roles.


## Exercise 2 - Creating a todo API

### Goal
Create a simple REST API for managing a list of todo items. Each user should only be able to see and manage their own todos.

### Instructions
1. Create a JPA entity named `Todo` with the following fields:
   - `id` (Long, primary key)
   - `title` (String) - not null
   - `completed` (boolean)
   - `user` (AppUser) - many-to-one relationship with `AppUser`
2. Create a Spring Data JPA repository interface named `TodoRepository` for the `Todo` entity. This repository should extend `JpaRepository<Todo, Long>` and include a method to find todos by the userId:
    ```java
    public interface TodoRepository extends JpaRepository<Todo, Long> {
        List<Todo> findAllByAppUserId(Long userId);
    }
    ```
3. Create a REST controller named `TodoController` with the following endpoints:
   - `GET /api/todos`: Returns a list of todos for the authenticated user.
   - `POST /api/todos`: Creates a new todo for the authenticated user. The request body should contain the `title` of the todo.
4. Ensure that all endpoints are secured so that only authenticated users can access them, and that users can only access their own todos.

    **Hint:** Inside the controller methods, you can access the authenticated user's information, by injecting the `Authentication` object. Use this information to associate todos with the correct user and to ensure that users can only access their own todos. Example:
    ```java
    public record TodoResponse(Long id, String title, boolean completed) {}

    @GetMapping
    public List<TodoResponse> getTodos(Authentication authentication) {
        String username = authentication.getName();
        AppUser user = appUserRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("User not found"));
        List<Todo> todos = todoRepository.findAllByAppUserId(user.getId());
        return todos.stream()
                .map(todo -> new TodoResponse(todo.getId(), todo.getTitle(), todo.isCompleted()))
                .toList();
    }
    ```
5. Create some test data by adding a few todos for each user in the database. You can do this by implementing the `CommandLineRunner` interface.

## Exercise 3 - Retrieving todos in the frontend

### Goal
Modify the frontend to fetch and display the list of todos for the authenticated user.

### Instructions
1. In the frontend, create a new function named `fetchTodos()` that makes a GET request to the `/api/todos` endpoint to retrieve the list of todos for the authenticated user.
```javascript
async function fetchTodos() {
    const response = await fetch("/api/todos");
    if (!response.ok) {
        throw new Error("Failed to fetch todos");
    }
    const todos = await response.json();
    return todos;
}
```
3. Add a new section in the `index.html` file to display the list of todos:
```html
<section id="todo-list" class="container hidden"></section>
```
4. Register the element inside the `setupUI()` function:
```javascript
UI_ELEMENTS.todoList = document.querySelector("#todo-list");
```
5. Create a new function named `displayTodos(todos)` that takes the list of todos and displays them in the UI. You can create a simple list to show the title of each todo and whether it is completed or not:
```javascript
function displayTodos(todos) {

    const todoList = document.createElement("ul");
    todoList.id = "todo-list";
    todos.forEach(todo => {
        const listItem = document.createElement("li");
        listItem.textContent = `${todo.title} - ${todo.completed ? "Completed" : "Not completed"}`;
        todoList.appendChild(listItem);
    });
    UI_ELEMENTS.todoList.innerHTML = "";
    UI_ELEMENTS.todoList.appendChild(todoList);
    UI_ELEMENTS.todoList.classList.remove("hidden");
}
```
6. Create a function to hide the todo list when the user logs out:
```javascript
function hideTodos() {
    UI_ELEMENTS.todoList.innerHTML = "";
    UI_ELEMENTS.todoList.classList.add("hidden");
}
```
7. Call the `fetchTodos()` function after a successful login to retrieve and display the todos for the authenticated user:
```javascript
form.reset();
setDisplayMessage("Login successful!", true);
const userInfo = await fetchUserInfo();
showUserInfo(userInfo);
hideLoginForm();

// ---- NEW CODE TO ADD IN handleLogin()
const todos = await fetchTodos();
displayTodos(todos);
```
8. Call the `hideTodos()` function after a successful logout to hide the todo list:
```javascript
// ---- CODE ALREADY PRESENT IN handleLogout()
hideUserInfo();
showLoginForm();
setDisplayMessage("Logged out successfully.", true);
try {
    await fetchUserInfo();
} catch {
    // Expected to fail with 401,
    // but this will trigger CSRF token generation
    // for the next login attempt
}
// ---- NEW CODE TO ADD IN handleLogout() AFTER fetchUserInfo()
hideTodos();
```
8. Inside the `initApp()` function, after setting up the UI, check if the user is already authenticated (e.g. from a previous session) and if so, fetch and display their todos:
```javascript
async function initApp() {
    setupUi();

    try {
        // This request serves two purposes:
        // 1. If the user is already logged in, we get their user info, 
        // or else we get a 401 unauthorized error, triggering the catch block
        // 2. If no CSRF cookie exists yet, Spring can generate one here
        const user = await fetchUserInfo();

        // If the request was successful, we have a logged in user and can show their info
        hideLoginForm();
        clearDisplayMessage();
        showUserInfo(user);

        // ---- NEW CODE TO ADD IN initApp() AFTER showUserInfo(user)
        const todos = await fetchTodos();
        displayTodos(todos);

    } catch (error) {
        // This is the normal path for anonymous users on first page load
        showLoginForm();
        hideUserInfo();
        clearDisplayMessage();
        hideTodos(); // Make sure todos are hidden for anonymous users
    }
}
```
9. Test the application by starting the application. Log in with different users and verify that each user can only see their own todos. Log out and verify that the todo list is hidden.

## Exercise 4 - Creating todos from the frontend

### Goal
Add functionality to the frontend to allow users to create new todos.

### Instructions
1. In the `index.html` file, add a form to create new todos:
```html
<form id="todo-form" class="hidden">
    <input type="text" id="todo-title" placeholder="Todo title" required />
    <button type="submit">Add Todo</button>
</form>
```
2. Register the form and input elements inside the `setupUI()` function:
```javascript
UI_ELEMENTS.todoForm = document.querySelector("#todo-form");
```
3. Create a new function named `handleTodoFormSubmit(event)` that will handle the form submission, send a POST request to the `/api/todos` endpoint to create a new todo, and then refresh the list of todos:
```javascript
async function handleTodoFormSubmit(event) {
    event.preventDefault();
    const form = event.target;

    const formData = new FormData(form);
    const title = formData.get("todo-title").trim();
    try {
        const csrfToken = getCsrfToken();
        const response = await fetch("/api/todos", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "X-XSRF-TOKEN": csrfToken
            },
            body: JSON.stringify({ title })
        });
        if (!response.ok) {
            throw new Error("Failed to create todo");
        }
        // Reset the form
        form.reset();
        // Refresh the list of todos
        const todos = await fetchTodos();
        displayTodos(todos);
    } catch (error) {
        // Handle error (optional in this exercise)
    }
}
```
4. Add an event listener to the form to handle submissions inside the `setupUI()` function:
```javascript
UI_ELEMENTS.todoForm.addEventListener("submit", handleTodoFormSubmit);
```
5. Show the todo form when the user is logged in, and hide it when the user is logged out. Create two new functions `showTodoForm()` and `hideTodoForm()` to handle this:
```javascript
function showTodoForm() {
    UI_ELEMENTS.todoForm.classList.remove("hidden");
}
function hideTodoForm() {
    UI_ELEMENTS.todoForm.classList.add("hidden");
}
```
6. Call the `showTodoForm()` function after a successful login, and the `hideTodoForm()` function after a successful logout:
```javascript
// After successful login
showUserInfo(userInfo);
showTodoForm(); // Show the todo form after login
// After successful logout
hideUserInfo();
hideTodoForm(); // Hide the todo form after logout
```
7. Test the application by logging in and creating new todos. Verify that the new todos are displayed in the list and that they are associated with the correct user (i.e. they are not visible when logging in with a different user).

## Reflection questions
- How does Spring Security know which todos belong to which user?
- What would happen if you forgot to associate the `Todo` entity with the `AppUser` entity?
- How does the frontend know which todos to display for the authenticated user?

## Extensions
- Replace the simple list of todos with a more user-friendly UI, such as a table.
- Add functionality to update and delete todos from the frontend.
- Add a "completed" checkbox to each todo item in the UI, and allow users to toggle the completed status of their todos.
- Implement error handling in the frontend to display error messages when API requests fail (e.g. when creating a new todo fails).
