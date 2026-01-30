# DTOs with Java Records

**Prerequisites:** You should have completed Exercise 03 (Bidirectional Relationships) with Customer and Order entities using `@JsonManagedReference` and `@JsonBackReference`.

**Goal:** Learn why and how to use DTOs (Data Transfer Objects) with Java Records to decouple your API from your database entities.

## Why Use DTOs?

### Problems with Exposing Entities Directly

1. **Security:** You might accidentally expose sensitive fields (passwords, internal IDs)
2. **Coupling:** Your API structure is tied to your database structure
3. **Over-fetching:** Clients get data they don't need
4. **Breaking changes:** Database changes force API changes
5. **Circular dependencies:** Need Jackson annotations (`@JsonBackReference`) to prevent infinite loops

### Benefits of DTOs

- **Control:** You decide exactly what data the API returns  
- **Stability:** Database changes don't break your API  
- **Security:** Only expose what's needed  
- **Flexibility:** Different endpoints can return different views of the same entity  
- **No circular dependencies:** DTOs are simple, flat objects

## What You'll Build

Replace your current API responses (entities with Jackson annotations) with clean DTOs using Java Records.

## Step 1: Understanding Java Records

Use Java Records to create immutable DTOs:

```java
// Java Record (concise)
public record CustomerDTO(Long id, String name, String email) {}
```

**Records automatically provide:**
- Constructor with all fields
- Getters (e.g., `id()`, `name()`, `email()`)
- `equals()`, `hashCode()`, `toString()`
- Immutability (all fields are final)

## Step 2: Create DTO Package and Classes

Create a new package `dto` and add these DTO records:

**CustomerDTO.java:**
```java
package com.example.demo.dto;

import java.util.List;

public record CustomerDTO(
    Long id,
    String name,
    String email,
    String phone,
    List<OrderSummaryDTO> orders
) {}
```

**OrderSummaryDTO.java:**
```java
package com.example.demo.dto;

import java.time.LocalDate;

public record OrderSummaryDTO(
    Long id,
    String orderNumber,
    LocalDate orderDate,
    Double totalAmount
) {}
```

**OrderDTO.java:**
```java
package com.example.demo.dto;

import java.time.LocalDate;

public record OrderDTO(
    Long id,
    String orderNumber,
    LocalDate orderDate,
    Double totalAmount,
    CustomerSummaryDTO customer
) {}
```

**CustomerSummaryDTO.java:**
```java
package com.example.demo.dto;

public record CustomerSummaryDTO(
    Long id,
    String name,
    String email
) {}
```

**Notice:**
- `CustomerDTO` includes a list of `OrderSummaryDTO` (not full orders)
- `OrderDTO` includes `CustomerSummaryDTO` (not full customer with all orders)
- No circular dependency! No need for `@JsonBackReference`
- We control exactly what data is included

## Step 3: Create Mapper Classes

Create a new package `mapper` and add mapper classes to convert between entities and DTOs:

**CustomerMapper.java:**
```java
package com.example.demo.mapper;

import com.example.demo.dto.CustomerDTO;
import com.example.demo.dto.CustomerSummaryDTO;
import com.example.demo.dto.OrderSummaryDTO;
import com.example.demo.model.Customer;
import com.example.demo.model.Order;

import java.util.ArrayList;
import java.util.List;

public class CustomerMapper {
    
    public static CustomerDTO toDTO(Customer customer) {
        List<OrderSummaryDTO> orderSummaries = new ArrayList<>();
        for (Order order : customer.getOrders()) {
            orderSummaries.add(toOrderSummaryDTO(order));
        }
        
        return new CustomerDTO(
            customer.getId(),
            customer.getName(),
            customer.getEmail(),
            customer.getPhone(),
            orderSummaries
        );
    }
    
    public static CustomerSummaryDTO toSummaryDTO(Customer customer) {
        return new CustomerSummaryDTO(
            customer.getId(),
            customer.getName(),
            customer.getEmail()
        );
    }
    
    private static OrderSummaryDTO toOrderSummaryDTO(Order order) {
        return new OrderSummaryDTO(
            order.getId(),
            order.getOrderNumber(),
            order.getOrderDate(),
            order.getTotalAmount()
        );
    }
}
```

