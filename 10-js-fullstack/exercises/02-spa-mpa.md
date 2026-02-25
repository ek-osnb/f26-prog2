# Single-Page Applications (SPA) vs Multi-Page Applications (MPA)

## Introduction

In this tutorial, you will learn the fundamental differences between **Multi-Page Applications (MPA)** and **Single-Page Applications (SPA)**, and how to implement client-side routing using **Navigo** - a simple, lightweight JavaScript router.

### What You'll Learn
- Understand the architecture of Multi-Page Applications
- Understand the architecture of Single-Page Applications
- Learn the pros and cons of each approach
- Build a simple MPA
- Convert it to an SPA
- Implement client-side routing with Navigo
- Handle route parameters and navigation
- Integrate routing with your component-based architecture

### Prerequisites
- Basic HTML, CSS, and JavaScript
- Understanding of DOM manipulation
- Familiarity with ES6 modules
- (Optional) Component-based architecture from previous exercises


## Part 1: Understanding Multi-Page Applications (MPA)

### What is an MPA?

A **Multi-Page Application** is the traditional web architecture where:
- Each page is a separate HTML file
- Clicking a link loads a completely new page from the server
- The browser makes a full page reload
- Each page has its own URL and HTML document

### MPA Architecture

```
User clicks link → Browser requests new page → Server sends HTML → Browser reloads entire page
```

**Example MPA Structure:**
```
website/
├── index.html          # Home page
├── about.html          # About page
├── products.html       # Products page
├── contact.html        # Contact page
├── styles.css          # Shared styles
└── script.js           # Shared scripts
```
<!-- 
### Pros of MPAs
✅ **SEO Friendly** - Each page has its own URL and content  
✅ **Simple to understand** - Traditional web development  
✅ **Browser history works automatically** - Back/forward buttons work  
✅ **No JavaScript required** - Works without JS enabled  
✅ **Easy to deploy** - Just upload HTML files  

### Cons of MPAs
❌ **Slower page transitions** - Full page reload every time  
❌ **More server requests** - Every navigation hits the server  
❌ **State is lost** - Application state resets on each page  
❌ **Redundant code** - Header/footer repeated in every file  
❌ **Less interactive** - Doesn't feel like an app   -->


## Part 2: Understanding Single-Page Applications (SPA)

### What is an SPA?

A **Single-Page Application** is a modern web architecture where:
- One HTML file serves the entire application
- Content changes are handled by JavaScript
- No full page reloads when navigating
- URLs are simulated using JavaScript routing
- Feels more like a native app

### SPA Architecture

```
User clicks link → JavaScript intercepts → Updates URL (History API) → Updates DOM → No server request
```

**Example SPA Structure:**
```
website/
├── index.html          # Single HTML file
├── styles.css
├── app.js              # Main application
├── router.js           # Client-side routing
├── HomePage.js         # Home view component
├── AboutPage.js        # About view component
├── ProductsPage.js     # Products view component
└── ContactPage.js      # Contact view component
```

<!-- ### Pros of SPAs
✅ **Fast navigation** - No page reloads, instant transitions  
✅ **Native app feel** - Smooth, responsive user experience  
✅ **State preservation** - Keep application state across views  
✅ **Less server load** - Fetch only data, not entire pages  
✅ **Code reusability** - Shared components across views  
✅ **Offline capability** - Can work with cached data  

### Cons of SPAs
❌ **SEO challenges** - Requires server-side rendering or prerendering  
❌ **Initial load time** - Must download all JavaScript upfront  
❌ **Requires JavaScript** - Won't work without JS  
❌ **Complexity** - More complex to build and maintain  
❌ **Browser history** - Must manually handle with History API   -->


## Part 3: Build a Simple Multi-Page Application

Let's build a simple task management MPA with three pages.

### Step 1: Create the Home Page

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager - Home</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav class="navbar">
        <div class="container">
            <h1>📝 Task Manager</h1>
            <ul class="nav-links">
                <li><a href="index.html" class="active">Home</a></li>
                <li><a href="tasks.html">Tasks</a></li>
                <li><a href="about.html">About</a></li>
            </ul>
        </div>
    </nav>

    <main class="container">
        <div class="hero">
            <h2>Welcome to Task Manager</h2>
            <p>Manage your tasks efficiently and stay productive!</p>
            <a href="tasks.html" class="btn">View Tasks</a>
        </div>
    </main>

    <footer class="footer">
        <p>&copy; 2026 Task Manager. All rights reserved.</p>
    </footer>
