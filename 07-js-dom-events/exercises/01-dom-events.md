# JavaScript - DOM Manipulation and Events Exercises

## Introduction
In this exercise, you will practice DOM manipulation and handling events by building a simple user management app. The app allows you to add users with a name and age, display the list of users, and delete users from the list. You will use event listeners to handle form submissions and button clicks, and you will manipulate the DOM to update the user list dynamically.

## Starter code:
Create the following files in your project:
- `index.html`: The HTML structure of the app.
- `style.css`: The CSS styles for the app.
- `app.js`: The JavaScript code to handle DOM manipulation and events.

### HTML (index.html)
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User management app</title>
</head>

<body>
    <main>
        <!-- User Form -->
        <section class="container">
            <h1>Form</h1>
            <form id="userForm">
                <input type="text" id="name" name="name" placeholder="Name" required>
                <input type="number" id="age" name="age" min="0" max="120" placeholder="Age" required>
                <button type="submit">Add User</button>
            </form>
        </section>

        <!-- User List -->
        <section class="container" id="output">
            <h2>Users</h2>
            <p id="result"></p>
            <ul id="userList">
                <!-- User items will be added here -->
            </ul>
        </section>
    </main>
</body>

</html>
```

### CSS (style.css)
```css
html {
    margin: 0;
    padding: 0;
    font-family: Arial, Helvetica, sans-serif
}

body {
    margin: 1em;
    padding: 0;
    box-sizing: border-box;
    background-color: #f4f4f4;
}

.container {
    max-width: 400px;
    margin: 2em auto;
    background-color: white;
    padding: 1em 2em 2em 2em;
    border-radius: 1em;
    /* box-shadow: 0 2px 4px rgba(0,0,0,0.1); */
}

input {
    width: 100%;
    padding: 0.7em;
    margin-bottom: 1em;
    box-sizing: border-box;
    font-size: 16px;
}

button {
    font-size: 16px;
    background-color: #2c28a7;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    width: 100%;
    padding: 10px;
}

button:hover {
    background-color: #212f88;
}

#output {
    font-family: Arial, sans-serif;
}

#userList {
    list-style-type: none;
    padding: 0;
}

#userList li {
    background-color: rgb(236, 236, 236);
    margin-bottom: 0.5em;
    padding: 1em;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.hidden {
    display: none;
}

#instructions {
    max-width: 400px;
    margin: 2em auto;
}

#instructions ol li {
    font-size: 18px;
    padding-bottom: 1em;
}

li {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

li button {
    width: auto;
    padding: 0.5em 1em;
    font-size: 14px;
}
```

### JavaScript (app.js)
```javascript
// ============================
// INSTRUCTIONS
// ============================
//
// Complete each TODO in order starting from TODO 1. 
// Follow the instructions for each TODO to complete the app.

window.addEventListener("DOMContentLoaded", initApp);

function initApp() {
    // TODO 6: render initial users using displayUsers()

    // TODO 13: Add submit event listener to #userForm

    // TODO: 21: Add click event listener to the <ul> element that contains the user list
    // Use event delegation to handle delete button clicks
    
}


function displayUsers() {
    // TODO 1: Get all users, and clear existing list in #userList
    // Hint: set innerHTML of #userList to an empty string
    // Hint: use api.getUsers() to get the list of users
    
	
    // TODO 2: Call displayUserItem for each user in the list of users
    // Hint: Use a for...of loop
    
}

function displayUserItem(user) {
    // TODO 3: Create an <li> element
    
    // TODO 4: Insert name and age (seperated by a dash) as text into <li> and 
    // set data-id attribute to the user's id
    // Hint: use setAttribute("data-id", user.id) to set the data-id attribute
    
    // TODO 5: Append <li> to the #userList
    
    // TODO 14: Create a button element
    
    // TODO 15: Set the text content of the button to "Delete"
    
    // TODO 16: Append the button to the <li>
    
}

function handleFormSubmit(event) {
    // TODO 7: Prevent default
    
    // TODO 8: Create a FormData object from the form
    
    // TODO 9: Create a new user object with the name and age
    
    // TODO 10: pass the new user object to api.createUser()
    
    // TODO 11: display users using displayUsers()
    
    // TODO 12: clear the form (use event.target.reset())
    
}


function handleDelete(event) {
    // TODO 17: Check if the clicked element is a delete button
    // Hint: if event.target.tagName is not equal to BUTTON
    // Return early from the function

    // TODO 18: Get the closest <li> element

    // TODO 19: use the api.deleteUser(id) function to delete
    // Hint: the id is stored in the data-id attribute of the <li>

    // TODO 20: re-render the users list by calling displayUsers()
}

// ===============================================
// DO NOT EDIT BELOW THIS LINE
// ===============================================

const api = (() => {
    const users = [
        { id: 1, name: "Alice", age: 30 },
        { id: 2, name: "Bob", age: 25 },
        { id: 3, name: "Charlie", age: 35 },
    ];

    let nextId = Math.max(...users.map(u => u.id)) + 1;

    function generateId() {   // private
        return nextId++;
    }

    return {
        getUsers() {
            return users.map(u => ({ ...u }));
        },
        deleteUser(id) {
            const index = users.findIndex(u => u.id === Number(id));
            if (index !== -1) users.splice(index, 1);
        },
        createUser(user) {

            users.push({
                id: generateId(),
                name: user.name,
                age: Number(user.age),
            });
        },
    };
})();
```

## Instructions

The exercises are devided into three parts, each with a set of TODOs.

### Part 0: Setup
1. Create the three files (`index.html`, `style.css`, and `app.js`) and copy the corresponding starter code into each file.

2. Add a `<script>` tag to the end of the `<body>` in `index.html` to link the `app.js` file.

3. Add a `<link>` tag in the `<head>` of `index.html` to link the `style.css` file.

### Part 1:

Finish TODO 1 - 6: Displaying users in the DOM, the instructions for each TODO are in the comments of the `app.js` file.


### Part 2: Todo 7 - 13: Handling form submission and adding users

Finish TODO 7 - 13: Handle form submission to add new users, the instructions for each TODO are in the comments of the `app.js` file.

### Part 3: Todo 14 - 21: Deleting users using event delegation

Finish TODO 14 - 21: Handle delete button clicks using event delegation, the instructions for each TODO are in the comments of the `app.js` file.