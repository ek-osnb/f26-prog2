# JavaScript Exercises: Simple DOM Manipulation

## Setup:
Create a new folder for this exercise.
Inside this folder, create two files: `index.html` and `app.js`.

Add the following boilerplate code to `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JavaScript DOM Exercise</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
        }
        .container {
            padding: 2em;
            height: 10em;
        }
        .container:nth-child(even) {
            background-color: #beb9b9;
        }
        .hidden {
            display: none;
        }
        button {
            cursor: pointer;
        }
    </style>
</head>
<body>
    <section class="container" id="ex1">
        <h1>JavaScript Exercise: Change background color</h1>
        <input type="text" id="colorPicker" />
        <button id="changeColorBtn">Set Background Color</button>
    </section>
    <section class="container" id="ex2">
        <h1>JavaScript Exercise: Toggle Visibility</h1>
        <button id="toggleBtn">Toggle Text</button>
        <p id="toggleText">This text can be toggled.</p>
    </section>
    <section class="container" id="ex3">
        <h1>JavaScript Exercise: Counter</h1>
        <h2>Value: <span id="counter">0</span></h2>
        <button id="increment">Increment</button>
        <button id="decrement">Decrement</button>
        <button id="reset">Reset</button>
    </section>
    <section class="container" id="ex4">
        <h1>JavaScript Exercise: Add input</h1>
        <input type="text" id="newInput" />
        <button id="addInputBtn">Add Item</button>
        <p>OUTPUT: <span id="output"></span></p>
    </section>
     <section class="container" id="ex5">
        <h1>JavaScript Exercise: Increase font size</h1>
        <button id="fontSizeBtn">Increase font Size</button>
    </section>
</body>
<script src="app.js"></script>
</html>
```

**Take some time to understand the structure of the HTML file.**

Start the live server (runs on `localhost:5500`) and open the browser to see the page.

## Exercise 1: Change Background Color

### Step 1: Basic Implementation

Inside `app.js`, write JavaScript code to change the background color of the page when the button is clicked.

```javascript
// app.js
document.querySelector("#changeColorBtn").addEventListener("click", function() {
    var color = document.querySelector("#colorPicker").value;
    document.body.style.backgroundColor = color;
});
```
This code adds an event listener to the button. When clicked, it retrieves the value from the input field and sets it as the background color of the page.

The `value` property of an input element gets the current value of that input field.

The `style` property of an element allows you to get or set the inline style of that element. Here, we are setting the `backgroundColor` property of the `body` element.

**NOTE:** `style` has properties for all CSS styles, but note that the property names are in camelCase (e.g., `backgroundColor` instead of `background-color`).


**Testing:**
1. Open the page in your browser.
2. Enter a color name (like "red", "blue", or "green") or a hex color code (like "#ff5733") in the input field.
3. Click the "Set Background Color" button.
4. Observe the background color of the page change to the specified color.

### Step 2: Extracting it into a function

This works but let's make it cleaner by extracting the logic into a function.
```javascript
"use strict";

document.addEventListener("load", initApp);

function initApp() {
    document.querySelector("#changeColorBtn").addEventListener("click", changeBackgroundColor);
}

// Exercise 1: Change Background Color
function changeBackgroundColor() {
    var color = document.querySelector("#colorPicker").value;
    document.body.style.backgroundColor = color;
}
```

The first line `"use strict";` enables strict mode which helps catch common coding mistakes and "unsafe" actions.
The `initApp` function is called when the document is fully loaded. It sets up the event listener for the button, which calls the `changeBackgroundColor` function when clicked. This keeps the code organized and easier to maintain.

## Exercise 2: Toggle Visibility

To toggle the visibility, we will tap into the `classList` property of the element.

The `classList` property returns the class names of an element as a DOMTokenList object. It has methods to add, remove, and toggle classes.

Try the following code:
```javascript
// Exercise 2: Toggle Visibility

function toggleVisibility() {
    const toggleText = document.querySelector("#toggleText");
    console.log(toggleText.classList);
}
```