</body>
</html>
```

### Step 2: Create the Tasks Page

Create `tasks.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager - Tasks</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav class="navbar">
        <div class="container">
            <h1>📝 Task Manager</h1>
            <ul class="nav-links">
                <li><a href="index.html">Home</a></li>
                <li><a href="tasks.html" class="active">Tasks</a></li>
                <li><a href="about.html">About</a></li>
            </ul>
        </div>
    </nav>

    <main class="container">
        <h2>My Tasks</h2>
        <div id="task-list">
            <div class="task">
                <input type="checkbox">
                <span>Complete JavaScript tutorial</span>
            </div>
            <div class="task">
                <input type="checkbox">
                <span>Build a SPA application</span>
            </div>
            <div class="task">
                <input type="checkbox">
                <span>Learn about routing</span>
            </div>
        </div>
    </main>

    <footer class="footer">
        <p>&copy; 2026 Task Manager. All rights reserved.</p>
    </footer>
</body>
</html>
```

### Step 3: Create the About Page

Create `about.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager - About</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav class="navbar">
        <div class="container">
            <h1>Task Manager</h1>
            <ul class="nav-links">
                <li><a href="index.html">Home</a></li>
                <li><a href="tasks.html">Tasks</a></li>
                <li><a href="about.html" class="active">About</a></li>
            </ul>
        </div>
    </nav>

    <main class="container">
        <h2>About Task Manager</h2>
        <p>Task Manager is a simple application to help you organize your daily tasks.</p>
        <p>Built with HTML, CSS, and JavaScript.</p>
    </main>

    <footer class="footer">
        <p>&copy; 2026 Task Manager. All rights reserved.</p>
    </footer>
</body>
</html>
```

### Step 4: Create Shared Styles

Create `styles.css`:

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

.navbar {
    background: #667eea;
    color: white;
    padding: 1rem 0;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.navbar .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 2rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.navbar h1 {
    font-size: 1.5rem;
}

.nav-links {
    list-style: none;
    display: flex;
    gap: 2rem;
}

.nav-links a {
    color: white;
    text-decoration: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    transition: background 0.3s;
}

.nav-links a:hover,
.nav-links a.active {
    background: rgba(255, 255, 255, 0.2);
}

main.container {
    max-width: 1200px;
    margin: 2rem auto;
    padding: 0 2rem;
    flex: 1;
}

.hero {
    text-align: center;
    padding: 4rem 2rem;
}

.hero h2 {
    font-size: 2.5rem;
    margin-bottom: 1rem;
    color: #333;
}

.hero p {
    font-size: 1.2rem;
    color: #666;
    margin-bottom: 2rem;
}

.btn {
    display: inline-block;
    padding: 1rem 2rem;
    background: #667eea;
    color: white;
    text-decoration: none;
    border-radius: 8px;
    transition: background 0.3s;
    margin: 0.5rem;
}

.btn:hover {
    background: #5568d3;
}

#task-list {
    margin-top: 2rem;
}

.task {
    padding: 1rem;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    gap: 1rem;
}

.task input[type="checkbox"] {
    width: 20px;
    height: 20px;
    cursor: pointer;
}

.footer {
    background: #f8f9fa;
    padding: 2rem;
    text-align: center;
    color: #666;
    margin-top: auto;
}
```

### Step 5: Test Your MPA

1. Open `index.html` in your browser
2. Click the navigation links
3. **Notice:** The page fully reloads each time (watch the browser tab icon)
4. **Notice:** The entire page flashes white during navigation
5. **Notice:** Browser history works automatically

**Key Observation:** Every click triggers a full page reload from the server.


## Part 4: Convert to a Single-Page Application

Now let's convert the MPA to an SPA using vanilla JavaScript (no router yet).

### Step 1: Create a Single HTML File

Create `spa-index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Task Manager - SPA</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav class="navbar">
        <div class="container">
            <h1>📝 Task Manager</h1>
            <ul class="nav-links">
                <li><a href="#/" data-link>Home</a></li>
                <li><a href="#/tasks" data-link>Tasks</a></li>
                <li><a href="#/about" data-link>About</a></li>
            </ul>
        </div>
    </nav>

    <main id="app" class="container">
        <!-- Content will be dynamically loaded here -->
    </main>

    <footer class="footer">
        <p>&copy; 2026 Task Manager. All rights reserved.</p>
    </footer>

    <script type="module" src="spa-app.js"></script>
</body>
</html>
```

### Step 2: Create the SPA JavaScript

