# Session 1 Exercises — Spring Security Foundations

## Overview

In this session, you will build a basic Spring Boot + Thymeleaf application secured with Spring Security.

You will practice:

* understanding authentication and authorization
* configuring form login
* protecting routes
* hashing passwords
* adding roles and testing access rules

---

## Exercise 1 — Observe Default Spring Security

### Goal

See what Spring Security does out of the box.

### Tasks

1. Create a new Spring Boot project with:

   * `Spring Web`
   * `Spring Security`
   * `Thymeleaf`
2. Add a controller with a home endpoint:

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

3. Create a simple `home.html` in the `src/main/resources/templates` directory.
4. Run the app and open `/` in the browser.
5. Notice that Spring Security redirects you to a login page.
6. Try logging in with the default credentials:
   * **username**: `user`
   * **password**: printed in the console when the app starts

### Questions

* Did you create a login page yourself?
* Why is access to `/` blocked?
* What does this tell you about the default behavior of Spring Security?

---

## Exercise 2 - Configure Spring Security to Allow Public Access to Home

### Goal
Make the home page accessible without logging in.

### Tasks
1. Create a `SecurityConfig` class.
    * Annotate it with `@Configuration` so that Spring picks it up.
2. Add a `SecurityFilterChain` bean that:
   * permits access to `/`
   * requires authentication for all other routes

    **HINT:**
    ```java
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());

        return http.build();
    }
    ```
3. Restart the app and refresh the home page. It should now be accessible without logging in.

### Questions
* What changed after adding the security config?
* Why is the home page now public?
* What would happen if we removed the `requestMatchers("/")` line?

---

## Exercise 3 - Create a new endpoint

### Goal
Add a new endpoint that requires authentication.

### Tasks
1. Create a new `html` page called `protected.html` with some *"secret content"*.
2. Add a new controller method that serves this page at `/protected`.
3. Try to access `/protected` in the browser.
4. Notice that you are redirected to the login page.
5. Log in with the default credentials and access `/protected` again.
6. Notice that you can now see the secret content.


---


## Exercise 4 — Create an In-Memory User

### Goal

Define your own user instead of using the default one.

### Tasks

1. Add a `PasswordEncoder` bean using BCrypt inside the `SecurityConfig` class.

    **HINT:**
    ```java
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    ```

2. Add a `UserDetailsService` bean with one user:
   * username: `user`
   * password: `pw`
   * role: `USER`

    **HINT:**
    ```java
    @Bean
    public UserDetailsService users(PasswordEncoder encoder) {
        UserDetails user = User.builder()
            .username("user")
            .password(encoder.encode("pw"))
            .roles("USER")
            .build();

        return new InMemoryUserDetailsManager(user);
    }
    ```
3. Restart the app and log in with the new credentials.
4. You should be able to access the protected page with the new user.


### Questions

* Why do we call `encoder.encode(...)`?
* Is the original password stored directly?
* What problem does password hashing solve?

---

## Exercise 5 - Adding roles and an admin page

### Goal
Add a new user with a different role and protect an admin page.

### Tasks
1. Add a second user to your `UserDetailsService` bean:
   * username: `admin`
   * password: `pw`
   * role: `ADMIN`

    **HINT:** Use the same pattern as the first user and add both users to the `InMemoryUserDetailsManager`:
    ```java
    @Bean
    public UserDetailsService users(PasswordEncoder encoder) {
        UserDetails user = User.builder()
            .username("user")
            .password(encoder.encode("pw"))
            .roles("USER")
            .build();
        UserDetails admin = User.builder()
            .username("admin")
            .password(encoder.encode("pw"))
            .roles("ADMIN")
            .build();
        return new InMemoryUserDetailsManager(user, admin);
    }
    ```

2. Create a new `admin.html` page with some *"admin content"*.
3. Add a new controller method that serves this page at `/admin`.
4. Update your security config so that:
   * `/` does not require authentication
   * `/protected` requires role `USER` or `ADMIN`
   * `/admin` requires role `ADMIN`

    **HINT:**
    ```java
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/").permitAll()
                .requestMatchers("/protected").hasAnyRole("USER", "ADMIN")
                .requestMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());

        return http.build();
    }
    ```
5. Test both users and verify that:
   * `user` can access `/protected` but not `/admin`
   * `admin` can access both `/protected` and `/admin`
   * unauthenticated users cannot access either `/protected` or `/admin`
   * All users can access `/`

### Questions
* Is logging in enough to access every page?
* What is the difference between being authenticated and being authorized?
* Why are roles useful?

---

## Exercise 6 - Show the logged-in user in the UI

### Goal
Use the current authentication information in the Thymeleaf templates.

