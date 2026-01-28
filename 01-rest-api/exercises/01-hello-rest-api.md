# Hello REST API with Spring Boot

## Before you start

In IntelliJ, create a new **Spring Boot 4** project that uses **JDK25** and Java 25 with the only dependency being **`Spring Web`**.

## Exercise 1 - Hello REST API

**Goal:** Understand what a REST endpoint is, what JSON looks like, and how Spring serializes objects.

1. Create a new class named `HelloController`, and add the `GET /api/hello` endpoint that returns JSON:
    ```json
    {
        "message": "Hello, World!",
        "timestamp": "2026-01-28T10:00:00"
    }
    ```
    *Replace the timestamp value with the current date and time, by using `LocalDateTime.now()`.*
    
    **Hint:** Use a `Map<String, Object>` to create the JSON response.

2. Test the endpoint using your browser or Postman by navigating to `http://localhost:8080/api/hello`.

3. In the same class, add another GET endpoint `/api/greet/{name}` that takes a path variable `name` and returns a JSON response:
    ```json
    {
        "message": "Hello, {name}!",
        "timestamp": "2026-01-28T10:00:00"
    }
    ```
    *Replace `{name}` with the actual name provided in the URL and the timestamp value with the current date and time.*

4. Test the endpoint using your browser or Postman by navigating to `http://localhost:8080/api/greet/YourName`. Replace `YourName` with any name you choose.

5. Add a `GET /api/echo?text=...` returning the text provided as a query parameter:
    ```json
    {
        "echo": "Your text here"
    }
    ```
    *Replace `Your text here` with the actual text provided in the query parameter and the timestamp value with the current date and time.*


6. Test the endpoint using your browser or Postman by navigating to `http://localhost:8080/api/echo?text=Hello`.

7. Instead of using a `HashMap`, create a new class named `GreetingResponse` with fields `message` and `timestamp`. Modify the `/api/hello` and `/api/greet/{name}` endpoints to return instances of `GreetingResponse` instead of a `HashMap`.