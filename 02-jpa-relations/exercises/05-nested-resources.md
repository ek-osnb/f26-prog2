# Adding endpoints for nested resources

Until now, if we want to add/remove an orderLine from an order, we need to send the whole order object with the updated list of orderLines. This is not very convenient, especially if we want to add/remove a single orderLine from an order.

We want to be able to add/remove an orderLine from an order using a dedicated endpoint. For example, we want to be able to do the following:
- `POST /api/orders/{orderId}/order-lines` to add a new orderLine to an order
- `DELETE /api/orders/{orderId}/order-lines/{orderLineId}` to remove an orderLine from an order


## Step 1: Adding the methods to the service interface and implementation

First, we need to add the methods to the service interface `OrderService`:

```java
void addOrderLine(Long orderId, OrderLineDto orderLineDto);
void removeOrderLine(Long orderId, Long orderLineId);
```

Next, we need to implement these methods in the `OrderServiceImpl` class:

```java
@Override
public OrderDto addOrderLine(Long id, OrderLineDto orderLineDto) {
    Optional<Order> order = orderRepository.findById(id);
    if (order.isEmpty()) {
        throw new RuntimeException("Order not found with id: " + id);
    }
    Order existingOrder = order.get();
    var line = OrderMapper.toEntity(orderLineDto);
    existingOrder.addOrderLine(line);
    return OrderMapper.toDto(orderRepository.save(existingOrder));

}

@Override
public OrderDto removeOrderLine(Long id, Long orderLineId) {
    Optional<Order> order = orderRepository.findById(id);
    if (order.isEmpty()) {
        throw new RuntimeException("Order not found with id: " + id);
    }
    Order existingOrder = order.get();
    OrderLine lineToRemove = null;

    for (var line : existingOrder.getOrderLines()) {
        if (line.getId().equals(orderLineId)) {
            lineToRemove = line;
            break;
        }
    }
    if (lineToRemove == null) {
        throw new RuntimeException(
                "OrderLine not found with id: " + orderLineId + " in order " + id
        );
    }
    existingOrder.removeOrderLine(lineToRemove);
    return OrderMapper.toDto(orderRepository.save(existingOrder));
}
```

Make sure that you understand how these methods work.

## Step 2: Adding the endpoints to the controller
Next, we need to add the endpoints to the `OrderController` class:

```java
@PostMapping("/{id}/order-lines")
public ResponseEntity<OrderDto> addOrderLine(@PathVariable Long id, @RequestBody OrderLineDto orderLineDto) {
    try {
        return ResponseEntity.ok(orderService.addOrderLine(id, orderLineDto));
    } catch (RuntimeException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
    }
}

@DeleteMapping("/{id}/order-lines/{orderLineId}")
public ResponseEntity<OrderDto> removeOrderLine(@PathVariable Long id, @PathVariable Long orderLineId) {
    try {
        return ResponseEntity.ok(orderService.removeOrderLine(id, orderLineId));
    } catch (RuntimeException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
    }
}
```

Now you can test the endpoints using the following `curl` commands:
To add an order line:
```bash
curl -X POST http://localhost:8080/api/orders/1/order-lines \
-H "Content-Type: application/json" \
-d '{
    "product": "Product D",
    "unitPrice": 30.0,
    "quantity": 2
}'
```

To remove an order line (replace `1` with the actual order line ID you want to remove):
```bash
curl -X DELETE http://localhost:8080/api/orders/1/order-lines/1
```

## Step 3 (Optional): Pushing to GitHub
Open the terminal in IntelliJ and run the following commands to push your changes to GitHub:

Add the changes to the staging area:
```bash
git add .
```

Commit the changes with a meaningful message:
```bash
git commit -m "Completed exercise 05"
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