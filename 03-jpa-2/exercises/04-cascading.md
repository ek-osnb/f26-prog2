# Exercise 04: Cascade Operations

## Learning Objective

Learn how to use `CascadeType` to automatically propagate operations from parent entities to child entities.

## Background

Currently, when you create a movie with details, you need to explicitly save the `MovieDetails` first. With **cascading**, JPA can automatically save related entities when you save the parent.

This is particularly useful for **composition** relationships where the child entity doesn't make sense without the parent (like `MovieDetails` without a `Movie`).

## Your Task

Add cascade operations to automatically persist and remove `MovieDetails` when working with `Movie`.

### Step 1: Add CascadeType.PERSIST to Movie Entity

Modify the `@OneToOne` annotation in `Movie.java`:

```java
@Entity
public class Movie {
    // ... existing fields

    // TODO: Modify @OneToOne to add cascade = CascadeType.PERSIST
    // Keep the @JoinColumn annotation
    @OneToOne // Add cascade here
    @JoinColumn(name = "movie_details_id")
    private MovieDetails movieDetails;
    
    // ... rest of the class
}
```

### Step 2: Simplify the Service Method

Modify `MovieService.addDetailsToMovie()` to remove the explicit save of details:

```java
public Movie addDetailsToMovie(Long movieId, MovieDetails details) {
    Movie movie = movieRepository.findById(movieId)
            .orElseThrow(() -> new RuntimeException("Movie not found"));
    
    // TODO: Remove the line that saves details to MovieDetailsRepository
    // With CascadeType.PERSIST, this is no longer needed!
    
    movie.setMovieDetails(details);
    details.setMovie(movie);
    
    return movieRepository.save(movie);
}
```

### Step 3: Test CascadeType.PERSIST

Test that details are automatically persisted:

```bash
### ADD MOVIE
POST http://localhost:8080/api/movies
Content-Type: application/json

{
  "title": "The Dark Knight",
  "releaseYear": 2008,
  "genre": "Action",
  "rating": {"score": 9.0, "voteCount": 2500000}
}

### ADD MOVIE DETAILS
POST http://localhost:8080/api/movies/1/details
Content-Type: application/json

{
  "plot": "Batman faces the Joker in a battle for Gotham soul.",
  "budget": 185000000,
  "runtime": 152,
  "productionCompany": "Warner Bros"
}

### VERIFY MOVIE WITH DETAILS
GET http://localhost:8080/api/movies/1
```

### Step 4: Add CascadeType.REMOVE (Optional Challenge)

Modify the cascade to also remove details when a movie is deleted:

```java
@OneToOne(cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
@JoinColumn(name = "movie_details_id")
private MovieDetails movieDetails;
```

Test the cascade delete:

```http
### OPTIONAL: DELETE MOVIE WITH CASCADE
DELETE http://localhost:8080/api/movies/1
```

## Expected Result

- With `CascadeType.PERSIST`: Saving a movie automatically saves its details
- With `CascadeType.REMOVE`: Deleting a movie automatically deletes its details

## Questions to Consider

1. When should you use `CascadeType.PERSIST`?
2. When should you use `CascadeType.REMOVE`? When should you avoid it?
3. What other cascade types exist, and when would you use them?
4. Should you add cascading to the `Movie` → `Actor` relationship? Why or why not?

## Common Pitfall

**Circular References**: If you try to return a `Movie` with `MovieDetails` that references back to `Movie`, you'll get infinite recursion in JSON serialization. This is why we need DTOs!

Continue to **Exercise 05** to solve this problem (without using `@JsonManagedReference` and `@JsonBackReference`) with DTOs!