### Background
Thymeleaf has built-in support for accessing authentication information in the templates. This allows you to show the logged-in user's name and conditionally display content based on their roles. To make this work, you need to make sure that you have the Thymeleaf extras Spring Security dependency in your project:

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>
```
> **Note:** Although it says "springsecurity6", this dependency works with Spring Security 7 as well.

You need to include the namespace in your Thymeleaf templates to use the security features:

```html
<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```
> This allows you to use the `sec` namespace for security-related attributes.

The thymeleaf extras can be used to access the current authentication information and check user roles directly in the templates. Here are some examples of how to use it:

```html
<p th:text="${#authentication.name}"></p>
<div sec:authorize="hasRole('ADMIN')">
    <p>This content is only visible to users with the ADMIN role.</p>
</div>
<div sec:authorize="isAnonymous()">
    <p>This content is only visible to unauthenticated users.</p>
</div>
<div sec:authorize="isAuthenticated()">
    <p>This content is only visible to authenticated users.</p>
</div>
<div sec:authorize="hasAnyRole('USER', 'ADMIN')">
    <p>This content is visible to users with either USER or ADMIN role.</p>
</div>
```

### Tasks
1. Edit `home.html`.
2. Display the current username using Thymeleaf.
    * The user's name should only be visible when they are logged in.
    * Optionally display different content for logged-in users and anonymous users. I.e. 
    * Show a welcome message to logged-in users.
    * Show a prompt to log in for anonymous users.

    **HINT:**
    ```html
    <html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
    <head>
        <title>Home</title>
    </head>
    <body>
        <div sec:authorize="isAuthenticated()">
            <p>Welcome, <span th:text="${#authentication.name}">User</span>!</p>
        </div>
        <div sec:authorize="isAnonymous()">
            <p>Welcome, guest! Please log in to see more content.</p>
            <a th:href="@{/login}">Login</a>
        </div>
    </body>
    </html>
    ```
3. Restart the app and test the home page both when logged in and when not logged in.

<!-- ### Questions
* Where does Thymeleaf get authentication information from?
* Why is it useful to show the logged-in user in the UI?
* How does Thymeleaf determine which content to show for different users?
* What other security-related conditions can you check with Thymeleaf extras?
* Can you show content only to users with a specific role? -->

---

## Exercise 7 (Optional): Add a Custom Login Page

### Goal

Replace the default login page with your own Thymeleaf page.

### Tasks

1. Create a `login.html` template.
2. Inside `login.html` create a form that submits to `/login` with fields for username and password.

    **Hint:**
    ```html
    <div th:if="${param.error}">
        Invalid username or password
    </div>

    <div th:if="${param.logout}">
        You have been logged out
    </div>
    <form th:action="@{/login}" method="post">
        <input type="text" name="username" />
        <input type="password" name="password" />
        <button type="submit">Sign in</button>
    </form>
    ```
3. Create a controller method to serve the login page at `/login`.
4. Update the `SecurityFilterChain` bean to:
   * permits `/login`
   * uses your custom login page

    **Hint:**
    ```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)  {
        http
            .authorizeHttpRequests(auth -> auth
                // Keep the previous rules
            )
            .formLogin(form -> form
                    .loginPage("/login") // Need a controller
                    .loginProcessingUrl("/login") // The URL to submit the username and password to
                    .defaultSuccessUrl("/", true)
                    .failureUrl("/login?error=true")
                    .permitAll()
            )

        return http.build();
    }
    ```
5. Restart the app and test the login flow with your new page.

### Questions

* Why must `/login` be public?
* What happens after a successful login?
* What happens if the login form field names are wrong?

---


## Exercise 8 — Add Logout

### Goal

Understand the full session-based login lifecycle.

### Tasks

1. Add a logout functionality to `home.html`.
    * You can use a simple form that submits a POST request to `/logout`. Make sure to include the `sec:authorize="isAuthenticated()"` attribute so that the logout button only appears for logged-in users.
        ```html
        <form sec:authorize="isAuthenticated()" th:action="@{/logout}" method="post">
            <button type="submit">Logout</button>
        </form>
        ```
2. Restart the app and test the logout functionality.

### Questions

* What changed after logout?
* What do you think happened to the session?
* Why is logout important in stateful authentication?

---

## Exercise 9 — Reflection Questions

Write short answers to these:

1. What is the difference between authentication and authorization?
2. Why do we need sessions if HTTP is stateless?
3. What is usually stored in a cookie in a session-based application?
4. Why should passwords never be stored in plain text?
5. When should a route use `permitAll()`?
6. Why can one logged-in user access `/admin` while another cannot?