**OrderMapper.java:**
```java
package com.example.demo.mapper;

import com.example.demo.dto.OrderDTO;
import com.example.demo.model.Order;

public class OrderMapper {
    
    public static OrderDTO toDTO(Order order) {
        return new OrderDTO(
            order.getId(),
            order.getOrderNumber(),
            order.getOrderDate(),
            order.getTotalAmount(),
            CustomerMapper.toSummaryDTO(order.getCustomer())
        );
    }
}
```

## Step 4: Create Service Layer

Create a new package `service` and add service interfaces and implementations:

**CustomerService.java (interface):**
```java
package com.example.demo.service;

import com.example.demo.dto.CustomerDTO;

import java.util.List;
import java.util.Optional;

public interface CustomerService {
    List<CustomerDTO> getAllCustomers();
    Optional<CustomerDTO> getCustomerById(Long id);
    CustomerDTO createCustomer(CustomerDTO customerDTO);
}
```

**CustomerServiceImpl.java:**
```java
package com.example.demo.service;

import com.example.demo.dto.CustomerDTO;
import com.example.demo.mapper.CustomerMapper;
import com.example.demo.model.Customer;
import com.example.demo.repository.CustomerRepository;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class CustomerServiceImpl implements CustomerService {
    
    private final CustomerRepository customerRepository;
    
    public CustomerServiceImpl(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }
    
    @Override
    public List<CustomerDTO> getAllCustomers() {
        List<Customer> customers = customerRepository.findAll();
        List<CustomerDTO> customerDTOs = new ArrayList<>();
        for (Customer customer : customers) {
            customerDTOs.add(CustomerMapper.toDTO(customer));
        }
        return customerDTOs;
    }
    
    @Override
    public Optional<CustomerDTO> getCustomerById(Long id) {
        Optional<Customer> customer = customerRepository.findById(id);
        if (customer.isPresent()) {
            return Optional.of(CustomerMapper.toDTO(customer.get()));
        }
        return Optional.empty();
    }
    
    @Override
    public CustomerDTO createCustomer(CustomerDTO customerDTO) {
        Customer customer = new Customer(
            customerDTO.name(),
            customerDTO.email(),
            customerDTO.phone()
        );
        Customer savedCustomer = customerRepository.save(customer);
        return CustomerMapper.toDTO(savedCustomer);
    }
}
```

**OrderService.java (interface):**
```java
package com.example.demo.service;

import com.example.demo.dto.OrderDTO;

import java.util.List;
import java.util.Optional;

public interface OrderService {
    List<OrderDTO> getAllOrders();
    Optional<OrderDTO> getOrderById(Long id);
}
```

**OrderServiceImpl.java:**
```java
package com.example.demo.service;

import com.example.demo.dto.OrderDTO;
import com.example.demo.mapper.OrderMapper;
import com.example.demo.repository.OrderRepository;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class OrderServiceImpl implements OrderService {
    
    private final OrderRepository orderRepository;
    
    public OrderServiceImpl(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
    
    @Override
    public List<OrderDTO> getAllOrders() {
        List<Order> orders = orderRepository.findAll();
        List<OrderDTO> orderDTOs = new ArrayList<>();
        for (Order order : orders) {
            orderDTOs.add(OrderMapper.toDTO(order));
        }
        return orderDTOs;
    }
    
    @Override
    public Optional<OrderDTO> getOrderById(Long id) {
        Optional<Order> order = orderRepository.findById(id);
        if (order.isPresent()) {
            return Optional.of(OrderMapper.toDTO(order.get()));
        }
        return Optional.empty();
    }
}
```

## Step 5: Update Controllers to Use DTOs

Update your controllers to use the service layer and return DTOs:

**CustomerController.java:**
```java
package com.example.demo.controller;

import com.example.demo.dto.CustomerDTO;
import com.example.demo.service.CustomerService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/customers")
public class CustomerController {
    
    private final CustomerService customerService;
    
    public CustomerController(CustomerService customerService) {
        this.customerService = customerService;
    }
    
    @GetMapping
    public List<CustomerDTO> getAllCustomers() {
        return customerService.getAllCustomers();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<CustomerDTO> getCustomerById(@PathVariable Long id) {
        Optional<CustomerDTO> customer = customerService.getCustomerById(id);
        if (customer.isPresent()) {
            return ResponseEntity.ok(customer.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }
    
    @PostMapping
    public CustomerDTO createCustomer(@RequestBody CustomerDTO customerDTO) {
        return customerService.createCustomer(customerDTO);
    }
}
```

