# Exercise 01: Value Objects with @Embeddable

## Learning Objective

Learn how to use `@Embeddable` and `@Embedded` to create value objects that encapsulate related fields within an entity.

## Background

Value objects are classes that represent a concept or measurement but don't have their own identity. They're stored in the same table as the owning entity.

In our movie database, instead of storing a simple rating number, we want to store both a **score** (e.g., 8.5) and the **number of votes** (e.g., 15000). These two values belong together and form a "Rating" concept.

## Your Task

Create a `Rating` value object and embed it in the `Movie` entity.

### Step 1: Create the Rating Class

Create a new class `Rating.java` in the `entity` package:

```java
package com.example.moviedb.entity;

import jakarta.persistence.Embeddable;

@Embeddable
public class Rating {
    private Double score;      // e.g., 8.5
    private Integer voteCount; // e.g., 15000

    // TODO: Add default constructor
    
    // TODO: Add constructor with parameters
    
    // TODO: Add getters and setters
}
```

**Requirements:**
- The class must be annotated with `@Embeddable`
- Add a default constructor (required by JPA)
- Add a constructor that takes `score` and `voteCount`
- Add getters and setters for both fields

### Step 2: Embed Rating in Movie Entity

Modify the `Movie` entity to include the `Rating`:

```java
@Entity
public class Movie {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    private Integer releaseYear;
    private String genre;

    // TODO: Add the embedded Rating field here
    
    // Remember to update constructors if needed
    // Add getter and setter for rating
}
```

**Hint:** Use the `@Embedded` annotation on the rating field.

## Testing Your Implementation

1. Restart your application (the database will be recreated)

2. Create a movie with a rating (using .http in IntelliJ):

    ```http
    ### CREATE MOVIE
    POST http://localhost:8080/api/movies
    Content-Type: application/json

    {
        "title": "The Shawshank Redemption",
        "releaseYear": 1994,
        "genre": "Drama",
        "rating": {
            "score": 9.3,
            "voteCount": 2500000
        }
    }
    ```

3. Retrieve the movie:

    ```http
    ### GET MOVIES BY ID
    GET http://localhost:8080/api/movies/1
    ```

You should see the rating embedded in the movie JSON response.

4. Check the H2 console at `http://localhost:8080/h2-console` and verify that the `MOVIE` table has columns `score` and `vote_count` (not a separate table!).

## Expected Result

The `Rating` fields should be stored directly in the `MOVIE` table as separate columns. When you retrieve a movie, the rating should appear as a nested object in the JSON.

## Questions to Consider

1. Why doesn't `Rating` need an `@Id` field?
2. What happens in the database? How many tables are created?
3. When would you use `@Embeddable` vs creating a separate entity?

Continue to **Exercise 02** to add relationships between entities!
