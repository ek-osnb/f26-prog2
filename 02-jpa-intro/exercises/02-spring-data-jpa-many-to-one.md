# Many-to-One Relationships with JPA

**Prerequisites:** You should have completed Exercise 01 (Spring Data JPA Basics) with a working Customer entity and repository.

**Goal:** Learn how to create Many-to-One relationships using JPA annotations and understand how foreign keys work.

## What You'll Build

Extend your application with an `Order` entity where **many orders belong to one customer**.

## Understanding Many-to-One

- **Many-to-One:** Multiple orders can belong to the same customer
- **Database structure:** The `Order` table will have a `customer_id` foreign key column
- **Owning side:** The `Order` entity owns the relationship (it has the foreign key)

## Step 1: Create the Order Entity

Create a new class `Order` in the `model` package:

```java
package com.example.demo.model;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "orders") // "order" is a reserved keyword in SQL, so we use "orders"
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String orderNumber;
    private LocalDate orderDate;
    private Double totalAmount;
    
    @ManyToOne
    private Customer customer;
    
    // Default constructor (required by JPA)
    public Order() {
    }
    
    // Constructor with fields
    public Order(String orderNumber, LocalDate orderDate, Double totalAmount, Customer customer) {
        this.orderNumber = orderNumber;
        this.orderDate = orderDate;
        this.totalAmount = totalAmount;
        this.customer = customer;
    }
    
    // Getters and setters
}
```

**Key annotations:**
- `@ManyToOne`: Defines the Many-to-One relationship
- `@Table(name = "orders")`: Specifies custom table name (since "order" is a SQL reserved keyword)

**Optional: Customize the foreign key column name:**

If you want to specify the foreign key column name explicitly, you can use `@JoinColumn`:

```java
@ManyToOne
@JoinColumn(name = "customer_id")  // Optional: defaults to "customer_id" anyway
private Customer customer;
```

## Step 2: Create the Order Repository

Create a new interface `OrderRepository` in the `repository` package:

```java
package com.example.demo.repository;

import com.example.demo.model.Order;
import org.springframework.data.jpa.repository.JpaRepository;

public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

## Step 3: Update Initial Data

Update your `InitData` class to create orders for the customers:

```java
package com.example.demo.config;

import com.example.demo.model.Customer;
import com.example.demo.model.Order;
import com.example.demo.repository.CustomerRepository;
import com.example.demo.repository.OrderRepository;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.time.LocalDate;

@Component
public class InitData implements CommandLineRunner {
    
    private final CustomerRepository customerRepository;
    private final OrderRepository orderRepository;
    
    public InitData(CustomerRepository customerRepository, OrderRepository orderRepository) {
        this.customerRepository = customerRepository;
        this.orderRepository = orderRepository;
    }
    
    @Override
    public void run(String... args) throws Exception {
        // Create and save customers FIRST
        // (same as before)
        // Add 5 Orders
    }
}
```

**Important:** Notice that we save customers **before** creating orders. This is because:
- The customer must exist in the database before we can reference it in an order
- JPA needs the customer's ID to set the foreign key

## Step 4: Create the Order Controller

Create a new class `OrderController` in the `controller` package:

```java
package com.example.demo.controller;

import com.example.demo.model.Order;
import com.example.demo.repository.OrderRepository;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderRepository orderRepository;
    
    public OrderController(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
    
    @GetMapping
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        return orderRepository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
}
```

## Step 5: Test Your Application

1. **Restart your application**

2. **Test the endpoints:**
   - `GET http://localhost:8080/api/orders`
   - You should see 5 orders with nested customer data
   - Example JSON:
   ```json
   {
     "id": 1,
     "orderNumber": "ORD-001",
     "orderDate": "2025-01-15",
     "totalAmount": 299.99,
     "customer": {
       "id": 1,
       "name": "John Doe",
       "email": "john@example.com",
       "phone": "12345678"
     }
   }
   ```

4. **Access the H2 Console** (`http://localhost:8080/h2-console`)


##  Exercise: Custom Query Methods

Spring Data JPA can create query methods automatically based on method names!

Add these methods to your `OrderRepository`:

```java
package com.example.demo.repository;

import com.example.demo.model.Order;
import com.example.demo.model.Customer;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface OrderRepository extends JpaRepository<Order, Long> {
    
    // Find all orders for a specific customer
    List<Order> findByCustomer(Customer customer);
    
    // Find all orders for a customer by customer ID
    List<Order> findByCustomerId(Long customerId);
    
    // Find orders by order number
    Order findByOrderNumber(String orderNumber);
}
```

**Test these methods** by adding a new endpoint in `OrderController`:

```java
@GetMapping("/customer/{customerId}")
public List<Order> getOrdersByCustomerId(@PathVariable Long customerId) {
    return orderRepository.findByCustomerId(customerId);
}
```

**Test:** `GET http://localhost:8080/api/orders/customer/1`

## Bonus: Add POST Endpoint for Orders

Add this endpoint to `OrderController` to create new orders:

```java
@PostMapping
public ResponseEntity<Order> createOrder(@RequestBody Order order) {
    // In a real application, you should validate that the customer exists
    Order savedOrder = orderRepository.save(order);
    return ResponseEntity.ok(savedOrder);
}
```

**Test with Postman:**

`POST http://localhost:8080/api/orders`

Body (JSON):
```json
{
  "orderNumber": "ORD-006",
  "orderDate": "2025-01-20",
  "totalAmount": 450.00,
  "customer": {
    "id": 2
  }
}
```

**Note:** You only need to provide the customer's ID, not all customer fields!

## What You Learned

- How to create Many-to-One relationships with `@ManyToOne`  
- How to use `@Table` to specify custom table names  
- How foreign keys work in JPA (Order table has `customer_id` column)  
- The importance of save order (parent before child)  
- How to create custom query methods in repositories  
- How to test relationships in the H2 Console  
- How nested objects are automatically serialized to JSON
