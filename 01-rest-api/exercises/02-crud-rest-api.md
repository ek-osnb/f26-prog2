# CRUD REST API with Spring Boot

In IntelliJ, create a new **Spring Boot 4** project that uses **JDK25** and Java 25 with the only dependency being **`Spring Web`**.

## Exercise 1 - Nested JSON Response

**Goal:** Learn how to create nested JSON responses using Java classes.

1. Create a new class named `Address` with the following fields:
    - `street` (String)
    - `city` (String)
    - `zipCode` (String)
2. Create another class named `User` with the following fields:
    - `id` (Long)
    - `name` (String)
    - `email` (String)
    - `address` (Address)
3. Create a new class named `UserController` (make it a rest controller), add a new endpoint `GET /api/users/{id}` that returns a `User` object for the given `id` (the data can be hardcoded for this exercise). The JSON response should look like this:

    ```json
    {
        "id": 1,
        "name": "John Doe",
        "email": "jd@ek.dk",
        "address": {
            "street": "Guldbergsgade 29",
            "city": "Copenhagen",
            "zipCode": "2200"
            }
    }
    ```
4. Test the endpoint using your browser or Postman by navigating to `http://localhost:8080/api/users/1`.

## Exercise 2 - List of Objects in JSON Response

**Goal:** Learn how to return a list of objects in a JSON response.

1. Create a list of `User` objects in the `UserController`.

2. In the `UserController`, add a new endpoint `GET /api/users` that returns a list of `User` objects with sample data (at least 3 users), each containing a nested `Address` object.

    ```json
    [
        {
            "id": 1,
            "name": "John Doe",
            "email": "jd@ek.dk",
            "address": {
                "street": "Guldbergsgade 29",
                "city": "Copenhagen",
                "zipCode": "2200"
            }
        },
        {
            "id": 2,
            "name": "Jane Smith",
            "email": "js@ek.dk",
            "address": {
                "street": "Amagerbrogade 100",
                "city": "Copenhagen",
                "zipCode": "2300"
            }
        }
    ]
    ```
3. Test the endpoint using your browser or Postman by navigating to `http://localhost:8080/api/users`.

4. Refactor the `api/users/{id}` endpoint to retrieve the user from the list based on the provided `id`. If the user is not found, return a `404 Not Found` response.
    - **Hint:** Use a loop to find the user
    - **Hint:** Use `ResponseEntity.notFound().build()` for 404 responses

## Exercise 3 - Bonus: Customize JSON output order
**Goal:** Learn how to customize the order of fields in the JSON response.

1. In the `User` class, use a Jackson annotation to specify the order of fields in the JSON output so that `id` comes first, followed by `name`, `email`, and then `address`.
    - **Hint:** Use the `@JsonPropertyOrder({"id", "name", "email", "address"})` annotation on the `User` class.
2. Test the `/api/users` and `/api/users/{id}` endpoints again to verify that the JSON output reflects the specified field order.
3. (Optional) Apply similar ordering to the `Address` class.
4. Test the endpoints again to verify the changes.

## Exercise 4 - POST,PUT, DELETE Operations
**Goal:** Fully implement CRUD operations for the `User` resource.


### 4.1 - POST endpoint
1. Create a new endpoint `POST /api/users` in the `UserController` that accepts a `User` object in the request body and returns the same user with a `201 Created` status code. The ID should be auto-generated (you can simulate this by incrementing a static variable).
    - **Hint:** Use the `@RequestBody` annotation to bind the incoming JSON to a `User` object.
    - **Hint:** Use `AtomicLong` or `Long` for generating unique IDs.
2. Test the endpoint using Postman by sending a `POST` request to `http://localhost:8080/api/users` with a JSON body representing a new user (without an ID). Verify that the response contains the created user with an assigned ID.
3. Add a `Location` header to the response that points to the URL of the newly created user (e.g., `/api/users/{id}`).
    - **Hint:** You can use `ResponseEntity.created(URI.create("/api/users/" + user.getId())).body(user)` to set the `Location` header.

### 4.2 - PUT endpoint
1. Create a new endpoint `PUT /api/users/{id}` in the `UserController that accepts a `User` object in the request body and updates the existing user with the specified ID.
2. If the user with the specified ID does not exist, return a `404 Not Found` response.
3. Test the endpoint using Postman by sending a `PUT` request to `http://localhost:8080/api/users/{id}` with a JSON body representing the updated user data. Verify that the response contains the updated user.

### 4.3 - DELETE endpoint
1. Create a new endpoint `DELETE /api/users/{id}` in the `UserController` that deletes the user with the specified ID and returns a `204 No Content` status code.
2. If the user with the specified ID does not exist, return a `204 No Content` response.
3. Test the endpoint using Postman by sending a `DELETE` request to `http://localhost:8080/api/users/{id}`. Verify that the response status code is `204 No Content`.



