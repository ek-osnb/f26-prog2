# Spring Data JPA Basics

**Goal:** Learn how to set up Spring Data JPA, create entities, repositories, and persist data to a database.

## What You'll Build

A simple REST API for managing customers with database persistence using Spring Data JPA and H2 database.

## Step 1: Create a New Spring Boot Project

In IntelliJ, create a new **Spring Boot 4** project with the following settings:
- **JDK:** 25
- **Java:** 25
- **Dependencies:** `Spring Web`, `Spring Data JPA`, `H2 Database`

Create the project and wait for IntelliJ to finish setting up.

## Step 2: Configure H2 Database

Create or edit the `src/main/resources/application.properties` file and add the following configuration:

```properties
# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:customerdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA/Hibernate Configuration
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# H2 Console Configuration
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

**Understanding the configuration:**
- `spring.datasource.url`: In-memory H2 database named "customerdb"
- `spring.jpa.hibernate.ddl-auto=create-drop`: Creates tables on startup, drops them on shutdown
- `spring.jpa.show-sql=true`: Prints SQL statements to console (useful for learning)
- `spring.h2.console.enabled=true`: Enables the H2 web console

## Step 3: Create the Customer Entity

Create a new package `model` and inside it, create a `Customer` class:

```java
package com.example.demo.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Customer {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    private String phone;
    
    // Default constructor (required by JPA)
    public Customer() {
    }
    
    // Constructor with fields
    public Customer(String name, String email, String phone) {
        this.name = name;
        this.email = email;
        this.phone = phone;
    }
    
    // Getters and setters
}
```

**Key annotations:**
- `@Entity`: Marks this class as a JPA entity (will be mapped to a database table)
- `@Id`: Marks the primary key field
- `@GeneratedValue(strategy = GenerationType.IDENTITY)`: Auto-generates ID values

## Step 4: Create the Customer Repository

Create a new package `repository` and inside it, create a `CustomerRepository` interface:

```java
package com.example.demo.repository;

import com.example.demo.model.Customer;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CustomerRepository extends JpaRepository<Customer, Long> {
    // JpaRepository provides CRUD methods out of the box:
    // - save()
    // - findById()
    // - findAll()
    // - deleteById()
    // - count()
    // and many more!
}
```

**Note:** You don't need to implement this interface! Spring Data JPA automatically creates the implementation at runtime.

## Step 5: Create Initial Data

Create a new package `config` and inside it, create an `InitData` class:

```java
package com.example.demo.config;

import com.example.demo.model.Customer;
import com.example.demo.repository.CustomerRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class InitData implements CommandLineRunner {
    
    private final CustomerRepository customerRepository;
    
    public InitData(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }
    
    @Override
    public void run(String... args) throws Exception {
        // Create and save customers
        Customer customer1 = new Customer("John Doe", "john@example.com", "12345678");
        Customer customer2 = new Customer("Jane Smith", "jane@example.com", "23456789");
        Customer customer3 = new Customer("Bob Johnson", "bob@example.com", "34567890");
        
        customerRepository.save(customer1);
        customerRepository.save(customer2);
        customerRepository.save(customer3);
        
        System.out.println("Initial data created: " + customerRepository.count() + " customers");
    }
}
```

**What is CommandLineRunner?**
- Runs automatically when the application starts
- Perfect for creating initial test data

## Step 6: Create the Customer Controller

Create a new package `controller` and inside it, create a `CustomerController` class:

```java
package com.example.demo.controller;

import com.example.demo.model.Customer;
import com.example.demo.repository.CustomerRepository;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/customers")
public class CustomerController {
    
    private final CustomerRepository customerRepository;
    
    public CustomerController(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }
    
    @GetMapping
    public List<Customer> getAllCustomers() {
        return customerRepository.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Customer> getCustomerById(@PathVariable Long id) {
        return customerRepository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public Customer createCustomer(@RequestBody Customer customer) {
        return customerRepository.save(customer);
    }
}
```

## Step 7: Test Your Application

1. **Run your application** (click the green play button in IntelliJ or run the main class)

2. **Check the console output:**
   - You should see SQL statements creating the `customer` table
   - You should see the message "Initial data created: 3 customers"

3. **Test the REST endpoints:**
   - Open your browser and go to: `http://localhost:8080/api/customers`
   - You should see a JSON array with 3 customers

4. **Access the H2 Console:**
   - Go to: `http://localhost:8080/h2-console`
   - **JDBC URL:** `jdbc:h2:mem:customerdb` (must match your application.properties)
   - **Username:** `sa`
   - **Password:** (leave empty)
   - Click "Connect"
   - Run SQL: `SELECT * FROM CUSTOMER;`
   - You should see your 3 customers in the table

## Step 8: Explore Data Persistence

**Question:** What happens to the data when you restart your application? Why?

**Hint:** Check the `spring.jpa.hibernate.ddl-auto` setting in your `application.properties`.

## Exercise: Add More Endpoints

Add the following endpoints to your `CustomerController`:

1. **PUT** `/api/customers/{id}` - Update a customer
   - Return 404 if customer not found
   - Return the updated customer on success

2. **DELETE** `/api/customers/{id}` - Delete a customer
   - Return 204 No Content on success

**Hints:**
- Use `customerRepository.existsById(id)` to check if customer exists
- Use `customerRepository.deleteById(id)` to delete
- Use `ResponseEntity.noContent().build()` for 204 responses

## What You Learned

- How to add Spring Data JPA dependencies  
- How to configure an H2 database  
- How to create JPA entities with `@Entity`, `@Id`, and `@GeneratedValue`  
- How to create repositories by extending `JpaRepository`  
- How to use `CommandLineRunner` for initial data  
- How to use the H2 Console to inspect your database  
- How to perform CRUD operations with JPA repositories
