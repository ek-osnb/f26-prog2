# Spring security and SPA (Single Page Applications)

## Introduction
In this exercise, you will learn how to integrate Spring Security with a single page application (SPA). You will learn how to configure Spring Security to secure you API endpoints using session cookies and CSRF protection. You will also learn how to setup the frontend to handle authentication and include CSRF tokens in API requests (for resource changing requests).

## Setup
We will be using a simple Vanilla JS frontend and a Spring Boot backend. The frontend will be served by a nginx server, which will also act as a reverse proxy to the backend. This way, we can avoid CORS issues and use cookie-based authentication without additional configuration.


## Exercise 1 - Setting up Spring Security configuration for a SPA

### Goal
Configure Spring Security to secure the API endpoints using session cookies and CSRF protection. The frontend should be able to authenticate users and access protected resources based on their roles.

### Instructions
1. Create a new Spring Boot project with the following dependencies:
    - Spring Web
    - Spring Security
2. Create a `SecurityConfig` class inside a `security` package and configure Spring Security to secure the API endpoints with session cookies and CSRF protection:
    ```java
    @Configuration
    public class SecurityConfig {
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            http
                // Enable cookie-based CSRF protection for SPAs
                .csrf(csrf -> csrf.spa())
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/api/**").authenticated()
                    .anyRequest().authenticated()
                )
                .formLogin(form -> form
                    .loginProcessingUrl("/api/login") // Endpoint for login requests
                    .successHandler((req, res, auth) -> res.setStatus(HttpServletResponse.SC_NO_CONTENT)) // 204 on success
                    .failureHandler((req, res, ex) -> res.setStatus(HttpServletResponse.SC_UNAUTHORIZED)) // 401 on failure
                )
                .logout(logout -> logout
                    .logoutUrl("/api/logout") // Endpoint for logout requests
                    .logoutSuccessHandler((req, res, auth) -> res.setStatus(HttpServletResponse.SC_NO_CONTENT)) // 204 on success
                )
                // Return 401 instead of redirecting to /login
                .exceptionHandling(eh -> eh
                        .authenticationEntryPoint((req, res, ex) -> res.sendError(HttpServletResponse.SC_UNAUTHORIZED))
                );
            return http.build();
        }
    }
    ```
3. Inside the `SecurityConfig` class we need a PasswordEncoder bean, and a UserDetailsService bean:
    ```java
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        // TEST USERS
        UserDetails user = User.builder()
                .username("user")
                .password(passwordEncoder.encode("pw"))
                .roles("USER")
                .build();
        UserDetails admin = User.builder()
                .username("admin")
                .password(passwordEncoder.encode("pw"))
                .roles("USER", "ADMIN")
                .build();
        return new InMemoryUserDetailsManager(user, admin);
    }
    ```

## Exercise 2 - CSRF generation and user data

### Goal
Learn how to generate CSRF tokens and returning an authenticated user, which is neccesary for the frontend to be able to display the logged in user's information.

### Instructions
1. Create a new API endpoint `/api/user`:
    ```java
    @RestController
    public class UserController {
        public record UserResponse(String username, List<String> roles) {}

        @GetMapping("/api/user")
        public UserResponse getUser(Authentication authentication) {
            String username = authentication.getName();
            List<String> roles = authentication.getAuthorities().stream()
                    .map(GrantedAuthority::getAuthority)
                    .collect(Collectors.toList());
            return new UserResponse(username, roles);
        }
    }
    ```
    > **This will return different responses depending on if the user is authenticated or not.**
    > 
    > **If the use is authenticated:**
    > - A JSON object with the username and roles of the authenticated user will be returned.
    >
    > **If the user is not authenticated:**
    >
    > - A `401 Unauthorized` response will be returned, and if there is no CSRF token in the request, a CSRF token will be generated and sent in a cookie in the response. The frontend can then read this token from the cookie and include it in the headers of subsequent requests that require CSRF protection (e.g. POST, PUT, DELETE requests).

## Exercise 3 - SPA Setup

### Goal
Integrate the Spring Security configuration with a simple SPA frontend. The frontend should be able to authenticate users, display the logged in user's information, and include CSRF tokens in API requests.

### Instructions
1. Create a new directory named `frontend` in the root of the your Spring Boot project, with the following structure:
    ```
    frontend/
    ├── src/
    │   ├── index.js
    │   └── app.js
    ├── nginx
    │   └── nginx.conf
    └── compose.yml
    ```
