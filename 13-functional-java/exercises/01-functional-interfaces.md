# Exercises: Functional Interfaces

In these exercises you will practice defining and composing your own functional interfaces, and work directly with the four fundamental functional interfaces: `Predicate`, `Function`, and `Consumer`.

---

## Exercise 1: Custom functional interface — `Validator<T>`

**1. Create a new Maven project (not Spring Boot) named `functional-interfaces-exercises`.**

**2. Define the following functional interface:**

```java
@FunctionalInterface
public interface Validator<T> {
    boolean validate(T t);

    default Validator<T> and(Validator<T> other) {
        return t -> this.validate(t) && other.validate(t);
    }

    default Validator<T> or(Validator<T> other) {
        return t -> this.validate(t) || other.validate(t);
    }

    default Validator<T> negate() {
        return t -> !this.validate(t);
    }
}
```

**3. Add the following `Product` record to your `Main` class:**

```java
record Product(String name, String category, double price, int stock) {}
```

**4. Implement the following validators as lambda expressions:**

> **Use the following pattern to apply your validators to the list of products:**
> ```java
> List<Product> result = products.stream().filter(myValidator::validate).toList();
> System.out.println(result);
> ```
> Think of `.stream().filter(...).toList()` as *"give me all elements where the validator returns true"*

1. A `Validator<Product>` that checks if the product is in the `"Electronics"` category. Use the stream pattern above to print all electronics.
2. A `Validator<Product>` that checks if the product price is less than `100.0`. Print all affordable products.
3. A `Validator<Product>` that checks if the product is in stock (`stock > 0`). Print all in-stock products.
4. Combine validators 1 and 3 using `and()` to get electronics that are in stock. Print the result.
5. Use `negate()` on validator 2 to find expensive products (price ≥ 100). Print the result.
6. Use `or()` to find products that are either `"Books"` or `"Clothing"`. Print the result.

**Use these products in your `main` method:**
```java
List<Product> products = List.of(
    new Product("Laptop",      "Electronics", 999.0,  5),
    new Product("Headphones",  "Electronics",  49.0,  0),
    new Product("Java Book",   "Books",        29.0, 10),
    new Product("T-shirt",     "Clothing",     19.0,  3),
    new Product("Smartwatch",  "Electronics",  79.0,  2)
);
```

---

## Exercise 2: `Predicate`, `Function`, and `Consumer`

Reuse the `Product` record and product list from Exercise 1.

**1. Define the following as named variables (not inline lambdas) before using them:**

- A `Predicate<Product>` that is `true` if the product has stock available.
- A `Predicate<Product>` that is `true` if the product price is above `50.0`.
- A `Function<Product, String>` that formats a product as `"<name> (<category>) - <price> kr"`.
- A `Consumer<Product>` that prints the formatted product string to the console.

**2. Use predicate composition, `map()`, and `forEach()` to:**

1. Print all products that are in stock **and** cost more than 50 kr — using the named predicate, function, and consumer above.
2. Compose the two predicates with `or()` to print all products that are either in stock or cost more than 50 kr.
3. Use `Predicate.not()` to print all products that are **out of stock**.

**Hint:** Wire them together using the stream pattern — the hint below shows exactly how the pieces connect:
```java
products.stream()          // iterate over the list
    .filter(inStock.and(expensive))  // keep only matching products (uses your Predicate)
    .map(format)           // transform each Product to a String (uses your Function)
    .forEach(print);       // print each String (uses your Consumer)
```

<!-- **SOLUTIONS**

### Exercise 1
```java
Validator<Product> isElectronics = p -> "Electronics".equals(p.category());
Validator<Product> isAffordable  = p -> p.price() < 100.0;
Validator<Product> isInStock     = p -> p.stock() > 0;

// 1. All electronics
System.out.println(products.stream().filter(isElectronics::validate).toList());
// [Laptop, Headphones, Smartwatch]

// 2. Affordable products (price < 100)
System.out.println(products.stream().filter(isAffordable::validate).toList());
// [Headphones, Java Book, T-shirt, Smartwatch]

// 3. In-stock products
System.out.println(products.stream().filter(isInStock::validate).toList());
// [Laptop, Java Book, T-shirt, Smartwatch]

// 4. Electronics in stock
Validator<Product> electronicsInStock = isElectronics.and(isInStock);
System.out.println(products.stream().filter(electronicsInStock::validate).toList());
// [Laptop, Smartwatch]

// 5. Expensive products (price >= 100)
Validator<Product> isExpensive = isAffordable.negate();
System.out.println(products.stream().filter(isExpensive::validate).toList());
// [Laptop]

// 6. Books or Clothing
Validator<Product> isBooks    = p -> "Books".equals(p.category());
Validator<Product> isClothing = p -> "Clothing".equals(p.category());
Validator<Product> isBooksOrClothing = isBooks.or(isClothing);
System.out.println(products.stream().filter(isBooksOrClothing::validate).toList());
// [Java Book, T-shirt]
```

### Exercise 2
```java
Predicate<Product> inStock   = p -> p.stock() > 0;
Predicate<Product> expensive = p -> p.price() > 50.0;

Function<Product, String> format =
    p -> String.format("%s (%s) - %.0f kr", p.name(), p.category(), p.price());

Consumer<Product> print = p -> System.out.println(format.apply(p));

// 1. In stock AND expensive
products.stream()
    .filter(inStock.and(expensive))
    .forEach(print);

// 2. In stock OR expensive
products.stream()
    .filter(inStock.or(expensive))
    .forEach(print);

// 3. Out of stock
products.stream()
    .filter(Predicate.not(inStock))
    .forEach(print);
```
-->
