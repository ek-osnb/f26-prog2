# Exercise: User Persistence and Spring Security

In this exercise, we will implement user persistence in our Spring Security application. We will create a custom `UserDetailsService` that retrieves user information from a database, and we will configure Spring Security to use this service for authentication.

## Background
By default, Spring Security provides an in-memory user store with a single user. However, in real applications, you will typically want to store user information in a database. To do this, you need to implement the `UserDetailsService` interface, which is responsible for loading user-specific data. This is the integration point between Spring Security and your user database.

## Prerequisites
You should have completed the previous exercise on Thymeleaf and Spring Security, as we will build upon that setup. Although this will also work with other authentication configurations, and doesn't need to be a thymeleaf application. 

For this exercise you need at least the following dependendencies in your project:
- `Spring Web`
- `Spring Security`
- `Spring Data JPA`
- `MySQL Driver`

## Exercise 1 - JPA Entities and Repositories

### Goal
Create a JPA entity for users and a corresponding repository to manage user data in the database. The integration with Spring Security will be done in the following exercises.

### Instructions
1. Create a JPA entity class named `Role` with the following fields:
   - `id` (Long, primary key)
   - `name` (String) - not null, unique

   > **Hint:** Use `@Column(nullable = false, unique = true)` to enforce the constraints on the `name` field.
    <!-- ```java
    @Entity
    public class Role {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        @Column(nullable = false, unique = true)
        private String name;
        // getters and setters and constructors
    }
    ``` -->
2. Create a JPA entity class named `AppUser` with the following fields:
   - `id` (Long, primary key)
   - `username` (String) - not null, unique
   - `password` (String) - not null
   - `roles` (Set<Role>) - many-to-many relationship with `Role`
    > **Hint:** Use `@ManyToMany` to define the relationship between `AppUser` and `Role`. You may also want to use `@JoinTable` to specify the join table for the many-to-many relationship.
     <!-- ```java
     @Entity
     public class User {
          @Id
          @GeneratedValue(strategy = GenerationType.IDENTITY)
          private Long id;
          @Column(nullable = false, unique = true)
          private String username;
          @Column(nullable = false)
          private String password;
          @ManyToMany(fetch = FetchType.EAGER)
          @JoinTable(
                name = "users_roles",
                joinColumns = @JoinColumn(name = "user_id"),
                inverseJoinColumns = @JoinColumn(name = "role_id")
          )
          private Set<Role> roles;
          // getters and setters and constructors
     }
     ``` -->
3. Create a Spring Data JPA repository interface named `AppUserRepository` for the `AppUser` entity. This repository should extend `JpaRepository<AppUser, Long>` and include a method to find a user by their username.

    **Hint:**
    ```java
    public interface AppUserRepository extends JpaRepository<AppUser, Long> {
        Optional<AppUser> findByUsername(String username);
    }
    ```
4. Create a Spring Data JPA repository interface named `RoleRepository` for the `Role` entity. This repository should extend `JpaRepository<Role, Long>` and include a method to find a role by its name.

    **Hint:**
    ```java
    public interface RoleRepository extends JpaRepository<Role, Long> {
        Optional<Role> findByName(String name);
    }
    ```

## Exercise 2 - Implementing UserDetailsService

### Goal
Implement a custom `UserDetailsService` that retrieves user information from the database using the `AppUserRepository`. This service will be used by Spring Security for authentication.

### Instructions
1. Create a class named `JpaUserDetailsService` that implements the `UserDetailsService` interface, which requires you to implement the `loadUserByUsername(String username)` method.
2. Inject the `AppUserRepository` into the `JpaUserDetailsService` class to access user data from the database.

    **Hint:**
    ```java
    public class JpaUserDetailsService implements UserDetailsService {
        private final AppUserRepository userRepository;

        public JpaUserDetailsService(AppUserRepository userRepository) {
            this.userRepository = userRepository;
        }

        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            AppUser user = userRepository.findByUsername(username)
                    .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
            return User.builder()
                    .username(user.getUsername())
                    .password(user.getPassword())
                    .authorities(user.getRoles().stream()
                            .map(role -> new SimpleGrantedAuthority(role.getName()))
                            .collect(Collectors.toList()))
                    .build();
        }
    }
    ```
