# Exercise: Package by Feature

In this exercise, you will refactor your fullstack project to use a **package-by-feature** structure. This means that instead of organizing your code by layer (e.g., controllers, services, repositories), you will organize it by feature (e.g., user, product, order).

The reason for this is that we can make the packages encapsulate related functionality and enforce clear boundaries between different parts of the application. The idea is to group related classes together, and only expose the necessary classes to other parts of the application. This can help to improve the maintainability and readability of the codebase.


## Instructions

### Exercise 1: Refactor to Package by Feature

1. Create a fork of your groups fullstack project. Clone the forked repository to your local machine and open it in your IDE. Create a new branch for this exercise (e.g., `package-by-feature`).

2. Identify the different features in your application. For example, you might have features like `movie`, `showtime`, `booking`, etc.
3. Create a package for each feature and move the relevant classes into their respective feature packages. Delete the old empty packages (e.g., `controller`, `service`, `repository` etc.).
4. The import statements should automatically update when you move the classes, but make sure to check for any import errors and fix them.
5. After refactoring, run your application and make sure everything is working as expected.

> Now the codebase structure has only changed in terms of how the files are organized, but the functionality should remain the same.


### Exercise 2: Access Modifiers an Package Encapsulation

The goal is to use Java's access modifiers to enforce clear boundaries between feature packages. Other packages should only interact with a feature through a **defined public interface** — typically the service class and relevant DTOs. Everything else (repository, entity, mapper) should be hidden.

Here is a general guideline you can follow for the access modifiers of different class roles:

| Class type                   | Visibility       |
|-----------------------------|------------------|
| Service interface               | `public` |
| Service implementation class     | **package-private** unless it doesn't implement a service interface  |
| DTOs (request/response)     | `public`           |
| REST Controller             | **package-private**  |
| Repository interface        | **package-private**  |
| Entity class                | `public` (required by JPA relationships) |
| Mapper / helper classes     | **package-private** |


1. Pick a feature that is not heavily depended on by other features.
2. Go through each class in the package and ask: *"Does any class outside this package need to reference this directly?"* If not, remove the public modifier (making it package-private).
3. Repeat for methods within classes — constructor and service methods called from outside stay public; internal helpers can be private.
4. After making these changes, run your application and ensure that everything still works as expected.

### Exercise 3: Use-case-driven service interfaces

Instead of having one huge service class per feature, you can break down the service layer into multiple interfaces that represent specific use cases or operations. 

For example, instead of having a single huge `MovieService` class with methods for all movie-related operations, you could have:

**Create use case:**
```java
public interface CreateMovieUseCase {
    MovieResponse handle(CreateMovieRequest request);
}
```

And then make the implementation class package-private:

```java
@Service
class CreateMovieService implements CreateMovieUseCase {
    public MovieResponse handle(CreateMovieRequest request) {
        // implementation
    }
}
```

**Query use case:**
```java
public interface QueryMovieUseCase {
    MovieResponse handle(QueryMovieRequest request);
}
```

And then make the implementation class package-private:

```java
@Service
class QueryMovieService implements QueryMovieUseCase {
    public MovieResponse handle(QueryMovieRequest request) {
        // implementation
    }
}
```

This way, you can have a clear separation of concerns and make it easier to understand the purpose of each service interface. It also allows for better testability and maintainability, as you can focus on specific use cases when writing tests or making changes to the codebase.

> So when you have a new feature or operation to implement, you can simply create a new use case interface and implementation without having to modify a large existing service class.

**Caveats:**
This approach can have some downsides:
- It can be a bit more work to set up initially, as you need to create multiple interfaces and implementation classes.
- Managing and maintaining a larger number of smaller classes can be more complex than a few larger ones.
- It may not be necessary for very simple features with only a few operations, where a single service class might suffice.

1. Pick one feature from Exercise 2 to refactor.
2. Identify each operation the service exposes — each one becomes its own use case (for example `CreateMovieUseCase`, `GetMoviesUseCase`, `DeleteMovieUseCase`).
3. Create a `public` interface for each use case with a single `handle` method (or a descriptively named method).
4. For each use case, create a package-private implementation class annotated with `@Service` that implements the interface.
5. Delete the original service class, moving the logic into the appropriate implementation classes.
6. Update any class that previously injected the old service to now inject the specific use-case interface it needs.
7. Run the application and verify everything works.

### Exercise 4 (optional): Repeat for all features

1. Repeat Exercise 2 and 3 for the other features in your application, applying the same principles of package encapsulation and use-case-driven service interfaces.
2. After refactoring all features, run your application and ensure that everything is working as expected.