**OrderController.java:**
```java
package com.example.demo.controller;

import com.example.demo.dto.OrderDTO;
import com.example.demo.service.OrderService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @GetMapping
    public List<OrderDTO> getAllOrders() {
        return orderService.getAllOrders();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<OrderDTO> getOrderById(@PathVariable Long id) {
        Optional<OrderDTO> order = orderService.getOrderById(id);
        if (order.isPresent()) {
            return ResponseEntity.ok(order.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

## Step 6: Remove Jackson Annotations from Entities

Now that we're using DTOs, we can remove the Jackson annotations from our entities!

**Customer.java:**
```java
package com.example.demo.model;

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
    // @JsonManagedReference <- REMOVE THIS
    private List<Order> orders = new ArrayList<>();
    
    // ... rest of the class
}
```

**Order.java:**
```java
package com.example.demo.model;

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
    // @JsonBackReference <- REMOVE THIS
    private Customer customer;
    
    // ... rest of the class
}
```

## Step 7: Test Your Refactored API

1. **Restart your application**

2. **Test Customer endpoint:** `GET http://localhost:8080/api/customers/1`


3. **Test Order endpoint:** `GET http://localhost:8080/api/orders/1`


**Notice:**
- Order now includes customer information (we control this with DTOs!)
- Customer includes order summaries (without nested customer objects)
- No circular dependencies
- Clean, predictable API responses

<!-- ## Understanding the Architecture

### Layered Architecture

```
┌─────────────────────────────────┐
│   Controller (REST API)         │  ← Returns DTOs
├─────────────────────────────────┤
│   Service (Business Logic)      │  ← Converts Entity ↔ DTO
├─────────────────────────────────┤
│   Repository (Data Access)      │  ← Works with Entities
├─────────────────────────────────┤
│   Database                      │  ← Stores data
└─────────────────────────────────┘
``` -->

**Responsibilities:**
- **Controller:** Handle HTTP requests, return DTOs
- **Service:** Business logic, entity ↔ DTO conversion
- **Repository:** Database operations with entities
- **Mapper:** Pure conversion logic (entity ↔ DTO)

## Exercise: Add More DTO Endpoints

Add these additional features:

### 1. Create Order via API

Add to `OrderService`:
```java
OrderDTO createOrder(String orderNumber, LocalDate orderDate, Double totalAmount, Long customerId);
```

Add to `OrderController`:
```java
@PostMapping
public ResponseEntity<OrderDTO> createOrder(@RequestBody CreateOrderRequest request) {
    // Implement this
}
```

Create a `CreateOrderRequest` record:
```java
public record CreateOrderRequest(
    String orderNumber,
    LocalDate orderDate,
    Double totalAmount,
    Long customerId
) {}
```

### 2. Customer Statistics DTO

Create a new DTO:
```java
public record CustomerStatsDTO(
    Long id,
    String name,
    int totalOrders,
    Double totalSpent
) {}
```

Add method to calculate this from a `Customer` entity.

## Best Practices

1. **Never expose entities directly** - Always use DTOs for API responses
2. **Keep DTOs simple** - No business logic in DTOs
3. **Use Records for DTOs** - Perfect for immutable data transfer
4. **Service layer handles conversion** - Controllers shouldn't know about entities
5. **Different DTOs for different views** - `CustomerDTO` vs `CustomerSummaryDTO`
6. **Mappers are pure functions** - No dependencies, easy to test


## What You Learned

- Why DTOs are important (security, stability, flexibility)  
- How to use Java Records to create immutable DTOs  
- How to create mapper classes for entity ↔ DTO conversion  
- How to implement a service layer for business logic  
- Layered architecture (Controller → Service → Repository)  
- How DTOs eliminate the need for Jackson circular dependency annotations  
- How to control exactly what data your API exposes