Create `spa-app.js`:

```js
// spa-app.js

// View templates
const views = {
    home: () => `
        <div class="hero">
            <h2>Welcome to Task Manager</h2>
            <p>Manage your tasks efficiently and stay productive!</p>
            <a href="#/tasks" class="btn" data-link>View Tasks</a>
        </div>
    `,
    
    tasks: () => `
        <h2>My Tasks</h2>
        <div id="task-list">
            <div class="task">
                <input type="checkbox">
                <span>Complete JavaScript tutorial</span>
            </div>
            <div class="task">
                <input type="checkbox">
                <span>Build a SPA application</span>
            </div>
            <div class="task">
                <input type="checkbox">
                <span>Learn about routing</span>
            </div>
        </div>
    `,
    
    about: () => `
        <h2>About Task Manager</h2>
        <p>Task Manager is a simple application to help you organize your daily tasks.</p>
        <p>Built with HTML, CSS, and JavaScript.</p>
    `,
    
    notFound: () => `
        <div class="hero">
            <h2>404 - Page Not Found</h2>
            <p>The page you're looking for doesn't exist.</p>
            <a href="#/" class="btn" data-link>Go Home</a>
        </div>
    `
};

// Router logic
function router() {
    // Get the hash (e.g., #/tasks)
    const hash = window.location.hash.slice(1) || '/';
    
    // Map hash to view
    let view;
    if (hash === '/') {
        view = views.home();
    } else if (hash === '/tasks') {
        view = views.tasks();
    } else if (hash === '/about') {
        view = views.about();
    } else {
        view = views.notFound();
    }
    
    // Update the app container
    document.getElementById('app').innerHTML = view;
    
    // Update active nav link
    updateActiveLink(hash);
}

// Update active navigation link
function updateActiveLink(currentHash) {
    document.querySelectorAll('.nav-links a').forEach(link => {
        link.classList.remove('active');
        if (link.getAttribute('href') === `#${currentHash}`) {
            link.classList.add('active');
        }
    });
}

// Listen for hash changes
window.addEventListener('hashchange', router);

// Initial load
router();
```

### Step 3: Test Your SPA

1. Open `spa-index.html` in your browser
2. Click the navigation links
3. **Notice:** No page reload! Content changes instantly
4. **Notice:** URL changes in the address bar (hash-based routing)
5. **Notice:** Browser back/forward buttons work

**Key Observation:** Navigation is instant with no page reload!

### Limitations of This Simple SPA

- Uses hash-based routing (`#/page`) instead of clean URLs
- No support for route parameters (e.g., `/tasks/123`)
- Manual route management is tedious
- No nested routes or route guards

**Solution:** Use a router library like Navigo!


## Part 5: Introduction to Navigo

**Navigo** is a simple, lightweight JavaScript router for SPAs. It provides:
- Clean URL routing (no hash required)
- Route parameters and wildcards
- Route hooks (before/after navigation)
- TypeScript support
- Small bundle size (~3KB)

### Installing Navigo

Add this to your HTML `<head>`:

```html
<script src="https://unpkg.com/navigo@8"></script>
```


## Part 6: Build an SPA with Navigo

Let's rebuild our SPA using Navigo with clean URLs and advanced routing features.

### Step 1: Create the HTML

Create `navigo-index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SPA intro</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav class="navbar">
        <div class="container">
            <h1>Task Manager</h1>
            <ul class="nav-links">
                <li><a href="/" data-navigo>Home</a></li>
                <li><a href="/tasks" data-navigo>Tasks</a></li>
                <li><a href="/about" data-navigo>About</a></li>
            </ul>
        </div>
    </nav>

    <main id="app" class="container">
        <!-- Content will be dynamically loaded here -->
    </main>

    <footer class="footer">
        <p>&copy; 2026 Task Manager. All rights reserved.</p>
    </footer>

    <!-- Navigo CDN -->
    <script src="https://unpkg.com/navigo@8"></script>
    <script type="module" src="navigo-app.js"></script>
</body>
</html>
```

### Step 2: Create the Router

Create `navigo-app.js`:

