# Exercise 05: DTOs with Java Records

## Learning Objective

Learn how to use Java Records as DTOs (Data Transfer Objects) to prevent circular references and control what data is exposed through your API.

## Background

By now, you've likely encountered a **circular reference** problem: 
- `Movie` references `MovieDetails`
- `MovieDetails` references back to `Movie`
- `Movie` references `Actor` list
- Each `Actor` might reference its `Movie` list

This causes **infinite recursion** when serializing to JSON!

**Solution**: Use DTOs to create a controlled, flat representation of your data without circular references.

## Your Task

Create DTO classes using Java Records and update your controller to use them instead of returning entities directly.

### Step 1: Create DTO Records

Create a new package `dto` and add these record classes:

**ActorResponse.java:**
```java
package com.example.moviedb.dto;

public record ActorResponse(
    Long id,
    String name,
    Integer birthYear
) {}
```

**MovieDetailsResponse.java:**
```java
package com.example.moviedb.dto;

public record MovieDetailsResponse(
    String plot,
    Integer budget,
    Integer runtime,
    String productionCompany
) {}
```

**MovieResponse.java:**
```java
package com.example.moviedb.dto;

import java.util.List;

public record MovieResponse(
    Long id,
    String title,
    Integer releaseYear,
    String genre,
    Double ratingScore,
    Integer ratingVoteCount,
    MovieDetailsResponse details,
    List<ActorResponse> actors
) {}
```

**CreateMovieRequest.java:**
```java
package com.example.moviedb.dto;

public record CreateMovieRequest(
    String title,
    Integer releaseYear,
    String genre,
    Double ratingScore,
    Integer ratingVoteCount
) {}
```

### Step 2: Add Mapping Methods to MovieService

Add methods to convert between entities and DTOs:

```java
private MovieResponse toMovieResponse(Movie movie) {
    // TODO: Map Movie entity to MovieResponse DTO
}

private Movie toMovieEntity(CreateMovieRequest request) {
    // TODO: Create new Movie from CreateMovieRequest
    // Create Rating object from ratingScore and ratingVoteCount
}
```

### Step 3: Update Service Methods to Use DTOs

Modify existing service methods:

```java
public MovieResponse createMovie(CreateMovieRequest request) {
    Movie movie = toMovieEntity(request);
    Movie saved = movieRepository.save(movie);
    return toMovieResponse(saved);
}

public MovieResponse getMovieById(Long id) {
    Movie movie = movieRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Movie not found"));
    return toMovieResponse(movie);
}

public List<MovieResponse> getAllMovies() {
    var movies =movieRepository.findAll();
    List<MovieResponse> responses = new ArrayList<>();
    for (Movie m : movies) {
        responses.add(toMovieResponse(m));
    }
    return responses;
}
```

### Step 4: Update Controller to Use DTOs

Modify `MovieController.java`:

```java
@PostMapping
public ResponseEntity<MovieResponse> createMovie(@RequestBody CreateMovieRequest request) {
    return ResponseEntity.ok(movieService.createMovie(request));
}

@GetMapping("/{id}")
public ResponseEntity<MovieResponse> getMovieById(@PathVariable Long id) {
    return ResponseEntity.ok(movieService.getMovieById(id));
}

@GetMapping
public ResponseEntity<List<MovieResponse>> getAllMovies() {
    return ResponseEntity.ok(movieService.getAllMovies());
}
```

## Testing Your Implementation

1. Restart your application

2. Create a complete movie with rating:

    ```http
    ### ADD MOVIE
    POST http://localhost:8080/api/movies
    Content-Type: application/json

    {
        "title": "The Matrix",
        "releaseYear": 1999,
        "genre": "Sci-Fi",
        "ratingScore": 8.7,
        "ratingVoteCount": 1800000
    }
    ```

3. Add actors and details (using previous endpoints)

4. Get the movie - should now return clean JSON without circular references:

    ```http
    ### VERIFY MOVIE WITH DETAILS
    GET http://localhost:8080/api/movies/1
    ```

## Expected Result

The API should return clean JSON with:
- Movie information
- Rating flattened (not nested)
- Details included (if present)
- Actors list included (if present)
- **NO circular references!**

## Questions to Consider

1. What are the benefits of using DTOs?
2. Why use Java Records instead of regular classes for DTOs?
3. How do DTOs help with API versioning?
4. When should you use different DTOs for request vs response?

Continue to **Exercise 06** to learn about testing JPA repositories!
