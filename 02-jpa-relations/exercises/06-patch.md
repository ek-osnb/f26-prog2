# Using PATCH for Partial Updates

**PATCH** requests are used to apply partial modifications to a resource. Unlike PUT requests, which require the entire resource representation, PATCH requests allow you to send only the fields that need to be updated.

We will only implement patching for the `status` field of an order for simplicity.


## Step 1: Create a DTO for the PATCH request
Create a new record `OrderUpdateDto` in the `ek.osnb.jpa.orders.dto` package:

```java
public record OrderUpdateDto(String status) {
}
```
This DTO will be used to capture the status update from the client.

## Step 2: Add a method in the service interface and implementation
In the `OrderService` interface, add a new method for updating the order status:

```java
OrderDto updateOrderStatus(Long id, OrderUpdateDto orderUpdateDto);
```

In the `OrderServiceImpl` class, implement the `updateOrderStatus` method:

```java
@Override
public OrderDto updateOrderStatus(Long id, OrderUpdateDto orderUpdateDto) {
    Optional<Order> existingOrder = orderRepository.findById(id);
    if (existingOrder.isEmpty()) {
        throw new RuntimeException("Order not found with id: " + id);
    }
    Order order = existingOrder.get();
    order.setStatus(OrderStatus.valueOf(orderUpdateDto.status()));
    Order updatedOrder = orderRepository.save(order);
    return OrderMapper.toDto(updatedOrder);
}
```

## Step 3: Add a PATCH endpoint in the controller
In the `OrderController` class, add a new method to handle PATCH requests for updating the order status:

```java
@PatchMapping("/{id}")
public ResponseEntity<OrderDto> updateOrderStatus(@PathVariable Long id, @RequestBody OrderUpdateDto orderUpdateDto) {
    try {
        return ResponseEntity.ok(orderService.removeOrderLine(id, orderLineId));
    } catch (RuntimeException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
    }
}
```

## Step 4: Testing the PATCH endpoint
You can test the PATCH endpoint using `curl` or Postman. Here is an example `curl` command to update the status of an order:
```bash
curl -X PATCH http://localhost:8080/api/orders/1 \
-H "Content-Type: application/json" \
-d '{"status": "SHIPPED"}'
```
This command updates the status of the order with ID 1 to "SHIPPED". You should receive a response with the updated order details.

## Step 5 (Optional): Pushing to GitHub
Open the terminal in IntelliJ and run the following commands to push your changes to GitHub:

Add the changes to the staging area:
```bash
git add .
```

Commit the changes with a meaningful message:
```bash
git commit -m "Completed exercise 06"
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


## Extra Notes:

Usually when doing `PATCH` requests, it should be possible to update multiple fields at once. This can be achieved by creating a more comprehensive DTO that includes all the fields that can be updated. For example, you could create an `OrderUpdateDto` that includes fields for `orderDate`, `status`, and `orderLines`. The service method would then check which fields are present in the DTO and update only those fields in the existing order.

This is left as an optional exercise for you to implement.