# Exercise 02: Many-to-Many Relationship

## Learning Objective

Learn how to implement a `@ManyToMany` relationship with a custom join table between `Movie` and `Actor` entities.

## Background

A movie can have multiple actors, and an actor can appear in multiple movies. This is a classic many-to-many relationship.

In the database, this relationship is managed through a **join table** that stores pairs of movie IDs and actor IDs.

## Your Task

Implement the many-to-many relationship between `Movie` and `Actor`, then create an endpoint to add actors to movies.

### Step 1: Add Relationship to Movie Entity

Modify `Movie.java` to add a list of actors:

```java
@Entity
public class Movie {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    private Integer releaseYear;
    private String genre;
    
    @Embedded
    private Rating rating;

    // TODO: Add @ManyToMany relationship with custom join table
    // Use @JoinTable with:
    //   - name = "movie_actor"
    //   - joinColumns = @JoinColumn(name = "movie_id")
    //   - inverseJoinColumns = @JoinColumn(name = "actor_id")
    // Initialize the list as new ArrayList<>()
    
    // TODO: Add getter and setter for actors list
}
```

### Step 2: Add Method to MovieService

Add a method to add an actor to a movie in `MovieService.java`:

```java
public Movie addActorToMovie(Long movieId, Long actorId) {
    // TODO: Find the movie by ID (throw exception if not found)
    // TODO: Find the actor by ID (throw exception if not found)
    // TODO: Add actor to movie's actors list
    // TODO: Save and return the updated movie
}
```

**Hint:** You'll need to inject `ActorRepository` into the service.

### Step 3: Add Controller Endpoint

Add an endpoint to `MovieController.java`:

```java
@PostMapping("/{movieId}/actors/{actorId}")
public ResponseEntity<Movie> addActorToMovie(
        @PathVariable Long movieId, 
        @PathVariable Long actorId) {
    // TODO: Call the service method
    // TODO: Return the updated movie
}
```

## Testing Your Implementation

1. Restart your application

2. Create a movie (using .http in IntelliJ):

    ```http
    ### CREATE A MOVIE
    POST http://localhost:8080/api/movies
    Content-Type: application/json

    {
      "title": "Inception",
      "releaseYear": 2010,
      "genre": "Sci-Fi",
      "rating": {"score": 8.8, "voteCount": 2000000}
    }
    ```

3. Create two actors:

    ```http
    ### CREATE ACTOR
    POST http://localhost:8080/api/actors
    Content-Type: application/json

    {"name": "Leonardo DiCaprio", "birthYear": 1974}

    ### ADD ACTOR NO 2
    POST http://localhost:8080/api/actors
    Content-Type: application/json

    {"name": "Joseph Gordon-Levitt", "birthYear": 1981}
    ```

4. Add actors to the movie:

    ```http
    ### ASSIGN ACTOR NO 1 TO MOVIE
    POST http://localhost:8080/api/movies/1/actors/1

    ### ASSIGN ACTOR NO 2 TO MOVIE
    POST http://localhost:8080/api/movies/1/actors/2
    ```

5. Get the movie and verify it includes the actors:

    ```http
    ### VERIFY MOVIE WITH ACTORS
    GET http://localhost:8080/api/movies/1
    ```

6. Check the H2 console and verify the `MOVIE_ACTOR` join table exists with the correct foreign keys.

## Expected Result

The movie JSON should include an `actors` array with both actors. The database should have a `MOVIE_ACTOR` table with rows connecting movies to actors.

## Questions to Consider

1. Which entity is responsible for managing the relationship (Movie or Actor)?
2. What would happen if you added the same actor twice to a movie?
3. How would you implement the reverse relationship (getting all movies for an actor)?

Continue to **Exercise 03** to learn about one-to-one relationships!