3. Configure Spring Security to use the `JpaUserDetailsService` for authentication. If you already have a bean for `UserDetailsService` in your security configuration, replace it with the new implementation. If not, add a new bean definition for `UserDetailsService` that returns an instance of `JpaUserDetailsService`. Make sure to also configure a `PasswordEncoder` bean, such as `BCryptPasswordEncoder`, to handle password encoding.

    **Hint:**
    ```java
    @Configuration
    public class SecurityConfig {
        
        // Previous security configuration...

        @Bean
        public UserDetailsService userDetailsService(AppUserRepository userRepository) {
            return new JpaUserDetailsService(userRepository);
        }

        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
    }
    ```
4. Create test users and and roles in the database to verify that authentication works correctly with the new `UserDetailsService`. Before storing the passwords in the database, make sure to encode them using the `PasswordEncoder` bean you configured.

    **Hint:**
    ```java
    @Component
    public class DataInitializer implements CommandLineRunner {
        private final AppUserRepository userRepository;
        private final RoleRepository roleRepository;
        private final PasswordEncoder passwordEncoder;

        public DataInitializer(AppUserRepository userRepository, RoleRepository roleRepository, PasswordEncoder passwordEncoder) {
            this.userRepository = userRepository;
            this.roleRepository = roleRepository;
            this.passwordEncoder = passwordEncoder;
        }

        @Override
        public void run(String... args) throws Exception {
            Role adminRole = new Role();
            adminRole.setName("ROLE_ADMIN");
            roleRepository.save(adminRole);

            Role userRole = new Role();
            userRole.setName("ROLE_USER");
            roleRepository.save(userRole);

            AppUser adminUser = new AppUser();
            adminUser.setUsername("admin");
            adminUser.setPassword(passwordEncoder.encode("pw"));
            adminUser.setRoles(Set.of(adminRole));
            userRepository.save(adminUser);

            AppUser regularUser = new AppUser();
            regularUser.setUsername("user");
            regularUser.setPassword(passwordEncoder.encode("pw"));
            regularUser.setRoles(Set.of(userRole));
            userRepository.save(regularUser);

        }
    }
    ```

5. Test the authentication process by logging in with the test users you created. Verify that the correct roles and authorities are assigned to each user and that access control works as expected based on their roles.

### Notes

#### Roles
In spring security, the roles must be prefixed with `ROLE_` to be recognized as authorities. For example, if you have a role named `ADMIN`, it should be stored in the database as `ROLE_ADMIN`. This is a convention used by Spring Security to differentiate between roles and other types of authorities. Other authorities that are not roles can be stored without the `ROLE_` prefix, like `READ_PRIVILEGE` or `WRITE_PRIVILEGE`.

#### Adapter Design Pattern
The `JpaUserDetailsService` is an example of the [adapter design pattern](https://refactoring.guru/design-patterns/adapter), which allows us to adapt our `User` JPA entity to the `UserDetails` interface required by Spring Security. This separation of concerns allows us to keep our domain model (the `User` entity) separate from the security model (the `UserDetails` interface), making our code more modular and easier to maintain.

#### Bean vs Service Annotation
In the security configuration, we defined a bean for `UserDetailsService` that returns an instance of `JpaUserDetailsService`. This is one way to tell Spring Security to use our custom `UserDetailsService` implementation.
You can instead of creating a bean in the config, annotate the `JpaUserDetailsService` class with `@Service`. The reason for using the bean approach, is to collect all security related beans in the same place, and to make it more explicit that this is the `UserDetailsService` that Spring Security should use. However, both approaches are valid and will work correctly.