Add the following line to the `initApp` function:
```javascript
document.querySelector("#toggleBtn").addEventListener("click", toggleVisibility);
```

When you click the button, you should see the class list of the paragraph logged in the console. At the moment it should only contain the class `hidden`

Now, let's use the `toggle` method to add or remove the `hidden` class when the button is clicked:
```javascript
// Exercise 2: Toggle Visibility
function toggleVisibility() {
    const toggleText = document.querySelector("#toggleText");
    console.log(toggleText.classList);
    toggleText.classList.toggle("hidden");
}
```

When you click the button, the paragraph should appear and disappear as you click it.

## Exercise 3: Counter
For this exercise, we will create a simple counter that can be incremented, decremented, and reset.

We need three functions: `increment`, `decrement`, and `reset`. Each function will update the counter value displayed on the page.

To change the text inside an HTML element, we can use the `textContent` property of the element.

```javascript
// Exercise 3: Counter
function increment() {
    const counter = document.querySelector("#counter");
    counter.textContent++; // = parseInt(counter.textContent) + 1;
    console.log("Counter incremented");
}
```
If you add the following line to the `initApp` function:
```javascript
document.querySelector("#increment").addEventListener("click", increment);
```
You should see the counter value increase by 1 each time you click the "Increment" button.

#### Student Task:
- Implement the `decrement` and `reset` functions in a similar way

## Exercise 4: Add Input
In this exercise, we will create a function that takes the value from an input field and appends it to a paragraph when a button is clicked.

```javascript
// Exercise 4: Add Input
function addInput() {
    const newInput = document.querySelector("#newInput").value;
    const output = document.querySelector("#output");
    output.textContent += newInput + " ";
    document.querySelector("#newInput").value = ""; // Clear the input field
    console.log("Input added");
}
```

Add the following line to the `initApp` function:
```javascript
document.querySelector("#addInputBtn").addEventListener("click", addInput);
```
When you enter text in the input field and click the "Add Item" button, the text should be appended to the paragraph below it.


## Exercise 5: Increase font size

## Step 1: Basic Implementation

We can extract all the `p` tags and increase their font size when a button is clicked.

```javascript
// Exercise 5: Increase font size
function handleFontSizeClick() {
    const paragraphs = document.querySelectorAll("p");
    paragraphs.forEach(increaseFontSize);
}

function increaseFontSize(tag) {
    console.log("Computed font size:", getComputedStyle(tag).fontSize);
}
```
The `querySelectorAll` method returns a list of all elements that match the specified CSS selector. In this case, it selects all `p` tags on the page.

Add the following line to the `initApp` function:
```javascript
document.querySelector("#fontSizeBtn").addEventListener("click", handleFontSizeClick);
```
When you click the "Increase Font Size" button, it should log the computed font size of each paragraph to the console.

Now, let's actually increase the font size. We can do this by parsing the current font size, adding a certain value to it, and then setting it back.

```javascript
function increaseFontSize(tag) {
    console.log("Computed font size:", getComputedStyle(tag).fontSize);
    const currentFontSize = parseFloat(getComputedStyle(tag).fontSize);
    tag.style.fontSize = (currentFontSize + 1) + "px";
}
```

The parseFloat function converts a string to a floating-point number. We use it to extract the numeric part of the font size (which is in pixels, e.g., "16px").

When you click the "Increase Font Size" button, the font size of all paragraphs should increase by 1 pixel each time.

## Step 2: Generalize the function

We can make the `handleFontSizeClick` function more general by allowing it to take a parameter for the html tag to be modified.

```javascript
function handleFontSize(tag) {
    const text = document.querySelectorAll(tag);
    text.forEach(increaseFontSize);
}
```

Now, we can modify the event listener to pass the `p` tag
```javascript
document.querySelector("#fontSizeBtn").addEventListener("click", () => handleFontSize("p"));
```
Notice that we are using an arrow function to call `handleFontSize` with the argument `"p"`. This is necessary because we want to pass a function to the event listener, not the result of calling the function. We could also use a regular function:
```javascript
document.querySelector("#fontSizeBtn").addEventListener("click", function() {
    handleFontSize("p");
});
```