2. Configure the `nginx.conf` file to serve the frontend and proxy API requests to the Spring Boot backend:
    ```nginx
    server {
        listen 80;

        location / {
            root /usr/share/nginx/html;
            index index.html;
            try_files $uri $uri/ /index.html; # For SPA routing
        }

        location /api/ {
            proxy_pass http://backend:8080; # Proxy API requests to the backend service
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    ```
3. Configure the `compose.yml` file to build and run the nginx server:
    ```yaml
    services:
      nginx:
        image: nginx:latest
        ports:
          - "80:80"
        volumes:
          - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
          - ./src:/usr/share/nginx/html:ro
    ```
4. Start the container using Docker compose:
    ```bash
    docker compose up -d
    ```

5. The frontend should now be running at `http://localhost`, and the backend should be running at `http://localhost/api/**`.

6. Create a `styles.css` in the `/frontend/src` directory with  with the following content:
    ```css
    form {
    display: flex;
    flex-direction: column;
    width: 300px;
    margin: 0 auto;
    }
    label {
        margin-top: 10px;
    }
    input {
        padding: 8px;
        margin-top: 5px;
    }
    button {
        margin-top: 15px;
        padding: 10px;
    }
    #message {
        margin-top: 20px;
        text-align: center;
        font-weight: bold;
    }
    .hidden {
        display: none;
    }

    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        border-radius: 10px;
        background-color: #f0f0f0;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        max-width: 400px;
        margin: 50px auto;
    }
    ```
7. Create an `index.html` file in the `/frontend/src` that loads the `styles.css` and `app.js`, and add the following form:
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>SPA with Spring Security</title>
        <link rel="stylesheet" href="styles.css">
        <script type="module" src="app.js" defer></script>
    </head>
    <body>
        <form id="login-form" class="hidden">
            <h1>Login</h1>
            <label for="username">Username:</label>
            <input type="text" id="username" name="username" placeholder="Username" required>
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" placeholder="Password" required>
            <button type="submit">Login</button>
            <div id="message" style="padding-bottom: 1em"></div>
        </form>
        <section id="user-info" class="container hidden"></section>
    </body>
    </html>
    ```

6. Create an `app.js` file in the `/frontend/src` directory with the following content:
    ```javascript
    document.addEventListener("DOMContentLoaded", initApp);

    const BASE_URL = ""; // nginx proxies requests, so relative URLs are enough

    const UI_ELEMENTS = {
        loginForm: null,
        messageDiv: null,
        userInfo: null
    };


    function initApp() {
        setupUi();
        showLoginForm();
    }

    function setupUi() {
        UI_ELEMENTS.loginForm = document.getElementById("login-form");
        UI_ELEMENTS.messageDiv = document.getElementById("message");
        UI_ELEMENTS.userInfo = document.getElementById("user-info");

        UI_ELEMENTS.loginForm.addEventListener("submit", handleLogin);
    }

    function hideLoginForm() {
        UI_ELEMENTS.loginForm.classList.add("hidden");
    }

    function showLoginForm() {
        UI_ELEMENTS.loginForm.classList.remove("hidden");
    }

    async function handleLogin(event) {
        event.preventDefault();

        const form = event.target;
        const formData = new FormData(form);
        const username = formData.get("username");
        const password = formData.get("password");

        try {
            const response = await fetch(`${BASE_URL}/api/login`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/x-www-form-urlencoded",
                },
                body: new URLSearchParams({ username, password })
            });

            if (!response.ok) {
                throw new Error("Login failed. Please check your credentials.");
            }

            form.reset();
            setDisplayMessage("Login successful!", true);
            // PLACEHOLDER #1: After successful login

        } catch (error) {
            setDisplayMessage(error.message, false);
        }
    }

    function setDisplayMessage(text, isSuccess) {
        UI_ELEMENTS.messageDiv.textContent = text;
        UI_ELEMENTS.messageDiv.style.color = isSuccess ? "green" : "red";
    }

    function clearDisplayMessage() {
        UI_ELEMENTS.messageDiv.textContent = "";
    }
    ```
7. Try to login with the credentials `user/pw` or `admin/pw`. You should see an error message! This is because we are not including the CSRF token in the login request:
    ```javascript
    function getCsrfToken() {
        const match = document.cookie.match(/XSRF-TOKEN=([^;]+)/);
        return match ? decodeURIComponent(match[1]) : null;
    }
    ```
    > The above function reads the CSRF token from the cookie. We can then include this token in the headers of our login request:
    ```javascript
    // inside handleLogin function
    const csrfToken = getCsrfToken();
    const response = await fetch(`${BASE_URL}/api/login`, {
        method: 'POST',
        headers: {
            "Content-Type": "application/x-www-form-urlencoded",
            "X-XSRF-TOKEN": csrfToken // Include the CSRF token in the header
        },
        body: new URLSearchParams({ username, password })
    });
    // ... rest of the code
    ```
8. Try to refresh the page and login again. This time it should work, and you should see a success message!


## Exercise 4 - Displaying logged in user's information

### Goal
Use the `/api/user` endpoint to fetch and display the logged in user's information in the frontend.

### Instructions
1. Create a function that fetches user information from the `/api/user` endpoint:
    ```javascript
    async function fetchUserInfo() {
        const response = await fetch(`${BASE_URL}/api/user`);
        if (!response.ok) {
            throw new Error("Failed to fetch user info");
        }
        return await response.json();
    }
    ```
2. Create a function that displays userinfo, and one that hides it:
    ```javascript
    function showUserInfo(user) {
        UI_ELEMENTS.userInfo.innerHTML = "";

        const article = document.createElement("article");
        article.style.padding = "1em";

        const usernamePtag = document.createElement("p");
        usernamePtag.textContent = `Logged in as: ${user.username}`;

        const rolesPtag = document.createElement("p");
        rolesPtag.textContent = `Roles: ${user.roles.join(", ")}`;

        const logoutButton = document.createElement("button");
        logoutButton.textContent = "Logout";
        logoutButton.addEventListener("click", handleLogout);

        article.appendChild(usernamePtag);
        article.appendChild(rolesPtag);
        article.appendChild(logoutButton);
        UI_ELEMENTS.userInfo.appendChild(article);

        UI_ELEMENTS.userInfo.classList.remove("hidden");
    }

    function hideUserInfo() {
        UI_ELEMENTS.userInfo.innerHTML = "";
        UI_ELEMENTS.userInfo.classList.add("hidden");
    }
    ```
3. Create a function that handles logout:
    ```javascript
    async function handleLogout() {
        try {
            const csrfToken = getCsrfToken();

            const response = await fetch(`${BASE_URL}/api/logout`, {
                method: "POST",
                headers: {
                    "X-XSRF-TOKEN": csrfToken || ""
                }
            });

            if (!response.ok) {
                throw new Error("Logout failed.");
            }

            // Hide user info and show login form again
            hideUserInfo();
            showLoginForm();
            setDisplayMessage("Logged out successfully.", true);

            // PLACEHOLDER #2: After successful logout
            
        } catch (error) {
            setDisplayMessage(error.message, false);
        }
    }
    ```
4. After a successful login, fetch the user info and display it - replace the `// PLACEHOLDER #1: After successful login` with the following code:
    ```javascript
    // TODO #1
    const userInfo = await fetchUserInfo();
    showUserInfo(userInfo);
    hideLoginForm();
    ```
