# Spring Data JPA - Cascading types

In Spring Data JPA, cascading is a way to propagate certain operations from a parent entity to its related child entities. This is particularly useful in one-to-many and many-to-one relationships, where you want to ensure that changes made to the parent entity are automatically reflected in the child entities.

## Parent-Child Relationship
In a parent-child relationship, the parent entity is the one that holds the reference to the child entities. For example, in a one-to-many relationship between `Order` and `OrderLine`, the `Order` entity is the parent, and the `OrderLine` entities are the children.

If the there is no `Order`, then there should not be any `OrderLine` either. Therefore, if we delete an `Order`, we also want to delete all its associated `OrderLine` entities. This is where cascading comes into play.

It is furthermore convienient to cascade the `PERSIST` operation, so that when we save an `Order`, all its associated `OrderLine` entities are also saved automatically.

## Step 1: Add CascadeType to OneToMany relationship
In the `Order` class, modify the `@OneToMany` annotation to include the `cascade` attribute. Set it to `CascadeType.PERSIST` to ensure that when an `Order` is saved, all its associated `OrderLine` entities are also saved.

```java
@JsonManagedReference
@OneToMany(mappedBy = "order", cascade = CascadeType.PERSIST)
private List<OrderLine> orderLines = new ArrayList<>();
```

## Step 2: Updating the service layer
In the `OrderService` class, update the `updateOrder` method to save the `Order` along with its `OrderLine` entities. You can do this by adding the `OrderLine` entities to the `Order` before saving it.
```java
@Override
public Order updateOrder(Long id, Order order) {
    Optional<Order> existingOrder = orderRepository.findById(id);
    if (existingOrder.isPresent()) {
        Order updatedOrder = existingOrder.get();
        updatedOrder.setOrderDate(order.getOrderDate());
        updatedOrder.setStatus(order.getStatus());

        updatedOrder.clearOrderLines();
        for (var line : order.getOrderLines()) {
            updatedOrder.addOrderLine(line);
        }
        return orderRepository.save(updatedOrder);
    }
    throw new RuntimeException("Order not found with id: " + id);
}
```

## Step 3: Testing the cascading behavior
You can test the cascading behavior by creating a new `Order` with a single `OrderLine` entity:

Try the following `curl` command:
```bash
curl -X POST http://localhost:8080/api/orders \
-H "Content-Type: application/json" \
-d '{
    "orderDate": "2025-08-30",
    "status": "PENDING",
    "orderLines": [
        {
            "product": "Product C",
            "unitPrice": 20.0,
            "quantity": 5
        }
    ]
}'
```

This should create a new `Order` along with its associated `OrderLine` entity. You can verify this by checking the database or by retrieving the order using a `GET` request.

## Step 4: Testing update
If you want to update an existing order, you may try the following curl command:
```bash
curl -X PUT http://localhost:8080/api/orders/1 \
-H "Content-Type: application/json" \
-d '{
    "orderDate": "2025-08-31",
    "status": "SHIPPED",
    "orderLines": [
        {
            "product": "Product 10",
            "unitPrice": 15.0,
            "quantity": 3
        }
    ]
}'
```

If you now try `GET /api/orders/1`, you should see that the order hasn't been updated. This is because, the cascade type `PERSIST` only applies to new entities being persisted, not to existing entities being updated. To achieve cascading behavior during updates, you need to add `CascadeType.MERGE` to the cascade attribute in the `@OneToMany` annotation.

```java
@JsonManagedReference
@OneToMany(mappedBy = "order", cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private List<OrderLine> orderLines = new ArrayList<>();
```

If you now try the `PUT` request again, followed by a `GET /api/orders/1`, you should see that the order has been updated along with its associated `OrderLine` entity.

## Step 5: Cascading delete
To enable cascading delete, you need to add `CascadeType.REMOVE` to the cascade attribute in the `@OneToMany` annotation. This will ensure that when an `Order` is deleted, all its associated `OrderLine` entities are also deleted.

```java
@JsonManagedReference
@OneToMany(mappedBy = "order", cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REMOVE})
private List<OrderLine> orderLines = new ArrayList<>();
```

Now, if you delete an `Order` using a `DELETE` request, all its associated `OrderLine` entities will also be deleted.

## Step 6 (Optional): Pushing to GitHub
Open the terminal in IntelliJ and run the following commands to push your changes to GitHub:

Add the changes to the staging area:
```bash
git add .
```

Commit the changes with a meaningful message:
```bash
git commit -m "Completed exercise 03"
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