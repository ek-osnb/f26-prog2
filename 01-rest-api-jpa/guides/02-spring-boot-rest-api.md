# REST APIs with Spring Boot

## Prerequisites
- A Spring Boot project set up with `Spring Web` dependency only.

## Objectives
- Understand the basics of REST APIs.
- Create a simple REST API using Spring Boot.
- Learn about serialization and deserialization in Spring Boot.
- Test the REST API using cURL.

## What is a REST API?
A REST API (Representational State Transfer Application Programming Interface) is a set of rules and conventions for building and interacting with web services. It allows different software applications to communicate over the internet using standard HTTP methods like GET, POST, PUT, DELETE, etc. 

REST APIs are stateless, meaning each request from a client contains all the information needed to process that request. They typically return data in JSON or XML format.

`JSON` (JavaScript Object Notation) is the most commonly used format for REST web APIs due to its simplicity and ease of use. An example of a JSON response might look like this:

```json
{
  "id": 1,
  "title": "Sample Post",
  "body": "This is a sample post content.",
  "userId": 1
}
```
An example of a REST API can be found at [JSONPlaceholder](https://jsonplaceholder.typicode.com/), which provides fake online REST APIs for testing and prototyping.

We will create simple REST web APIs using Spring Boot, which you are already familiar with.

A REST API is designed around resources, which are identified by URIs (Uniform Resource Identifiers). Each resource can be manipulated using standard HTTP methods:

- `GET`: Retrieve a resource or a collection of resources.
- `POST`: Create a new resource.
- `PUT`: Update an existing resource.
- `DELETE`: Remove a resource.

## Hello World REST API In Spring Boot
In Spring Boot, we can create a simple REST API that returns a greeting message.

```java
@RestController
@RequestMapping("/api/greetings")
public class GreetingController {

    @GetMapping
    public ResponseEntity<String> getGreeting() {
        return ResponseEntity.ok("Hello, World!");
    }
}
```
In this example, we define a `GreetingController` class annotated with `@RestController`, which indicates that it is a controller where every method returns a domain object instead of a view. The `@RequestMapping` annotation specifies the base URI for all methods in this controller.

The `getGreeting` method is annotated with `@GetMapping`, which maps HTTP GET requests to this method. When a client sends a GET request to `/api/greetings`, it will receive a response with the message "Hello, World!" in the body.

To test this API, you can use tools like Postman or cURL. For example, using cURL, you can run the following command in your terminal:

```bash
curl -X GET http://localhost:8080/api/greetings
```
This should return the response:
```
Hello, World!
```

### Returning JSON Response
But this isn't a JSON response. To return a JSON response, we can modify the method to return a `Greeting` object:

```java
public class Greeting {
    private String message;

    public Greeting(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```
Now, we can modify the `getGreeting` method to return a `Greeting` object:

```java
@GetMapping
public ResponseEntity<Greeting> getGreeting() {
    Greeting greeting = new Greeting("Hello, World!");
    return ResponseEntity.ok(greeting);
}
```
With this change, when you send a GET request to `/api/greetings`, the response will be in JSON format:
```json
{
  "message": "Hello, World!"
}
```

## Serialization and Deserialization
Serialization is the process of converting an object into a format that can be easily stored or transmitted, such as JSON or XML. Deserialization is the reverse process, where the data is converted back into an object.
In Spring Boot, this is handled automatically by the Jackson library, which is included by default when you create a Spring Boot project with the `Spring Web` dependency.

When you return an object from a controller method, Spring Boot automatically serializes it to JSON. Similarly, when you send a JSON request body to a controller method, Spring Boot deserializes it into a Java object.

## Returning a more complex object
Let's create a more complex object to return from our API. We will create a `Person` and an `Address` class, and return a `Person` object that contains an `Address`.

```java
public class Address {
    private String street;
    private String city;
    private String zipCode;

    // Constructors, getters, and setters
}
```
```java
public class Person {
    private Long id;
    private String name;
    private int age;
    private Address address;

    // Constructors, getters, and setters
}
```
Now, let's create a new controller to return a `Person` object with an `Address`:

```java
@RestController
@RequestMapping("/api/persons")
public class PersonController {
    
    private List<Person> persons = new ArrayList<>();

    private Long nextId = 1L;

    public PersonController() {
        Address address = new Address("Guldbergsgade 29", "Copenhagen", "2200");
        Person person = new Person(nextId++, "osnb", 33, address);
        persons.add(person);
    }

    @GetMapping
    public ResponseEntity<List<Person>> getPersons() {
        return ResponseEntity.ok(persons);
    }
}
```
When you send a GET request to `/api/persons`, you will receive a JSON response like this:
```json
[
    {
        "id": 1,
        "name": "osnb",
        "age": 33,
        "address": {
            "street": "Guldbergsgade 29",
            "city": "Copenhagen",
            "zipCode": "2200"
        }
    }
]
```
The `PersonController` class that handles requests to `/api/persons`. The `getPersons` method is mapped to GET requests and returns a list of `Person` objects as a JSON response.

## Adding CRUD operations
You can add more CRUD operations (Create, Read, Update, Delete) to your REST API by adding more methods to your controller. 

### Post Method

```java

@PostMapping
public ResponseEntity<Person> createPerson(@RequestBody Person newPerson) {
    newPerson.setId(nextId++); // This is just for demo purposes. In a real application, IDs are usually generated elsewhere.
    persons.add(newPerson);
    return ResponseEntity.status(HttpStatus.CREATED).body(newPerson);
}
```

This method is mapped to POST requests and takes a `Person` object from the request body. It adds the new `Person` to the list and returns a response with status code 201 (Created) and the created `Person` object.

We can test this API using cURL:

```bash
curl -X POST http://localhost:8080/api/persons -H "Content-Type: application/json" -d '{
    "name": "ABC",
    "age": 30,
    "address": {
        "street": "Falkoner Alle 1",
        "city": "Frederiksberg",
        "zipCode": "2000"
    }
}'
```

### Put Method
To update an existing person, you can add a `PUT` method:

```java
@PutMapping("/{id}")
public ResponseEntity<Person> updatePerson(@PathVariable Long id, @RequestBody Person updatedPerson) {
    for (Person person : persons) {
        if (person.getId().equals(id)) {
            person.setName(updatedPerson.getName());
            person.setAge(updatedPerson.getAge());
            person.setAddress(updatedPerson.getAddress());
            return ResponseEntity.ok(person);
        }
    }
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
}
```
This method is mapped to PUT requests and takes an `id` from the **path variable** and a `Person` object from the **request body**. It updates the existing `Person` with the given `id` and returns the updated `Person` object. If the `Person` with the given `id` is not found, it returns a 404 (Not Found) response.

We can test this API using cURL:

```bash
curl -X PUT http://localhost:8080/api/persons/1 -H "Content-Type: application/json" -d '{
    "name": "DEF",
    "age": 40,
    "address": {
        "street": "Rådhuspladsen 1",
        "city": "Copenhagen",
        "zipCode": "1500"
    }
}'
```


### Delete Method
To delete a person, you can add a `DELETE` method:
```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deletePerson(@PathVariable Long id) {
    for (Person person : persons) {
        if (person.getId().equals(id)) {
            persons.remove(person);
            return ResponseEntity.noContent().build();
        }
    }
    return ResponseEntity.status(HttpStatus.NOT_FOUND).build();
}
```
This method is mapped to DELETE requests and takes an `id` from the **path variable**. It removes the `Person` with the given `id` from the list and returns a 204 (No Content) response. If the `Person` with the given `id` is not found, it returns a 404 (Not Found) response.

We can test this API using cURL:

```bash
curl -X DELETE http://localhost:8080/api/persons/1
```

## Extracting query parameters

Query parameters are often used in REST APIs to filter or sort data. And example of a query parameter is `?age=30` in the URL `http://localhost:8080/api/persons?age=30`. Here we expect to get all persons with age 30. You can use `&` to add more query parameters, like `?age=30&city=Copenhagen`.

In Spring Boot, you can extract **query parameters** from the request URL using the `@RequestParam` annotation. For example, to filter persons by age and city, you can modify the `getPersons` method like this:
```java
@GetMapping
public ResponseEntity<List<Person>> getPersonsByAgeAndCity(
        @RequestParam(required = false) Integer age,
        @RequestParam(required = false) String city) {
    
    List<Person> filteredPersons = new ArrayList<>();
    for (Person person : persons) {
        if (age != null && person.getAge() != age) {
            continue;
        }

        if (city != null && !person.getAddress().getCity().equalsIgnoreCase(city)) {
            continue;
        }
        filteredPersons.add(person);
    }
    
    return ResponseEntity.ok(filteredPersons);
}
```
In this example, the `getPersonsByAgeAndCity` method takes two optional query parameters: `age` and `city`. It filters the list of persons based on the provided parameters and returns the filtered list.

In this example, the `getPersonsByAgeAndCity`:
- `/api/persons` → returns all persons
- `/api/persons?age=30` → only persons with age 30
- `/api/persons?city=frederiksberg` → only persons living in Frederiksberg
- `/api/persons?age=30&city=frederiksberg` → only persons with age 30 and city Frederiksberg

You can test these endpoints using cURL:

```bash
curl -X GET "http://localhost:8080/api/persons?age=30&city=frederiksberg"
```

## Handling HTTP Status Codes
When building REST APIs, it's important to return appropriate HTTP status codes to indicate the result of the request. Here are some common status codes:
- `200 OK`: The request was successful.
- `201 Created`: A new resource was successfully created.
- `204 No Content`: The request was successful, but there is no content to return.
- `400 Bad Request`: The request was invalid or cannot be processed.
- `404 Not Found`: The requested resource was not found.
- `500 Internal Server Error`: An error occurred on the server.

This is the reason why we use `ResponseEntity` in our controller methods, as it allows us to control the status code along with the response body. The status codes can be set using the `ResponseEntity` methods like `ok()`, `status()`, `noContent()`, `badRequest()`, `notFound()`, and `internalServerError()`.

```java
// Status code 200 OK
return ResponseEntity.ok(persons);
// Status code 201 Created
return ResponseEntity.status(HttpStatus.CREATED).body(newPerson);
// Status code 204 No Content
return ResponseEntity.noContent().build();
// Status code 400 Bad Request
return ResponseEntity.badRequest().body("Invalid request");
// Status code 404 Not Found
return ResponseEntity.notFound().build();
// Status code 500 Internal Server Error
return ResponseEntity.internalServerError().build();
```

