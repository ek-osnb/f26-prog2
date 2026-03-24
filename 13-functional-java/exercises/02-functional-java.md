# Exercises: Functional Java and Streams

In this exercise you will practice the use of lambda expressions and the Streams API in Java. You will implement several functions that utilize these concepts to manipulate collections of data.


## Exercise 1: Coding Bat
- Visit [Coding Bat - Functional1](https://codingbat.com/java/Functional-1) and solve the first 5 problems.
- Visit [Coding Bat - Functional2](https://codingbat.com/java/Functional-2) and solve the first 5 problems.

**Notice that codebat does not provide `.toList()` method, so instead you need to use `.collect(Collectors.toList())` to convert streams back to lists.**

## Exercise 2: Orders, OrderLines and Products

**1. Create a new Maven project (not Spring Boot) named `functional-java-exercises`.**


**2. Inside the Main class add the following:**

```java
public class Main {

    record Product(String productId, String category, double price) { }
    record OrderLine(Product product, int quantity) { }
    record Order(String orderId, List<OrderLine> orderLines) { }

    public static List<Order> setup() {
        List<Product> products = List.of(
            new Product("A100", "Electronics", 199.99),
            new Product("B200", "Books", 29.99),
            new Product("C300", "Clothing", 49.99),
            new Product("D400", "Electronics", 99.99),
            new Product("E500", "Books", 15.99)
        );

        List<Order> orders = List.of(
            new Order("O1", List.of(
                new OrderLine(products.get(0), 1),
                new OrderLine(products.get(1), 2)
            )),
            new Order("O2", List.of(
                new OrderLine(products.get(2), 3),
                new OrderLine(products.get(3), 1)
            )),
            new Order("O3", List.of(
                new OrderLine(products.get(4), 5),
                new OrderLine(products.get(1), 1)
            ))
        );
        return orders;
    }

    public static void main(String[] args) {
        List<Order> orders = setup();

        // TODO: Implement the functions below and call them here to test
    }

    
    // 1. Calculate the total price of all orders.
    // 2. Find all products in the "Books" category.
    // 3. Get a list of all unique categories from the products.
    // 4. Find the most expensive product.
    // 5. Calculate the average price of products in the "Electronics" category.
    // 6. Get a list of order IDs where the total order price exceeds $100.
    // 7. Count how many products are there in each category.
}
```

**3. Implement the following functions using lambda expressions and the Streams API:**
1. **Calculate the total price of all orders.**

2. **Find all products in the "Books" category.**

3. **Get a list of all unique categories from the products.**

4. **Find the most expensive product.**

5. **Calculate the average price of products in the "Electronics" category.**

6. **Get a list of order IDs where the total order price exceeds $100.**

7. **Count how many products are there in each category.**


<!-- **SOLUTIONS**
```java
// 1. Calculate the total price of all orders.
public static double calculateTotalOrderPrice(List<Order> orders) {
    return orders.stream()
            .flatMap(order -> order.orderLines().stream())
            .mapToDouble(line -> line.product().price() * line.quantity())
            .sum();
}

// 2. Find all products in the "Books" category.
public static List<Product> findBooksCategoryProducts(List<Order> orders) {
    return orders.stream()
            .flatMap(order -> order.orderLines().stream())
            .map(OrderLine::product)
            .filter(product -> "Books".equals(product.category()))
            .distinct()
            .toList();
}

// 3. Get a list of all unique categories from the products.
public static List<String> getUniqueCategories(List<Order> orders) {
    return orders.stream()
            .flatMap(order -> order.orderLines().stream())
            .map(line -> line.product().category())
            .distinct()
            .toList();
}

// 4. Find the most expensive product.
public static Optional<Product> findMostExpensiveProduct(List<Order> orders) {
    return orders.stream()
            .flatMap(order -> order.orderLines().stream())
            .map(OrderLine::product)
            .max(Comparator.comparingDouble(Product::price));
}

// 5. Calculate the average price of products in the "Electronics" category.
public static double calculateAverageElectronicsPrice(List<Order> orders) {
    return orders.stream()
            .flatMap(order -> order.orderLines().stream())
            .map(OrderLine::product)
            .filter(product -> "Electronics".equals(product.category()))
            .mapToDouble(Product::price)
            .average()
            .orElse(0.0);
}

// 6. Get a list of order IDs where the total order price exceeds $100.
public static List<String> getHighValueOrderIds(List<Order> orders) {
    return orders.stream()
            .filter(order -> order.orderLines().stream()
                    .mapToDouble(line -> line.product().price() * line.quantity())
                    .sum() > 100)
            .map(Order::orderId)
            .toList();
}

// 7. Count how many products are there in each category.
public static Map<String, Long> countProductsByCategory(List<Order> orders) {
    return orders.stream()
            .flatMap(order -> order.orderLines().stream())
            .map(line -> line.product().category())
            .collect(Collectors.groupingBy(category -> category, Collectors.counting()));
} 
``` -->