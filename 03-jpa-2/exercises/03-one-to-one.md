# Exercise 03: One-to-One Relationship

## Learning Objective

Learn how to implement a bidirectional `@OneToOne` relationship between `Movie` and `MovieDetails` entities.

## Background

A movie has exactly one set of details (plot, budget, runtime, production company), and those details belong to exactly one movie. This is a one-to-one relationship.

In a **bidirectional** relationship, both entities reference each other. One side must be marked as the **owner** of the relationship (the side with the foreign key).

## Your Task

Create a bidirectional one-to-one relationship between `Movie` and `MovieDetails`, then add an endpoint to attach details to a movie.

### Step 1: Add Relationship to Movie Entity

Modify `Movie.java` to reference `MovieDetails`:

```java
@Entity
public class Movie {
    // ... existing fields (id, title, releaseYear, genre, rating, actors)

    // TODO: Add @OneToOne relationship
    // Use @JoinColumn(name = "movie_details_id")
    // This makes Movie the owner of the relationship
    
    // TODO: Add getter and setter for movieDetails
}
```

### Step 2: Add Reverse Relationship to MovieDetails Entity

Modify `MovieDetails.java` to reference back to `Movie`:

```java
@Entity
public class MovieDetails {
    // ... existing fields (id, plot, budget, runtime, productionCompany)

    // TODO: Add @OneToOne(mappedBy = "movieDetails")
    // mappedBy indicates this is NOT the owner side
    
    // TODO: Add getter and setter for movie
}
```

### Step 3: Add Method to MovieService

Add a method to attach details to a movie:

```java
public Movie addDetailsToMovie(Long movieId, MovieDetails details) {
    // TODO: Find the movie by ID
    // TODO: Save the details first using MovieDetailsRepository
    // TODO: Set the details on the movie
    // TODO: Set the movie reference on details (for bidirectional consistency)
    // TODO: Save and return the updated movie
}
```

**Hint:** You'll need to inject `MovieDetailsRepository` into the service.

### Step 4: Add Controller Endpoint

Add an endpoint to `MovieController.java`:

```java
@PostMapping("/{id}/details")
public ResponseEntity<Movie> addDetailsToMovie(
        @PathVariable Long id,
        @RequestBody MovieDetails details) {
    // TODO: Call the service method
    // TODO: Return the updated movie
}
```

## Step 4: Use `@JsonManagedReference` and `@JsonBackReference`
To prevent infinite recursion during JSON serialization, modify the `Movie` and `MovieDetails` entities, adding `@JsonManagedReference` and `@JsonBackReference` annotations.


## Testing Your Implementation

1. Restart your application

2. Create a movie (using .http in IntelliJ):

    ```http
    ### ADD MOVIE
    POST http://localhost:8080/api/movies
    Content-Type: application/json

    {
      "title": "The Dark Knight",
      "releaseYear": 2008,
      "genre": "Action",
      "rating": {"score": 9.0, "voteCount": 2500000}
    }
    ```

3. Add details to the movie:

    ```http
    ### ADD MOVIE DETAILS
    POST http://localhost:8080/api/movies/1/details
    Content-Type: application/json

    {
      "plot": "Batman faces the Joker in a battle for Gotham soul.",
      "budget": 185000000,
      "runtime": 152,
      "productionCompany": "Warner Bros"
    }
    ```

4. Get the movie and verify it includes the details:

    ```http
    ### VERIFY MOVIE WITH DETAILS
    GET http://localhost:8080/api/movies/1
    ```

5. Check the H2 console:
   - Verify the `MOVIE` table has a `movie_details_id` foreign key column
   - Verify the `MOVIE_DETAILS` table does NOT have a foreign key to `MOVIE`

## Expected Result

The movie JSON should include the `movieDetails` object. The foreign key should be in the `MOVIE` table (owner side), not in the `MOVIE_DETAILS` table.

## Questions to Consider

1. Which entity is the owner of the relationship? How can you tell?
2. Why do we need `mappedBy` in the `MovieDetails` entity?
3. What happens if you try to save a movie with details without saving the details first?

Continue to **Exercise 04** to learn about cascading operations!
