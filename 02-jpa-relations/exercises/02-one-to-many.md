# Spring Data Jpa - One to Many Relationship

In this exercise, we will add a one-to-many relationship between two entities: `Order` and `OrderLine`. Previously, you created a many-to-one relationship from `OrderLine` to `Order`. Now, we will add the reverse side of the relationship. This is known as a bidirectional relationship.

## Step 1: Add the OneToMany annotation
In the `Order` class, add a field to represent the list of order lines. Use the `@OneToMany` annotation to define the relationship. The `mappedBy` attribute should point to the field in the `OrderLine` class that owns the relationship.

```java
@OneToMany(mappedBy = "order")
private List<OrderLine> orderLines = new ArrayList<>();
```

Add getters and setters for the `orderLines` field as well.

**Note:** The `mappedBy` attribute indicates that the `Order` entity is not the owner of the relationship. The `OrderLine` entity owns the relationship, as it contains the foreign key.

## Step 2: Running the application
If you run the application, and try `GET /api/orders` in your browser, you will notice that you will get a weird result. It may look like:

```json
{"id":1,"orderDate":"2025-08-28","status":"PAID","orderLines":[{"id":1,"product":"Product A","unitPrice":50.0,"quantity":2,"order":{"id":1,"orderDate":"2025-08-28","status":"PAID","orderLines":[{"id":1,"product":"Product A","unitPrice":50.0,"quantity":2,"order":{"id":1,"orderDate":"2025-08-28","status":"PAID","orderLines":[{"id":1,"product":"Product A","unitPrice":50.0,"quantity":2,"order":{"id":1,"orderDate":"2025-08-28","status":"PAID","orderLines":[{"id":1,"product":"Product A","unitPrice":50.0,"quantity":2,"order":{"id":1,"orderDate":"2025-08-28","status":"PAID","orderLines":[{"id":1,"product":"Product A","unitPrice":50.0,"quantity":2,"order":{"id":1,
...
```

This is called infinite recursion. It happens because when Jackson (the library that Spring Boot uses to convert Java objects to JSON) tries to serialize the `Order` object, it sees the `orderLines` field and tries to serialize it as well. Then, it sees the `order` field in each `OrderLine` and tries to serialize it again, leading to an infinite loop.

## Step 3: Preventing infinite recursion
To prevent this, we can use the `@JsonManagedReference` and `@JsonBackReference` annotations from the `com.fasterxml.jackson.annotation` package.

In the `Order` class, annotate the `orderLines` field with `@JsonManagedReference`:

```java
@JsonManagedReference
@OneToMany(mappedBy = "order")
private List<OrderLine> orderLines = new ArrayList<>();
```

In the `OrderLine` class, annotate the `order` field with `@JsonBackReference`:

```java
@JsonBackReference
@ManyToOne
private Order order;
```

If you run the application again and try `GET /api/orders`, you should now see a proper JSON response without infinite recursion:

```json
[
    {
        "id": 1,
        "orderDate": "2025-08-28",
        "status": "PAID",
        "orderLines": [
            {
            "id": 1,
            "product": "Product A",
            "unitPrice": 50,
            "quantity": 2
            },
            {
            "id": 2,
            "product": "Product B",
            "unitPrice": 30,
            "quantity": 1
            }
        ]
    },
]
```

## Step 4: Adding a helper method
It is considered a good practice to keep both sides of a bidirectional relationship in sync. This means that when you add an `OrderLine` to an `Order`, you should also set the `Order` in the `OrderLine`. 

Instead of directly manipulating the `orderLines` list, you can create a method in the `Order` class to add an `OrderLine`:

```java
public void addOrderLine(OrderLine orderLine) {
    orderLines.add(orderLine);
    orderLine.setOrder(this);
}
```
Similarly, you can create a method to remove an `OrderLine`:

```java
public void removeOrderLine(OrderLine orderLine) {
    orderLines.remove(orderLine);
    orderLine.setOrder(null);
}
```

For completness, you can also add a method to clear all order lines:

```java
public void clearOrderLines() {
    for (OrderLine orderLine : new ArrayList<>(orderLines)) {
        removeOrderLine(orderLine);
    }
}
```

Also remove the `setOrderLines()` method to avoid manipulating the list directly.

## Step 5: Keeping the relationship in sync
With the helper methods in place, you can now update your run method in the `InitData` class to use the `addOrderLine` method:

Replace this:
```java
// Add the relationship
orderLine1.setOrder(order1);
orderLine2.setOrder(order1);
orderLine3.setOrder(Order2);
orderLine4.setOrder(Order2);
orderLine5.setOrder(Order2);
```

With this:
```java
// Add the relationship
order1.addOrderLine(orderLine1);
order1.addOrderLine(orderLine2);
order2.addOrderLine(orderLine3);
order2.addOrderLine(orderLine4);
order2.addOrderLine(orderLine5);
```

This ensures that both sides of the relationship are kept in sync, and it will avoid stale data.

## Step 6 (Optional): Pushing to GitHub
Open the terminal in IntelliJ and run the following commands to push your changes to GitHub:

Add the changes to the staging area:
```bash
git add .
```

Commit the changes with a meaningful message:
```bash
git commit -m "Completed exercise 02"
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