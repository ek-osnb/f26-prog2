# Bidirectional One-to-Many Relationships

**Prerequisites:** You should have completed Exercise 02 (Many-to-One Relationships) with Customer and Order entities.

**Goal:** Learn how to create bidirectional relationships, understand the owning vs inverse side, and handle JSON serialization issues.

## What You'll Build

Add a `List<Order>` to the `Customer` entity so you can navigate from Customer to Orders (and vice versa).

## Understanding Bidirectional Relationships

**Current situation (Unidirectional):**
- `Order` → `Customer`: You can navigate from an order to its customer 
- `Customer` → `Orders`: You CANNOT navigate from a customer to their orders 

**After this exercise (Bidirectional):**
- `Order` → `Customer`: 
- `Customer` → `Orders`: 

**Important concepts:**
- **Owning side:** The side with the foreign key (`Order` has `customer_id`)
- **Inverse side:** The side with `mappedBy` (`Customer` with `List<Order>`)
- Only the owning side matters for database updates

## Step 1: Add Orders List to Customer

Update your `Customer` class in the `model` package:

```java
package com.example.demo.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Customer {
    
    // ... existing fields ...
    
    @OneToMany(mappedBy = "customer")
    private List<Order> orders = new ArrayList<>();
    
    // ... existing constructors, getters, setters ...

    public List<Order> getOrders() {
        return orders;
    }
    
    public void setOrders(List<Order> orders) {
        this.orders = orders;
    }
}
```

**Key points:**
- `@OneToMany(mappedBy = "customer")`: Creates the One-to-Many relationship
- `mappedBy = "customer"`: Refers to the `customer` field in the `Order` class
- This tells JPA: "Don't create a new foreign key; use the one that already exists in the Order table"
- Initialize the list to avoid `NullPointerException`

## Step 2: Test the Bidirectional Relationship

1. **Restart your application**

2. **Check the database in H2 Console:**
   - No new tables or columns should be created
   - The `ORDERS` table still only has a `CUSTOMER_ID` column

3. **Test the customer endpoint:** `GET http://localhost:8080/api/customers/1`

**Expected behavior:** You should see an error or empty response! Let's understand why.

## Step 3: Understanding the Problem - Infinite Recursion

When you try to access `/api/customers/1`, Jackson (the JSON serializer) tries to:

1. Serialize `Customer` → includes `orders` list
2. Serialize each `Order` → includes `customer`
3. Serialize `Customer` again → includes `orders` list
4. Serialize each `Order` again → includes `customer`
5. **Infinite loop!** 

## Step 4: Fix with Jackson Annotations

Update both `Customer` and `Order` classes to handle JSON serialization:

**Customer class:**
```java
package com.example.demo.model;

import com.fasterxml.jackson.annotation.JsonManagedReference;
import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Customer {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    private String phone;
    
    @OneToMany(mappedBy = "customer")
    @JsonManagedReference
    private List<Order> orders = new ArrayList<>();
    
    // ... rest of the class (constructors, getters, setters)
}
```

**Order class:**
```java
package com.example.demo.model;

import com.fasterxml.jackson.annotation.JsonBackReference;
import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "orders")
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String orderNumber;
    private LocalDate orderDate;
    private Double totalAmount;
    
    @ManyToOne
    @JsonBackReference
    private Customer customer;
    
    // ... rest of the class (constructors, getters, setters)
}
```

**What do these annotations do?**
- `@JsonManagedReference`: On the "parent" side (Customer) - includes this in JSON
- `@JsonBackReference`: On the "child" side (Order) - **excludes** this from JSON to break the cycle

## Step 5: Test Again

1. **Restart your application**

2. **Test:** `GET http://localhost:8080/api/customers/1`

**Expected JSON:**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "12345678",
  "orders": [
    {
      "id": 1,
      "orderNumber": "ORD-001",
      "orderDate": "2025-01-15",
      "totalAmount": 299.99
    },
    {
      "id": 2,
      "orderNumber": "ORD-002",
      "orderDate": "2025-01-16",
      "totalAmount": 149.50
    },
    {
      "id": 5,
      "orderNumber": "ORD-005",
      "orderDate": "2025-01-19",
      "totalAmount": 1299.99
    }
  ]
}
```

3. **Test:** `GET http://localhost:8080/api/orders/1`

**Notice:** The order **no longer includes** the customer object (because of `@JsonBackReference`)

```json
{
  "id": 1,
  "orderNumber": "ORD-001",
  "orderDate": "2025-01-15",
  "totalAmount": 299.99
}
```

## Step 6: Add Helper Methods (Best Practice)

To maintain both sides of the relationship, add helper methods to `Customer`:

```java
package com.example.demo.model;

import com.fasterxml.jackson.annotation.JsonManagedReference;
import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Customer {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    private String phone;
    
    @OneToMany(mappedBy = "customer")
    @JsonManagedReference
    private List<Order> orders = new ArrayList<>();
    
    // Constructors...
    
    // Helper method to add an order
    public void addOrder(Order order) {
        orders.add(order);
        order.setCustomer(this);
    }
    
    // Helper method to remove an order
    public void removeOrder(Order order) {
        orders.remove(order);
        order.setCustomer(null);
    }
    
    // Getters and setters...
}
```

**Why use helper methods?**
- Ensures both sides of the relationship are always in sync
- Prevents bugs where you update one side but forget the other

**Update your `InitData` to use the helper method:**

```java
@Override
public void run(String... args) throws Exception {
    // Create and save customers
    
    // Create orders and add them to customers using helper method
    
    // Use helper method to maintain both sides
    
    System.out.println("Initial data created:");
    System.out.println("- Customers: " + customerRepository.count());
    System.out.println("- Orders: " + orderRepository.count());
}
```

## Step 7: Understanding Owning vs Inverse Side

**Owning side (`Order`):**
- Has the foreign key column (`customer_id`)
- Changes here affect the database
- Setting `order.setCustomer(customer)` updates `customer_id` in the database

**Inverse side (`Customer`):**
- Uses `mappedBy`
- For convenience only (navigation in Java)
- Adding orders to `customer.getOrders()` does **NOT** update the database!
- You **must** set the owning side: `order.setCustomer(customer)`

**Test this:**
```java
// This WORKS (owning side)
order.setCustomer(customer);
orderRepository.save(order);  // customer_id is saved 

// This DOES NOT WORK (inverse side only)
customer.getOrders().add(order);
customerRepository.save(customer);  // customer_id is NOT saved 

// This WORKS (both sides maintained)
customer.addOrder(order);  // Uses helper method that updates both sides
orderRepository.save(order);  // customer_id is saved 
```

<!-- ## What You Learned

 How to create bidirectional relationships with `@OneToMany(mappedBy = "...")`  
 Understanding the owning side vs inverse side  
 How to fix infinite recursion with `@JsonManagedReference` and `@JsonBackReference`  
 Why helper methods are important for maintaining relationship consistency  
 That bidirectional relationships don't change the database structure  
 How to navigate from both sides of a relationship in Java -->

<!-- ## Trade-offs: Unidirectional vs Bidirectional

**Unidirectional (Order → Customer only):**
-  Simpler
-  No JSON serialization issues
-  Can't easily get all orders for a customer from the Customer object

**Bidirectional (both directions):**
-  Can navigate both ways
-  Convenient for business logic
-  More complex
-  Requires managing both sides
-  JSON serialization issues (need `@JsonManagedReference`/`@JsonBackReference`)

**Alternative to bidirectional:** Use repository query methods instead:
```java
orderRepository.findByCustomerId(customerId)  // Works without bidirectional relationship
``` -->
