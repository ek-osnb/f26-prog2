# Adding dtos

So far, we have been exposing our entities directly through the REST API. While this is acceptable for simple applications, it is generally not a good practice for production applications. Exposing entities directly can lead to several issues, including:
- **Security**: Sensitive information in the entity might be exposed unintentionally.
- **Tight coupling**: Changes to the entity structure can break the API contract.
- **Over-fetching/Under-fetching**: Clients might receive more or less data than they need.

To address these issues, we can use Data Transfer Objects (DTOs). DTOs are simple objects that are used to transfer data between layers of an application. They do not contain any business logic and are typically used to shape the data that is sent to and received from the client.

**We will use DTOs to transfer data between the service layer and the controller layer. This way, we can control exactly what data is exposed through the API and how it is structured.**

## Step 1: Creating DTOs

Create a new package `dto` in the `ek.osnb.jpa.orders` package. Inside the `dto` package, create two records: `OrderDto` and `OrderLineDto`.

```java
public record OrderDto(Long id, LocalDate orderDate, String status, List<OrderLineDto> orderLines) {
}
```

```java
public record OrderLineDto(Long id, double unitPrice, int quantity, String product) {
}
```

## Step 2: Creating a mapper
To transform between entities and DTOs, we will create a mapper class in the `ek.osnb.jpa.orders.dto` package. Create a new class `OrderMapper` with the following methods:

```java
public class OrderMapper {
        public static OrderDto toDto(Order order) {
        List<OrderLineDto> orderLines = new ArrayList<>();

        for (var line : order.getOrderLines()) {
            orderLines.add(toDto(line));
        }

        return new OrderDto(order.getId(), order.getOrderDate(), order.getStatus().name(), orderLines);
    }

    public static OrderLineDto toDto(OrderLine orderLine) {
        return new OrderLineDto(orderLine.getId(), orderLine.getUnitPrice(), orderLine.getQuantity(), orderLine.getProduct());
    }

    public static Order toEntity(OrderDto orderDto) {
        Order order = new Order();
        order.setOrderDate(orderDto.orderDate());
        order.setStatus(OrderStatus.valueOf(orderDto.status()));

        for (var lineDto : orderDto.orderLines()) {
            var line = toEntity(lineDto);
            order.addOrderLine(line);
        }

        return order;
    }

    public static OrderLine toEntity(OrderLineDto orderLineDto) {
        OrderLine orderLine = new OrderLine();
        orderLine.setProduct(orderLineDto.product());
        orderLine.setUnitPrice(orderLineDto.unitPrice());
        orderLine.setQuantity(orderLineDto.quantity());
        return orderLine;
    }
}
```

## Step 3: Updating the service interface and implementation
In the `OrderService` interface, update the methods to use the `OrderDto` instead:

```java
public interface OrderService {
    List<OrderDto> getAllOrders(OrderStatus status);
    OrderDto getOrderById(Long id);
    OrderDto createOrder(OrderDto orderDto);
    OrderDto updateOrder(Long id, OrderDto orderDto);
    void deleteOrder(Long id);
}
```

In the `OrderServiceImpl` class, to return the correct types, and update the methods to use the `OrderMapper` to convert between entities and DTOs:

```java
@Service
public class OrderServiceImpl implements OrderService {

    private final OrderRepository orderRepository;
    private final OrderLineRepository orderLineRepository;

    public OrderServiceImpl(OrderRepository orderRepository, OrderLineRepository orderLineRepository) {
        this.orderRepository = orderRepository;
        this.orderLineRepository = orderLineRepository;
    }

    @Override
    public List<OrderDto> getAllOrders(OrderStatus status) {
        List<Order> orders = orderRepository.findAll();
        List<OrderDto> orderDtos = new ArrayList<>();
        for (var order : orders) {
            orderDtos.add(OrderMapper.toDto(order));
        }
        return orderDtos;
    }

    @Override
    public OrderDto getOrderById(Long id) {
        Optional<Order> order = orderRepository.findById(id);
        if (order.isPresent()) {
            return OrderMapper.toDto(order.get());
        }
        throw new RuntimeException("Order not found with id: " + id);
    }

    @Override
    public OrderDto createOrder(OrderDto orderDto) {
        Order order = OrderMapper.toEntity(orderDto);
        order.setId(null); // Ensure the ID is null for new entities
        return OrderMapper.toDto(orderRepository.save(order));
    }

    @Override
    public OrderDto updateOrder(Long id, OrderDto orderDto) {
        Optional<Order> existingOrder = orderRepository.findById(id);
        if (existingOrder.isPresent()) {
            Order order = OrderMapper.toEntity(orderDto);
            Order updatedOrder = existingOrder.get();
            updatedOrder.setOrderDate(order.getOrderDate());
            updatedOrder.setStatus(order.getStatus());
            updatedOrder.clearOrderLines();
            for (var line : order.getOrderLines()) {
                updatedOrder.addOrderLine(line);
            }
            return OrderMapper.toDto(orderRepository.save(updatedOrder));
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
}
```

## Step 4: Updating the controller
Finally, update the `OrderController` to use the `OrderDto` instead of the `Order` entity:

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    private final OrderService orderService;
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping
    public ResponseEntity<List<OrderDto>> getAllOrders(@RequestParam(required = false) OrderStatus status) {
        return ResponseEntity.ok(orderService.getAllOrders(status));
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderDto> getOrderById(@PathVariable Long id) {
        try {
            return ResponseEntity.ok(orderService.getOrderById(id));
        } catch (RuntimeException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
        }
    }

    @PostMapping
    public ResponseEntity<OrderDto> createOrder(@RequestBody OrderDto orderDto) {
        return ResponseEntity.status(HttpStatus.CREATED).body(orderService.createOrder(orderDto));
    }

    @PutMapping("/{id}")
    public ResponseEntity<OrderDto> updateOrder(@PathVariable Long id, @RequestBody OrderDto orderDto) {
        try {
            return ResponseEntity.ok(orderService.updateOrder(id, orderDto));
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
}
```

Take note that the controller methods now return and accept `OrderDto` objects instead of `Order` entities.

## Step 5: Removing `@JsonManagedReference` and `@JsonBackReference`
Since we are no longer exposing the entities directly, we can remove the `@JsonManagedReference` and `@JsonBackReference` annotations from the `Order` and `OrderLine` classes.

The dtos make sure that we only expose the necesarry data, and we avoid issues with infinite recursion in JSON serialization.


## Step 6: Testing the application
Run the application and test the endpoints using a tool like Postman or curl. You should be able to create, retrieve, update, and delete orders using the DTOs.

## Step 7 (Optional): Pushing to GitHub
Open the terminal in IntelliJ and run the following commands to push your changes to GitHub:

Add the changes to the staging area:
```bash
git add .
```

Commit the changes with a meaningful message:
```bash
git commit -m "Completed exercise 04"
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