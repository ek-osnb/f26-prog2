# Model a JSON REST API

**Goal:** Practice modeling nested JSON structures and creating REST endpoints.

## The JSON Structure

You are given this JSON response from a public books API:

```json
{
  "id": 1,
  "title": "Effective Java",
  "isbn": "978-0-13-468599-1",
  "publishedYear": 2018,
  "edition": 3,
  "author": {
    "id": 101,
    "name": "Joshua Bloch",
    "birthYear": 1961,
    "nationality": "American"
  },
  "publisher": {
    "name": "Addison-Wesley",
    "country": "USA",
    "founded": 1942
  }
}
```

## Your Task

Create a Spring Boot REST API that mimics this structure (in-memory, no persistence).

**Requirements:**

1. Create the model classes: `Book`, `Author`, and `Publisher`
2. Create a `BookController` with these endpoints:
   - `GET /api/books` - Returns a list of at least 3 books
   - `GET /api/books/{id}` - Returns a single book or 404 if not found
3. Use hardcoded data in your controller
4. Test your endpoints using browser/Postman

**Hints:**
- Use `@RestController` and `@RequestMapping("/api/books")` on your controller class
- Use `@GetMapping` for GET endpoints
- Use `ResponseEntity.notFound().build()` for 404 responses
- Store books in a `List<Book>` in your controller