# Exercise: Adding GitHub Login

In this exercise, we will enable users to log in using their GitHub accounts. The GitHub users will not be stored in a database, but we will use the information provided by GitHub to create a user session.

## Prerequisites:
This exercise builds on the previous exercise, so make sure you have completed it before starting this one.

## Exercise 1: Create a GitHub OAuth App
1. Go to **GitHub settings** and navigate to "Developer settings" > "OAuth Apps" > "New OAuth App" [Link](https://github.com/settings/apps)
2. Fill in the application details:
   - **Application name**: Choose a name for your application
   - **Homepage URL**: `http://localhost`
   - **Callback URL**: `http://localhost/login/oauth2/code/github`
3. After registering the application, you will receive a **`Client ID`** and **`Client Secret`**, which you can use in your Spring Boot application's configuration.
4. Take a note of the `Client ID` and `Client Secret`, as you will need them in the next step.

The callback URL is the endpoint in your application that GitHub will redirect to after the user has authenticated. Spring Security will create a session for the user and store the authentication information in it, and return a session cookie to the client.


## Exercise 2: Configure Spring Security for GitHub Login

### Step 1: Add OAuth2 Client Dependency
Inside the `bff` app, add the following dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-oauth2-client</artifactId>
</dependency>
```

### Step 2: Configure OAuth2 Client
In your `application.properties` file, add the following configuration to enable GitHub OAuth2 login:
```properties
spring.security.oauth2.client.registration.github.client-id=YOUR_CLIENT_ID
spring.security.oauth2.client.registration.github.client-secret=YOUR_CLIENT_SECRET
spring.security.oauth2.client.registration.github.scope=read:user,user:email
```

Replace `YOUR_CLIENT_ID` and `YOUR_CLIENT_SECRET` with the values you obtained from GitHub in Exercise 1.

### Step 3: Enabling OAuth2 Login
In your `SecurityConfig` class, enable OAuth2 login by adding the following configuration:
```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http
) {
    http
            // ... other configurations
            .oauth2Login(Customizer.withDefaults())
            // ... other configurations
    return http.build();
}
```
This configuration enables OAuth2 login with the default settings. Spring Security will handle the OAuth2 login flow, including redirecting to GitHub for authentication and processing the callback.

### Step 4: Adding a login button in the frontend
In your frontend application, you can add a login button that redirects the user to the GitHub login page. For example, you can add the following HTML code to your login page:
```html
<a href="/oauth2/authorization/github">Login with GitHub</a>
```
When the user clicks this link, they will be redirected to GitHub's login page. After they authenticate, they will be redirected back to your application, and Spring Security will create a session for them.

### Step 5: Testing the GitHub Login
1. Start your Spring Boot application.
2. Open your frontend application in a web browser and click the "Login with GitHub" button.
3. You will be redirected to GitHub's login page. Log in with your GitHub credentials.
4. After successful authentication, you will be redirected back to your application, and you should see that you are logged in with your GitHub account.

## Exercise 3: Accessing User Information
When using GitHub login, Spring Security will create an `OAuth2User` object that contains the user's information provided by GitHub. You can access this information in your controllers or services. But since we support GitHub login, and login with our own users, we need to handle both cases. 

### Step 1: Create a TokenSubjectExtractor
Inside the `bff` app, create a new class called `TokenSubjectExtractor` inside the `security` package. This class will be responsible for extracting the subject (username) from the authentication token, whether the user is logged in using username/password or GitHub.

```java
@Component
public class TokenSubjectExtractor {

    public String extract(Authentication authentication) {
        if (authentication == null || !authentication.isAuthenticated()) {
            return null; // or throw an exception if you prefer
        }

        if (authentication.getPrincipal() instanceof OAuth2User oauth2User) {
            // for GitHub, the username is in the "login" attribute
            return oauth2User.getAttribute("login");
        }

        if (authentication.getPrincipal() instanceof UserDetails userDetails) {
            // User is logged in with username/password
            return userDetails.getUsername();
        }

        return authentication.getName(); // fallback to the default name
    }
}
```

If you add other Identity Providers in the future, you can extend this class to handle those as well.

### Step 2: Use the TokenSubjectExtractor
Inside the `JwtTokenGenerator` class, use constructor injection to inject the `TokenSubjectExtractor` and use it to extract the subject when generating a JWT token.

You need to replace this line:
```java
String subject = authentication.getName();
```

with this:
```java
String subject = tokenSubjectExtractor.extract(authentication);
```

This way, the `JwtTokenGenerator` will work for both username/password logins and GitHub logins, extracting the correct subject from the authentication token.

### Step 3: Updating the User Controller
Inside the `UserController` class use constructor injection to inject the `TokenSubjectExtractor` and use it to extract the subject when generating a JWT token.

You need to replace this line:
```java
return new UserResponse(username, roles);
```

with this:
```java
String subject = tokenSubjectExtractor.extract(authentication);

return new UserResponse(subject, roles);
```

This way, the `UserController` will return the correct username for both username/password logins and GitHub logins.

## Further notes
Even though we have enabled GitHub login, we are still generating JWT tokens for the users. This means that after a user logs in with GitHub, they will receive a JWT token that they can use to authenticate with the resource server. The resource server will validate the JWT token and allow access to protected resources based on the user's roles and permissions.

This is because our resource server only understands JWT tokens issued by our application, and not the authentication tokens issued by GitHub. When the user tries to access a protected resource from the frontend, that request will be sent to the bff, which will generate a short lived JWT token for the user and forward the request to the resource server with the JWT token in the Authorization header. The resource server will then validate the JWT token and allow access to the protected resource if the token is valid and the user has the necessary permissions.