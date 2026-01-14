# Spring Data JPA: ManyToOne


In this exercise, you will learn how to define a ManyToOne relationship using Spring Data JPA. You will work with two entities: `Order` and `OrderLine`. An `Order` can have multiple `OrderLine` items, but each `OrderLine` is associated with only one `Order`.


## Step 1: Configuring the database
We will need to configure the database connection in the `application.properties` file located in `src/main/resources`.

Add the following properties to configure an in-memory H2 database:
```properties
spring.application.name=spring-jpa-relations

# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 Console Configuration
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA & Hibernate Configuration
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true
```

**Run the application to see if everything is working fine.**

## Step 2: From models to Entities

We want to make the following models an **Entity**:
- `Order`
- `OrderLine`

We will use the `BaseEntity` class to avoid code duplication. You can search for the class in **IntelliJ** by pressing `Shift` key twice, and type `BaseEntity` to find the class.

Because `Order` is a reserved keyword in SQL, we will rename the table to `orders` using the `@Table` annotation.

```java
@Entity
@Table(name = "orders")
public class Order extends BaseEntity {
    // ...
}
```

And similarly for `OrderLine`:
```java
@Entity
public class OrderLine extends BaseEntity {
    // ...
}
```

**Run the application to see if everything is working fine.**

Then navigate to the H2 console at (in your browser) `http://localhost:8080/h2-console`, and connect to the database. You should see the tables `orders` and `order_line` created in the database.

## Step 3: Define a ManyToOne relationship

We want to define a many-to-one relationship between the `OrderLine` and `Order` entities.

### How to define a ManyToOne relationship
You will use the `@ManyToOne` annotation to define many-to-one relationships between entities.

**Example:** If entity `A` has a many-to-one relationship with entity `B`, then the class `A` will have a field of type `B`:
```java
@Entity
public class A {
    // ...
    @ManyToOne
    private B b;
    // ...
}
```
**NOTE:** Class `A` has the responsibility to maintain the relationship!


### Your task
Define the following relationships between the entities:
- An `OrderLine` has a many-to-one relationship with an `Order`. This means that many order lines can be associated with one order.

**Run the application to see if everything is working fine.**

Check the H2 console to see if the foreign key column `order_id` has been created in the `order_line` table.

## Step 4: Adding getter and setter methods for the relationship

In the OrderLine class, add getter and setter methods for the `Order` field. We will need these methods to set and retrieve the associated order for an order line.


## Step 5: Adding repository access

To access the database from your service layer, you will need to define the repository interfaces.
You can create repository interfaces by extending the `JpaRepository` interface provided by Spring Data JPA.

**Example:**
```java
public interface ARepository extends JpaRepository<A, Long> {
}
```

**NOTE:** Inside the diamond operator `<>`, the first parameter is the entity type, and the second parameter is the type of the entity's primary key.

At runtime, Spring Data JPA will automatically provide the implementation for this interface - so we don't need to implement it ourselves.

### Your task
Make the following interfaces extend `JpaRepository`:
- `OrderRepository` for the `Order` entity.
- `OrderLineRepository` for the `OrderLine` entity.

**Run the application to see if everything is working fine.**

## Step 6: Adding sample data
The `InitData` class is already extending the `CommandLineRunner` interface, which means that the `run` method will be executed when the application starts.

**NOTE** The `CommandLineRunner` interface is used to execute code after the Spring Boot application has started. This is useful for initializing data or performing setup tasks, after all beans are created and the application context is ready.


### Your task
In the `InitData` class, we want to add sample data to the database.

You will need to inject the `OrderRepository`, and `OrderLineRepository` into the `InitData` class using constructor injection.

```java
private final OrderRepository orderRepository;
private final OrderLineRepository orderLineRepository;

public InitData(OrderRepository orderRepository, OrderLineRepository orderLineRepository) {
    this.orderRepository = orderRepository;
    this.orderLineRepository = orderLineRepository;
}
```

To add sample data, you can create an `Order` object, and multiple `OrderLine` objects associated with that order. Then save them to the database using the respective repositories.

```java
@Override
public void run(String... args) throws Exception {
    Order order1 = new Order(LocalDate.now(), OrderStatus.PAID);
    Order Order2 = new Order(LocalDate.now(), OrderStatus.PAID);
    orderRepository.save(order1);
    orderRepository.save(Order2);
    // Or use orderRepository.saveAll(List.of(order1, order2));

    OrderLine orderLine1 = new OrderLine("Product A", 50.0, 2);
    OrderLine orderLine2 = new OrderLine("Product B", 30.0, 1);

    OrderLine orderLine3 = new OrderLine("Product C", 20.0, 5);
    OrderLine orderLine4 = new OrderLine("Product D", 15.0, 3);
    OrderLine orderLine5 = new OrderLine("Product E", 25.0, 4);

    // Add the relationship
    orderLine1.setOrder(order1);
    orderLine2.setOrder(order1);
    orderLine3.setOrder(Order2);
    orderLine4.setOrder(Order2);
    orderLine5.setOrder(Order2);

    orderLineRepository.saveAll(List.of(orderLine1, orderLine2, orderLine3, orderLine4, orderLine5));
}
```

**Run the application to see if everything is working fine.**

