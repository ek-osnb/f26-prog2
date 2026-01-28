# CRUD REST API with Spring Boot

**Prerequisites:** You should have completed Exercise 02 (Modelling an API) where you created a Book API with nested Author and Publisher objects.

## Starting Point

You should already have:
- Model classes: `Book`, `Author`, and `Publisher`
- `BookController` with:
  - `GET /api/books` - Returns list of books
  - `GET /api/books/{id}` - Returns single book or 404

Now you will extend this API with CREATE, UPDATE, and DELETE operations.

## Exercise 1 - Bonus: Customize JSON output order
**Goal:** Learn how to customize the order of fields in the JSON response.

1. In the `Book` class, use a Jackson annotation to specify the order of fields in the JSON output so that `id` comes first, followed by `title`, `isbn`, `publishedYear`, `edition`, `author`, and then `publisher`.
    - **Hint:** Use the `@JsonPropertyOrder({"id", "title", "isbn", "publishedYear", "edition", "author", "publisher"})` annotation on the `Book` class.
2. Test the `/api/books` and `/api/books/{id}` endpoints again to verify that the JSON output reflects the specified field order.
3. (Optional) Apply similar ordering to the `Author` and `Publisher` classes.
4. Test the endpoints again to verify the changes.

## Exercise 2 - POST, PUT, DELETE Operations
**Goal:** Fully implement CRUD operations for the `Book` resource.

### 2.1 - POST endpoint
1. Create a new endpoint `POST /api/books` in the `BookController` that accepts a `Book` object in the request body and returns the same book with a `201 Created` status code. The ID should be auto-generated (you can simulate this by incrementing a static variable).
    - **Hint:** Use the `@RequestBody` annotation to bind the incoming JSON to a `Book` object.
    - **Hint:** Use `@PostMapping` for POST endpoints.
    - **Hint:** Use `AtomicLong` or `Long` for generating unique IDs.
2. Test the endpoint using Postman by sending a `POST` request to `http://localhost:8080/api/books` with a JSON body representing a new book (without an ID). Verify that the response contains the created book with an assigned ID.
3. Add a `Location` header to the response that points to the URL of the newly created book (e.g., `/api/books/{id}`).
    - **Hint:** You can use `ResponseEntity.created(URI.create("/api/books/" + book.getId())).body(book)` to set the `Location` header.

### 2.2 - PUT endpoint
1. Create a new endpoint `PUT /api/books/{id}` in the `BookController` that accepts a `Book` object in the request body and updates the existing book with the specified ID.
2. If the book with the specified ID does not exist, return a `404 Not Found` response.
3. Test the endpoint using Postman by sending a `PUT` request to `http://localhost:8080/api/books/{id}` with a JSON body representing the updated book data. Verify that the response contains the updated book.

### 2.3 - DELETE endpoint
1. Create a new endpoint `DELETE /api/books/{id}` in the `BookController` that deletes the book with the specified ID and returns a `204 No Content` status code.
2. If the book with the specified ID does not exist, return a `204 No Content` response.
3. Test the endpoint using Postman by sending a `DELETE` request to `http://localhost:8080/api/books/{id}`. Verify that the response status code is `204 No Content`.

## Exercise 3 - Testing Your Complete API

Now that you have a full CRUD API, test all operations in sequence:

1. **GET** `/api/books` - Verify you see your initial 3+ books
2. **POST** `/api/books` - Add a new book (e.g., "Clean Code" by Robert C. Martin)
3. **GET** `/api/books` - Verify the new book appears in the list
4. **GET** `/api/books/{id}` - Retrieve the specific book you just created
5. **PUT** `/api/books/{id}` - Update the book (e.g., change the edition)
6. **GET** `/api/books/{id}` - Verify the update was successful
7. **DELETE** `/api/books/{id}` - Delete the book
8. **GET** `/api/books/{id}` - Verify you get a 404 Not Found