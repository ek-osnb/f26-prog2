# CRUD REST API with Spring Boot

## Before you start

<!-- In IntelliJ, create a new **Spring Boot 4** project that uses **JDK25** and Java 25 with the only dependency being **`Spring Web`**. -->

In this exercise you will create a simple REST API for managing a to-do list using Spring Boot. The following endpoints will be implemented:
- `GET /api/todos` - Retrieve all to-do items
- `GET /api/todos/{id}` - Retrieve a specific to-do item by ID
    - If the to-do item with the specified ID does not exist, return a `404 Not Found` response, by throwing a NotFoundException.
- `POST /api/todos` - Create a new to-do item
    - The repsonse should return a `201 Created` status code along with the created to-do item in the response body.
- `PUT /api/todos/{id}` - Update an existing to-do item by ID
    - If the to-do item with the specified ID does not exist, return a `404 Not Found` response, by throwing a NotFoundException.
- `DELETE /api/todos/{id}` - Delete a to-do item by ID and return a `204 No Content` status code.
- `PATCH /api/todos/{id}/completed` - Mark a to-do item as completed
    - If the to-do item with the specified ID does not exist, return a `404 Not Found` response, by throwing a NotFoundException (should be done in the service layer).

To do this exercise, fork and clone the GitHub repository:
<!-- TODO: Add repo link -->

Open the project in IntelliJ and complete the tasks below.

The project contains a list of REST API tests located in `src/test/java/.../TodoControllerTest.java`. You can run these tests to verify that your implementation is correct.

<!-- You should only modify the `TodoController.java` for this exercise. -->

## Tests
The tests cover all the endpoints you will implement. The tests are already defined, before you start coding. It is not fully TDD (Test Driven Development) - but it will give you an idea, how tests can drive development.You can run the tests at any time to check your progress.

The tests should be *"self documenting"*, meaning that by reading the test methods you should be able to understand what each endpoint is supposed to do.

You should be familiar with `MockMvc`, but here we use `MockMvcTester` which is a newer and more fluent way to write tests for Spring MVC applications. You can still use `MockMvc` if you prefer that, but i would recommend trying out `MockMvcTester`. You can check out the documentation [here](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#test-mvc-testclient) for more information.

## `.http` files
In the `src/test/resources/http` directory, you will find `.http` files that contain example HTTP requests for testing the API manually using the IntelliJ HTTP client. 

At the moment the **service layer** and **repository layer** are not implemented, so the requests will not work. Later you will implement persistence using Spring Data JPA, and then you can use these files to test the API manually.

## Exercise 1 - Understand the provided code

### The model
Look at the `Todo` class. This class represents a to-do item with fields for `id`, `title`, `description`, and `completed` status.

### Exception handling
Look through the exceptions package to understand how custom exceptions are defined for handling not found and bad request scenarios. You should be familiar with `@ControllerAdvice`. Here we use `@RestControllerAdvice` which is similar but specifically for REST controllers.

The error reponses follow the [Problem Details for HTTP APIs (RFC 7807)](https://datatracker.ietf.org/doc/html/rfc7807) standard.

## Exercise 2 - Implement the CRUD REST API
Implement the CRUD operations in the `TodoController` class to make all the tests pass in `TodoControllerMockMvcTest.java`.

## Exercise 3 - Implement the service interface
Implement the `TodoService` interface in a new class called `TodoServiceImpl`. 
The `TodoServiceImpl` should use the `TodoRepository` interface (which is not implemented) to perform the CRUD operations.

## Exercise 4 - Implement the repository layer using Spring Data JPA
The neccesary dependencies for Spring Data JPA and H2 database are already included in the `pom.xml` file, but commented out. Uncomment them to use Spring Data JPA.

In `application.properties`, configure the H2 database connection and JPA settings as needed.

Make the `Todo` class a JPA entity by adding the necessary annotations.
Make the `TodoRepository` a Spring Data JPA repository by extending the appropriate interface.

## Exercise 5 - Create initial data using CommandLineRunner
Create a class that implements `CommandLineRunner` to insert some initial to-do items into the database when the application starts.

Test the application by running it and using the provided `.http` files to send requests to the API.

## Exercise 6 - Test using Postman or curl or .http files
Manually test the API endpoints and verify that they work as expected. You should be able to use all the CRUD operations on to-do items.

Make sure you try all the tools (Postman, curl, and IntelliJ HTTP client with `.http` files) to get familiar with them.




