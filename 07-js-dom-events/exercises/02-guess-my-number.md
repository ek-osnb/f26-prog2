# Guess My Number – Exercise

In this exercise, you will build a fully interactive "Guess My Number" game using JavaScript, HTML, and CSS.

## Goal
Create a game where the computer picks a random number between two values (e.g., 1 and 100), and the user tries to guess it. The game should give feedback after each guess and track the number of attempts.

## Starter Code

### HTML (index.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="stylesheet" href="style.css">
	<title>Guess My Number</title>
</head>
<body>
	<main class="container">
		<h1>Guess My Number!</h1>
		<p class="instructions">I'm thinking of a number between <span id="min">1</span> and <span id="max">100</span>. Can you guess it?</p>
		<form id="guessForm" autocomplete="off">
			<input type="number" id="guessInput" min="1" max="100" placeholder="Enter guess" required>
			<button type="submit">Guess</button>
		</form>
		<p id="feedback"></p>
		<p id="attempts"></p>
		<button id="restartBtn" style="display:none;">Restart Game</button>
	</main>
	<script src="app.js"></script>
</body>
</html>
```
### CSS (style.css)
```css
body {
	background: #f4f6fb;
	min-height: 100vh;
	margin: 0;
	font-family: 'Segoe UI', Arial, sans-serif;
}

.container {
	max-width: 400px;
	margin: 60px auto;
	background: #fff;
	border-radius: 12px;
	box-shadow: 0 4px 16px rgba(0,0,0,0.10);
	padding: 36px 32px 28px 32px;
	text-align: center;
}

h1 {
	color: #2563eb;
	margin-bottom: 18px;
	font-size: 2em;
	letter-spacing: 1px;
}

.instructions {
	color: #444;
	margin-bottom: 24px;
	font-size: 1.1em;
}

#guessForm {
	display: flex;
	gap: 10px;
	justify-content: center;
	margin-bottom: 18px;
}

#guessInput {
	padding: 10px 12px;
	font-size: 1.1em;
	border: 1px solid #bcd0ee;
	border-radius: 5px;
	width: 120px;
	outline: none;
	transition: border 0.2s;
}
#guessInput:focus {
	border: 1.5px solid #2563eb;
}

button {
	padding: 10px 22px;
	font-size: 1.1em;
	border: none;
	border-radius: 5px;
	background: #4f8cff;
	color: #fff;
	cursor: pointer;
	transition: background 0.2s;
}
button:hover {
	background: #2563eb;
}

#feedback {
	min-height: 32px;
	margin: 18px 0 8px 0;
	font-size: 1.15em;
	font-weight: 500;
}
#feedback.info {
	color: #2563eb;
}
#feedback.success {
	color: #1ca97a;
}
#feedback.warning {
	color: #eab308;
}

#attempts {
	color: #888;
	font-size: 1em;
	margin-bottom: 10px;
}

#restartBtn {
	margin-top: 18px;
	background: #ff5c5c;
	color: #fff;
	border: none;
	border-radius: 5px;
	padding: 10px 22px;
	font-size: 1.1em;
	cursor: pointer;
	transition: background 0.2s;
}
#restartBtn:hover {
	background: #d90429;
}

.hidden {
	display: none;
}
```
### JavaScript (app.js)
```javascript
const MIN = 1;
const MAX = 100;
let secretNumber;
let attempts;

// DOM elements to be used:
// const guessForm = document.querySelector("#guessForm");
// const guessInput = document.querySelector("#guessInput");
// const feedback = document.querySelector("#feedback");
// const attemptsDisplay = document.querySelector("#attempts");
// const restartBtn = document.querySelector("#restartBtn");

// TODO
function initGame() {
	// TODO 6: Call the `resetGame` function to initialize the game state when the page loads.

	// TODO 14: Add event listener to the guess form
    
	// TODO 15: Add event listener to the restart button
    
}

// TODO
function resetGame() {
    // TODO 1: Generate a random number between MIN and MAX, and store it in the `secretNumber` variable.
	// Hint: Use `Math.random()` and `Math.floor()` to generate the random number.
    
	// TODO 2: Reset the attempts counter to 0.
    
	// TODO 3: Update the attemptsDisplay in the DOM to show the reset attempts count.
    
	// TODO 4: Clear any feedback messages in the DOM.
    
	// TODO 5: Add the "hidden" class to the restart button to hide it.
    
}

function handleGuess(event) {
	// TODO 7: Prevent the default form submission behavior to avoid page reloads.
    
	// TODO 8: Get the user's guess from the input field and convert it to a number (use `parseInt` ex. parseInt("2")).
    
	// TODO 9: Create a boolean variable `isBetweenRange` that checks if the guess is a valid number and within the defined range (between MIN and MAX).

	// TODO 10: Create a boolean variable `isValidNumber` that checks if the guess is a valid number (not NaN use isNan function).


	// TODO 11: If the guess is a valid number and within the range, increment the attempts counter and update the attempts display in the DOM.

	// TODO 12: Provide feedback to the user:
	// - If the guess is correct, display a congratulatory message in the feedback element, and show the restart button (remove hidden class from restartBtn).
	// - If the guess is too low, display a message indicating that the guess is too low.
	// - If the guess is too high, display a message indicating that the guess is too high.

	// TODO 13: Clear the form input field (use event.target.reset() to clear the form after submission).
}
```

## Instructions
1. Start by setting up the project structure with the provided HTML, CSS, and JavaScript files.
2. Follow the TODO comments (in order) in the `app.js` file to implement the game logic step by step.
3. Test the game in the browser to ensure it works as expected.

## Bonus Challenge
1. Add a feature to track and display the user's previous guesses, should be displayed in a list below the game, and reset when the game is restarted.
2. Add a feature to store the best score (least attempts) in the browser's local storage and display it on the page
> [!TIP]
> Local storage allows you to save data in the user's browser, so it persists even after the page is refreshed. You can use `localStorage.setItem(key, value)` to save data and `localStorage.getItem(key)` to retrieve it. For example, you could save the best score with `localStorage.setItem("bestScore", attempts)` and retrieve it with `const bestScore = localStorage.getItem("bestScore")`.