5. Try refreshing the page. Log in again, and you should see the logged in user's information displayed, along with a logout button. Try clicking the logout button, and you should be logged out and see the login form again.


## Exercise 5 - Fix refreshing the page and losing the logged in state

### Goal
When we refresh the page, we lose the logged in state and have to log in again. This is because we are not checking if the user is already authenticated when the app starts. We can fix this by calling the `/api/user` endpoint on app startup to check if the user is already authenticated and also to trigger CSRF token generation.

### Instructions
1. Inside the `initApp()` replace the content with the following code:
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
        } catch (error) {
            // This is the normal path for anonymous users on first page load
            showLoginForm();
            hideUserInfo();
            clearDisplayMessage();
        }
    }
    ```
2. Now try refreshing the page after logging in. You should stay logged in and see your user info, instead of being logged out.


## Exercise 6 - Fixing missing CSRF token after logout

### Goal
After logging out, if you try to log in again without refreshing the page, you will get a 403 error. This is because Spring Security invalidates the CSRF token on logout, by removing the cookie. We can fix this by making a request to the `/api/user` endpoint after logout to trigger CSRF token generation.

### Instructions
1. Inside the `handleLogout()` function, after a successful logout, replace `// PLACEHOLDER #2: After successful logout` with the following code to fetch the user info, which will trigger CSRF token generation:
    ```javascript
    // PLACEHOLDER #2: After successful logout
    try {
        await fetchUserInfo();
    } catch {
        // Expected to fail with 401,
        // but this will trigger CSRF token generation
        // for the next login attempt
    }
    ```
2. Now try logging out and then logging in again without refreshing the page. You should be able to log in successfully without getting a 403 error.


## Reflection questions
- Why do we need to fetch the user info on app startup to handle page refreshes?
- What would happen if we didn't trigger CSRF token generation after logout?
- How would you handle role-based access control in the frontend? For example, showing/hiding certain UI elements based on the user's roles?