```js
// navigo-app.js

// Initialize Navigo router
// Use '/' as root if hosting at domain root, or '/your-app' if in a subfolder
const router = new Navigo('/');

// Get the app container
const app = document.getElementById('app');

// View rendering functions
function renderHome() {
    app.innerHTML = `
        <div class="hero">
            <h2>Welcome to Task Manager</h2>
            <p>Manage your tasks efficiently and stay productive!</p>
            <a href="/tasks" data-navigo class="btn">View Tasks</a>
        </div>
    `;
    updateActiveLink('/');
}

function renderTasks() {
    app.innerHTML = `
        <h2>My Tasks</h2>
        <div id="task-list">
            <div class="task">
                <input type="checkbox">
                <span>Complete JavaScript tutorial</span>
            </div>
            <div class="task">
                <input type="checkbox">
                <span>Build a SPA application</span>
            </div>
            <div class="task">
                <input type="checkbox">
                <span>Learn about routing</span>
            </div>
        </div>
        <div style="margin-top: 2rem;">
            <a href="/tasks/1" data-navigo class="btn">View Task 1</a>
            <a href="/tasks/2" data-navigo class="btn">View Task 2</a>
        </div>
    `;
    updateActiveLink('/tasks');
}

function renderTaskDetail(params) {
    const taskId = params.data.id;
    app.innerHTML = `
        <h2>Task Details</h2>
        <div class="task" style="font-size: 1.2rem;">
            <p><strong>Task ID:</strong> ${taskId}</p>
            <p><strong>Title:</strong> Task ${taskId}</p>
            <p><strong>Status:</strong> In Progress</p>
        </div>
        <a href="/tasks" data-navigo class="btn">Back to Tasks</a>
    `;
    updateActiveLink('/tasks');
}

function renderAbout() {
    app.innerHTML = `
        <h2>About Task Manager</h2>
        <p>Task Manager is a simple application to help you organize your daily tasks.</p>
        <p>Built with HTML, CSS, JavaScript, and Navigo for routing.</p>
    `;
    updateActiveLink('/about');
}

function render404() {
    app.innerHTML = `
        <div class="hero">
            <h2>404 - Page Not Found</h2>
            <p>The page you're looking for doesn't exist.</p>
            <a href="/" data-navigo class="btn">Go Home</a>
        </div>
    `;
}

// Update active navigation link
function updateActiveLink(path) {
    document.querySelectorAll('.nav-links a').forEach(link => {
        link.classList.remove('active');
        const href = link.getAttribute('href');
        if (href === path || (path.startsWith(href) && href !== '/')) {
            link.classList.add('active');
        }
    });
}

// Define routes
router
    .on('/', renderHome)
    .on('/tasks', renderTasks)
    .on('/tasks/:id', renderTaskDetail)  // Route with parameter
    .on('/about', renderAbout)
    .notFound(render404);

// Resolve the initial route
router.resolve();
```

### Step 3: Test Navigo Routing

### Understanding the Code

**Navigo Initialization:**
```js
const router = new Navigo('/');
```
- Creates a router instance
- `/` means app is hosted at the domain root

**Route Definition:**
```js
router
    .on('/', renderHome)                    // Exact match
    .on('/tasks', renderTasks)              // Exact match
    .on('/tasks/:id', renderTaskDetail)     // Parameter route
    .notFound(render404);                   // Fallback
```

**Route Parameters:**
```js
function renderTaskDetail(params) {
    const taskId = params.data.id;  // Access the :id parameter
    // Use taskId to fetch and display task
}
```

**The `data-navigo` Attribute:**
```html
<a href="/tasks" data-navigo>Tasks</a>
```
- Tells Navigo to intercept this link
- Prevents full page reload
- Handles navigation via router


## Part 7: Advanced Navigo Features

### Route Parameters

```js
// Single parameter
router.on('/users/:id', (params) => {
    console.log(params.data.id);  // "123" from /users/123
});

// Multiple parameters
router.on('/posts/:year/:month/:slug', (params) => {
    console.log(params.data.year);   // "2026"
    console.log(params.data.month);  // "02"
    console.log(params.data.slug);   // "my-post"
});
```

### Query Parameters

```js
router.on('/search', (params) => {
    console.log(params.params);  // { q: "javascript", type: "tutorial" }
    // From URL: /search?q=javascript&type=tutorial
});
```

### Programmatic Navigation

```js
// Navigate to a route
router.navigate('/tasks');

// Navigate with parameters
router.navigate('/tasks/42');

// Navigate with query params
router.navigate('/search', { q: 'javascript' });

// Go back
window.history.back();
```

### Named Routes

```js
router
    .on('/tasks/:id', renderTaskDetail, 'task-detail')
    .resolve();

// Navigate by name
router.navigate('task-detail', { id: 42 });
```

### Additional Resources

- [Navigo Documentation](https://github.com/krasimir/navigo)
- [MDN History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)