Check the H2 console to see if the sample data has been added to the `orders` and `order_line` tables.

## Step 7: Adding service layer
The service layer is responsible for the business logic of the application. It acts as an intermediary between the controller layer and the repository layer.

In the repo you will notice that there is a service interface called `OrderService`. This interface defines the contract for the service layer.

Make the `OrderServiceImpl` class implement the `OrderService` interface, and add the `@Service` annotation to the class.

```java
@Service
public class OrderServiceImpl implements OrderService {
    // ...
}
```

Use constructor injection to inject the `OrderRepository` and `OrderLineRepository` into the `OrderServiceImpl` class.

```java
private final OrderRepository orderRepository;
private final OrderLineRepository orderLineRepository;

public OrderServiceImpl(OrderRepository orderRepository, OrderLineRepository orderLineRepository) {
    this.orderRepository = orderRepository;
    this.orderLineRepository = orderLineRepository;
}
```

## Step 8: Implementing service methods

Implement the methods defined in the `OrderService` interface in the `OrderServiceImpl` class.

```java
@Override
    public List<Order> getAllOrders(OrderStatus status) {
        return orderRepository.findAll();
    }

    @Override
    public Order getOrderById(Long id) {
        Optional<Order> order = orderRepository.findById(id);
        if (order.isPresent()) {
            return order.get();
        }
        throw new RuntimeException("Order not found with id: " + id);
    }

    @Override
    public Order createOrder(Order order) {
        order.setId(null); // Ensure the ID is null for new entities
        return orderRepository.save(order);
    }

    @Override
    public Order updateOrder(Long id, Order order) {
        Optional<Order> existingOrder = orderRepository.findById(id);
        if (existingOrder.isPresent()) {
            Order updatedOrder = existingOrder.get();
            updatedOrder.setOrderDate(order.getOrderDate());
            updatedOrder.setStatus(order.getStatus());
            // Update other fields as necessary
            return orderRepository.save(updatedOrder);
        }
        throw new RuntimeException("Order not found with id: " + id);
    }

    @Override
    public void deleteOrder(Long id) {
        if (orderRepository.existsById(id)) {
            orderRepository.deleteById(id);
        } else {
            throw new RuntimeException("Order not found with id: " + id);
        }
    }
```

Take a moment to understand how each method works.

**Run the application to see if everything is working fine.**

## Step 9: Injecting the service layer into the controller layer
The controller layer is responsible for handling HTTP requests and responses. It acts as an intermediary between the client and the service layer.

Use constructor injection to inject the `OrderService` (interface) into the `OrderController` class.

```java
private final OrderService orderService;
public OrderController(OrderService orderService) {
    this.orderService = orderService;
}
```

## Step 10: Implementing controller methods

Implement the methods defined in the `OrderController` class to handle HTTP requests.

```java
    @GetMapping
    public ResponseEntity<List<Order>> getAllOrders(@RequestParam(required = false) OrderStatus status) {
        return ResponseEntity.ok(orderService.getAllOrders(status));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        try {
            return ResponseEntity.ok(orderService.getOrderById(id));
        } catch (RuntimeException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
        }
    }

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        return ResponseEntity.status(HttpStatus.CREATED).body(orderService.createOrder(order));
    }

    @PutMapping("/{id}")
    public ResponseEntity<Order> updateOrder(@PathVariable Long id, @RequestBody Order order) {
        try {
            return ResponseEntity.ok(orderService.updateOrder(id, order));
        } catch (RuntimeException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteOrder(@PathVariable Long id) {
        try {
            orderService.deleteOrder(id);
            return ResponseEntity.noContent().build();
        } catch (RuntimeException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).build();
        }
    }
```

Take a moment to understand how each method works.

**Questions:**
- Why do we use `ResponseEntity` in the controller methods?
- Why do we use `try-catch` blocks in some controller methods?

**Run the application to see if everything is working fine.**

## Step 11: Mapping Enums

In the database, you will see that status is mapped to an integer, this is because we haven't specified that we want to use a `String`. To achieve this, add the following annotation to your `OrderStatus`:

```java
@Entity
@Table(name ="orders")
public class Order extends BaseEntity {

    // Other fields...

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    // Other fields, constructors, getters, and setters
}
```

If you run your application now, and open `http://localhost:8080/h2-console`, you will notice that the `status` field is now mapped to a string in the database.

## Step 12 (Optional): Pushing to GitHub
Open the terminal in IntelliJ and run the following commands to push your changes to GitHub:

Add the changes to the staging area:
```bash
git add .
```

Commit the changes with a meaningful message:
```bash
git commit -m "Completed exercise 01"
```

Push the changes to the remote repository:
```bash
git push -u origin main
```

To see the status of your git repository, you can run:
```bash
git status
```

There should be no changes left to commit.


## Conclusion

Test the API endpoints using a tool like Postman or curl.

**Notice** that in the response of the `GET /api/orders` and `GET //api/orders/{id}` endpoints, the `orderLines` field is not included. This is beacause the `Order` entity does not have a reference to the `OrderLine` entity. We could create a controller and service layer for the `OrderLine` entity to manage the order lines separately. But for the user of the API, it would be more convenient to have the order lines included in the order response. We will learn how to do this in the next exercise.

