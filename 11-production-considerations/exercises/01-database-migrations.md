# Flyway Migrations with Spring Boot

## Table of Contents

1. [Why Flyway?](#1-why-flyway)
2. [Project Setup](#2-project-setup)
3. [IntelliJ Plugin Setup](#3-intellij-plugin-setup)
4. [V1 – Create the `orders` table](#4-v1--create-the-orders-table)
5. [V2 – Create the `order_item` table](#5-v2--create-the-order_item-table)
6. [V3 – Add the foreign key](#6-v3--add-the-foreign-key)
7. [V4 – Seed data (the stakes are real now)](#7-v4--seed-data-the-stakes-are-real-now)
8. [Pitfalls](#8-pitfalls)
9. [Hands-on Exercise – Adding `price` the right way](#9-hands-on-exercise--adding-price-the-right-way)
10. [Best Practices](#10-best-practices)
11. [Recovery Walkthrough – Escaping a Dirty State](#11-recovery-walkthrough--escaping-a-dirty-state)

---

## 1. Why Flyway?

### The development shortcut that kills production

When you first learn Spring Data JPA, you probably set this in `application.properties`:

```properties
spring.jpa.hibernate.ddl-auto=create-drop
```

This tells Hibernate to **drop and recreate the entire database schema every time the application starts**. It is convenient during development — you change an entity field and the table just updates. No SQL needed.

But imagine you are running this in production with real customer data. A developer restarts the service. The database is wiped. Every row. Gone.

Even the less nuclear `update` mode is dangerous in production: Hibernate will attempt to add columns but **will never drop them, rename them, or migrate existing data**. The result is a schema that slowly drifts from what your code expects.

### The professional alternative

Flyway solves this by treating your database schema exactly like source code: **versioned, reviewed, and applied incrementally**. Instead of letting Hibernate guess what the database should look like, you write explicit SQL migration scripts. Flyway tracks which scripts have already been applied in a table called `flyway_schema_history` and only runs new ones.

| | `create-drop` / `update` | Flyway |
|---|---|---|
| Who controls the schema? | Hibernate (guesses from entities) | You (explicit SQL) |
| What happens to existing data? | Potentially wiped or broken | Preserved and migrated |
| Audit trail? | None | `flyway_schema_history` table |
| Safe in production? | ❌ No | ✅ Yes |

### What this tutorial covers

You will build an `Order` / `OrderItem` domain from scratch, creating each migration file step by step. Real data will be seeded midway, so when you later need to add a new column you will face the same backfill challenge you would in production.

---

## 2. Project Setup

### Dependencies

The project already has everything it needs in `pom.xml`:

```xml
<!-- Flyway core Spring Boot integration -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-flyway</artifactId>
</dependency>

<!-- MySQL-specific Flyway dialect -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

Flyway runs **automatically on application startup** via Spring Boot's auto-configuration. There is nothing extra to install on your machine — no CLI, no separate tool.

### application.properties

```properties
spring.jpa.hibernate.ddl-auto=validate
```

This is the key setting. `validate` tells Hibernate to **compare your JPA entities against the actual database schema on startup** and throw an error if they do not match. It will never touch the schema itself. Flyway is responsible for the schema — Hibernate just checks that everything lines up. This gives you a safety net: if you forget to write a migration for a new entity field, the application will refuse to start.

### Database

The project uses Docker Compose to run MySQL locally. Start it with:

```bash
docker compose up -d
```

The `spring.docker.compose.lifecycle-management=start_only` property in `application.properties` means Spring Boot will start the compose services automatically when the app boots, but will not stop them when it shuts down.

### Migration file location

Flyway automatically scans `src/main/resources/db/migration/` for SQL files. This folder currently exists but is **empty** — that is your starting point.

### Naming convention

Flyway migration filenames follow a strict pattern:

```
V{version}__{description}.sql
  ^         ^^
  version   two underscores
```

Examples: `V1__create_orders.sql`, `V2__create_order_item.sql`

- The version can be any number or dot-separated number (`V1`, `V2`, `V4.1`, `V4.2`)
- The description after the double underscore is free text (use underscores for spaces)
- Versions are applied in ascending order
- **Once applied, a file must never be changed** (more on this in [Pitfalls](#8-pitfalls))

---

## 3. IntelliJ Plugin Setup

Two plugins will make your Flyway workflow significantly smoother.

### JPA Buddy

**JPA Buddy** can inspect your JPA entities and the live database schema, diff them, and generate the migration SQL for you automatically.

1. Open `Settings → Plugins → Marketplace`
2. Search for **JPA Buddy** and install it
3. Restart IntelliJ

Once installed, when you add a field to a JPA entity you will see a **JPA Buddy** panel in the bottom toolbar. You can right-click an entity → `Flyway Versioned Migration` → it generates the correct `ALTER TABLE` SQL based on what has changed.

> **When to use it:** JPA Buddy is great for generating the initial DDL for new tables and straightforward `ALTER TABLE` additions. For multi-step migrations (like the expand-contract pattern you will learn in [section 9](#9-hands-on-exercise--adding-price-the-right-way)), you still write the SQL yourself because JPA Buddy only sees the entity state, not the intermediate steps.

### Flyway Migration Creation

This plugin adds syntax highlighting, version validation, and quick navigation for Flyway SQL files.

1. Open `Settings → Plugins → Marketplace`
2. Search for **Flyway Migration Creation** and install it
3. Restart IntelliJ

Once installed, migration files in `db/migration/` get a Flyway icon and the plugin will warn you if two files share the same version number.

---

## 4. V1 – Create the `orders` table

### Write the migration

Create the file `src/main/resources/db/migration/V1__create_orders.sql`:

```sql
CREATE TABLE orders
(
    id          BIGINT AUTO_INCREMENT NOT NULL,
    description VARCHAR(255)          NULL,
    CONSTRAINT pk_orders PRIMARY KEY (id)
);
```

### Write the entity

Create `src/main/java/com/example/demo/order/Order.java`:

```java
package com.example.demo.order;

import com.example.demo.orderitem.OrderItem;
import jakarta.persistence.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }

    public List<OrderItem> getItems() { return items; }
    public void setItems(List<OrderItem> items) { this.items = items; }
}
```

> **Note:** The `@OneToMany` references `OrderItem` which does not exist yet. This will cause a compile error until you complete V2. Alternatively, add the `@OneToMany` relationship after V2.

### Write the repository

Create `src/main/java/com/example/demo/order/OrderRepository.java`:

```java
package com.example.demo.order;

import org.springframework.data.jpa.repository.JpaRepository;

public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

### Boot the application

Run the application. In the startup logs you will see Flyway execute the migration:

```
Flyway Community Edition ... by Redgate
Database: jdbc:mysql://localhost:3311/mydatabase (MySQL ...)
Successfully validated 1 migration (execution time 00:00.015s)
Current version of schema `mydatabase`: << Empty Schema >>
Migrating schema `mydatabase` to version "1 - create orders"
Successfully applied 1 migration to schema `mydatabase`
```

Connect to MySQL and verify:

```sql
SELECT * FROM flyway_schema_history;
```

You will see one row with `version = 1`, `success = 1`, and a checksum of the file contents.

---

## 5. V2 – Create the `order_item` table

### Write the migration

Create `src/main/resources/db/migration/V2__create_order_item.sql`:

```sql
CREATE TABLE order_item
(
    id   BIGINT AUTO_INCREMENT NOT NULL,
    name VARCHAR(255)          NULL,
    CONSTRAINT pk_order_item PRIMARY KEY (id)
);
```

Notice there is **no `order_id` foreign key yet**. We add the relationship in V3. This mirrors a real workflow: sometimes you create a table first and link it to existing tables in a follow-up migration, especially when multiple teams are working in parallel.

### Write the entity

Create `src/main/java/com/example/demo/orderitem/OrderItem.java`:

```java
package com.example.demo.orderitem;

import com.example.demo.order.Order;
import jakarta.persistence.*;

@Entity
@Table(name = "order_item")
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public Order getOrder() { return order; }
    public void setOrder(Order order) { this.order = order; }
}
```

> **Note:** `OrderItem` already has `order_id` mapped via `@JoinColumn`. The column does not exist in the DB yet — that is fine because `ddl-auto=validate` will only check columns that are mapped to **non-nullable** constraints. Since `order_id` will be `NULL`able, Hibernate's validation will pass once V3 runs.

### Boot the application

Flyway will apply only `V2` — it skips `V1` because it has already been applied (the checksum matches).

---

## 6. V3 – Add the foreign key

### Write the migration

Create `src/main/resources/db/migration/V3__add_order_item_fk.sql`:

```sql
ALTER TABLE order_item
    ADD order_id BIGINT NULL;

ALTER TABLE order_item
    ADD CONSTRAINT fk_order_item_on_order FOREIGN KEY (order_id) REFERENCES orders (id);
```

We add `order_id` as `NULL` because there is no data yet. If there were existing rows in `order_item`, adding a `NOT NULL` column without a default would fail. We will revisit this exact scenario in the exercise.

### Boot the application

Two migrations have now been applied (`V1`, `V2`). On this boot, only `V3` runs.

### Add the REST controller

Now that the full domain is wired up, add the controller and DTOs.

Create `src/main/java/com/example/demo/order/OrderDto.java`:

```java
package com.example.demo.order;

import java.util.List;

public record OrderDto(
        Long id,
        String description,
        List<OrderItemDto> items
) {
    public record OrderItemDto(
            Long id,
            String name
    ) {}
}
```

Create `src/main/java/com/example/demo/order/OrderService.java`:

```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }


    @Transactional(readOnly = true)
    public List<OrderDto> getAllOrders() {
        List<Order> orders = orderRepository.findAll();
        List<OrderDto> orderDtos = new ArrayList<>();
        for (Order order : orders) {
            orderDtos.add(toDto(order));
        }
        return orderDtos;
    }

    private OrderDto toDto(Order order) {
        List<OrderDto.OrderItemDto> itemDtos = new ArrayList<>();
        for (OrderItem item : order.getItems()) {
            itemDtos.add(toDto(item));
        }
        return new OrderDto(order.getId(), order.getDescription(), itemDtos);
    }

    private OrderDto.OrderItemDto toDto(OrderItem item) {
        return new OrderDto.OrderItemDto(
                item.getId(),
                item.getName(),
                item.getPrice()
        );
    }
}
```
> **Why `@Transactional(readOnly = true)`?**  
> `Order.items` is mapped with `FetchType.LAZY` — the items list is only loaded from the database when it is first accessed. This access happens inside the `.map()` call in the controller. For lazy loading to work, the JPA session must still be open at that point. `@Transactional` keeps the session open for the duration of the method. Without it you would get a `LazyInitializationException`.

Create `src/main/java/com/example/demo/order/OrderController.java`:

```java
package com.example.demo.order;

import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/orders")
public class OrderController {


    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping
    public List<OrderDto> getAllOrders() {
        return orderService.getAllOrders();
    }
}
```

Boot the app and hit `GET http://localhost:8080/orders`. You will get an empty array `[]` — no data yet.

---

## 7. V4 – Seed data (the stakes are real now)

### Write the migration

Create `src/main/resources/db/migration/V4__seed_data.sql`:

```sql
INSERT INTO orders (description)
VALUES ('Order 1'),
       ('Order 2'),
       ('Order 3');

INSERT INTO order_item (name, order_id)
VALUES ('Laptop', 1),
       ('Mouse', 1),
       ('Keyboard', 2),
       ('Monitor', 2),
       ('Webcam', 3);
```

### Boot and verify

Boot the app and hit `GET http://localhost:8080/orders`. You should now see:

```json
[
  {
    "id": 1,
    "description": "Order 1",
    "items": [
      { "id": 1, "name": "Laptop" },
      { "id": 2, "name": "Mouse" }
    ]
  },
  {
    "id": 2,
    "description": "Order 2",
    "items": [
      { "id": 3, "name": "Keyboard" },
      { "id": 4, "name": "Monitor" }
    ]
  },
  {
    "id": 3,
    "description": "Order 3",
    "items": [
      { "id": 5, "name": "Webcam" }
    ]
  }
]
```

**These rows now live in your database.** From this point on, any schema change you make must account for them. This is the reality of production: you cannot pretend the data does not exist.

---

## 8. Pitfalls

> ⚠️ **Read this section carefully — do not follow these steps.** The examples below demonstrate what goes wrong so you know what to avoid and how to recognise the errors.

### Pitfall 1: Editing an already-applied migration

Suppose you open `V4__seed_data.sql` and add another `INSERT` row, then restart the app. Flyway recomputes the file's checksum and compares it against what was stored in `flyway_schema_history` when the migration was first applied. They will not match.

**The error you will see:**

```
Migration checksum mismatch for migration version 4
-> Applied to database : 112233445
-> Resolved locally    : 998877665
```

The application refuses to start. This is intentional — Flyway is protecting you from silent, untracked changes to the schema history. **The fix is to never edit an applied file.** If you need additional data, create a new migration (`V5`, etc.).

### Pitfall 2: Adding a NOT NULL column in one step with existing data

Imagine you want to add a `price` column directly as `NOT NULL` with no default:

```sql
-- ❌ Do NOT do this when rows already exist
ALTER TABLE order_item ADD price DECIMAL(10,2) NOT NULL;
```

MySQL will reject this statement because the 5 existing `order_item` rows would have no value for `price`. The app will fail to start and the migration will be marked as failed in `flyway_schema_history`.

**The error you will see (from MySQL):**

```
Invalid default value for 'price' /
Column 'price' cannot be null
```

This leaves your database in an inconsistent state — see [Pitfall 3](#pitfall-3-mysql-and-dirty-states) below.

The correct approach — which you will practice in the exercise — is to:
1. Add the column as `NULL` first
2. Backfill all existing rows with a value
3. Then tighten it to `NOT NULL`

### Pitfall 3: MySQL and dirty states (non-transactional DDL)

MySQL **does not support transactional DDL**. In PostgreSQL, if a migration script fails halfway through, the entire script is rolled back automatically. In MySQL, every DDL statement (`CREATE TABLE`, `ALTER TABLE`, etc.) is committed immediately and **cannot be rolled back**.

This means a migration file that contains:

```sql
ALTER TABLE order_item ADD price DECIMAL(10,2) NOT NULL; -- ❌ fails here
INSERT INTO order_item (name, price, order_id) VALUES ('Extra item', 99.99, 1); -- never runs
```

will leave the database **partially changed** — the `ALTER TABLE` may or may not have been applied, but Flyway marks the migration as `failed` (success = 0) in `flyway_schema_history`. On the next boot, Flyway will see a failed migration and refuse to run any further migrations until the situation is manually resolved.

**How to recover** is covered in [section 11](#11-recovery-walkthrough--escaping-a-dirty-state).

### Pitfall 4: Renaming a column destructively in one step

Suppose you want to rename `order_item.name` to `order_item.product_name`. The naive approach:

```sql
-- ❌ Destroys all existing data in the column
ALTER TABLE order_item RENAME COLUMN name TO product_name;
```

If you are doing a zero-downtime deployment (old and new versions of the app running simultaneously), the old version of the code still reads `name`. The moment this migration runs, the old version crashes.

The safe approach is **expand-contract**:
1. `V5` – Add `product_name VARCHAR(255) NULL` (both columns exist, old code still works)
2. `V6` – `UPDATE order_item SET product_name = name` (backfill)
3. `V7` – Deploy the new code that reads `product_name`
4. `V8` – Drop the old `name` column (only safe once no code references it)

---

## 9. Hands-on Exercise – Adding `price` the right way

The product manager wants every `order_item` to have a `price`. There are already 5 items in the database with no price. You need to add the column without losing data and without breaking the running application.

Follow the expand-contract pattern across three migrations.

### Step 1 — V5: Add the column as NULL

Create `src/main/resources/db/migration/V5__add_price_to_order_item.sql`:

```sql
ALTER TABLE order_item
    ADD price DECIMAL(10, 2) NULL;
```

Boot the app. The migration runs. The 5 existing rows now have `price = NULL`. The application still starts because the column is nullable — Hibernate's `ddl-auto=validate` accepts it.

Hit `GET http://localhost:8080/orders` — items still appear, price is not exposed yet (the DTO does not have the field). **This is intentional.** The DB is being prepared before the code changes.

### Step 2 — V6: Backfill existing rows

Create `src/main/resources/db/migration/V6__backfill_price.sql`:

```sql
UPDATE order_item
SET price = ROUND((RAND() * 490) + 10, 2)
WHERE price IS NULL;
```

`RAND()` returns a float between 0 and 1. Multiplying by 490 and adding 10 gives a random price between 10.00 and 500.00. `ROUND(..., 2)` trims to 2 decimal places.

Boot the app. Connect to MySQL and verify:

```sql
SELECT id, name, price FROM order_item;
```

All 5 rows should now have a non-null price. Every time this migration runs on a fresh database it will produce different random values — which is fine for development seed data.

> **In production you would not use `RAND()`** — you would use a meaningful business default (e.g., copy from a pricing table, use a known default rate, or flag rows for manual review). The point is: you must set a value before you can enforce `NOT NULL`.

### Step 3 — V7: Tighten the constraint

Now that every row has a value, you can safely enforce `NOT NULL`.

Create `src/main/resources/db/migration/V7__make_price_not_null.sql`:

```sql
ALTER TABLE order_item
    MODIFY price DECIMAL(10, 2) NOT NULL;
```

Boot the app. This `ALTER TABLE` will succeed because there are no `NULL` values left. Hibernate's `ddl-auto=validate` will now expect a non-nullable column in the schema.

### Step 4 — Update the entity

Add the `price` field to `OrderItem.java`:

```java
import java.math.BigDecimal;

// inside OrderItem class:
private BigDecimal price;

public BigDecimal getPrice() { return price; }
public void setPrice(BigDecimal price) { this.price = price; }
```

The full updated entity:

```java
package com.example.demo.orderitem;

import com.example.demo.order.Order;
import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name = "order_item")
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private BigDecimal price;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }

    public Order getOrder() { return order; }
    public void setOrder(Order order) { this.order = order; }
}
```

### Step 5 — Update the DTO and controller

Update `OrderDto.java` to include `price` in the nested record:

```java
package com.example.demo.order;

import java.math.BigDecimal;
import java.util.List;

public record OrderDto(
        Long id,
        String description,
        List<OrderItemDto> items
) {
    public record OrderItemDto(
            Long id,
            String name,
            BigDecimal price      // ← add this
    ) {}
}
```

Update `OrderController.java` to pass `price` into the DTO:

```java
order.getItems().stream()
        .map(item -> new OrderDto.OrderItemDto(
                item.getId(),
                item.getName(),
                item.getPrice()   // ← add this
        ))
        .toList()
```

### Step 6 — Verify

Boot the app. `ddl-auto=validate` checks that the entity matches the schema — it should pass cleanly.

Hit `GET http://localhost:8080/orders`. You will see items with random prices:

```json
[
  {
    "id": 1,
    "description": "Order 1",
    "items": [
      { "id": 1, "name": "Laptop",   "price": 342.17 },
      { "id": 2, "name": "Mouse",    "price": 89.55  }
    ]
  },
  ...
]
```

You have successfully added a non-nullable column to a table with existing data — without data loss, without a failed migration, and without downtime.

---

## 10. Best Practices

### ✅ Never edit an applied migration file

Once a migration has been applied to any environment (local, staging, production), treat it as **immutable**. Flyway's checksum mechanism will immediately detect any change and halt the application. If you need to fix something, create a new migration.

### ✅ Use descriptive file names

Bad: `V5__changes.sql`  
Good: `V5__add_price_to_order_item.sql`

The description is purely for humans — Flyway only uses the version number for ordering. Good names make the `flyway_schema_history` table readable as an audit log.

### ✅ One logical change per file

Resist the temptation to bundle many `ALTER TABLE` statements into one file. Small, focused migrations are easier to review, easier to understand in the history table, and easier to recover from if something goes wrong.

### ✅ Keep DDL and DML separate

Do not mix `ALTER TABLE` (DDL) and `INSERT`/`UPDATE` (DML) in the same migration file. MySQL commits DDL immediately — if a later DML statement in the same file fails, the DDL cannot be rolled back.

```sql
-- ❌ Dangerous: mix of DDL and DML in one file
ALTER TABLE order_item ADD price DECIMAL(10,2) NULL;
UPDATE order_item SET price = 99.99;

-- ✅ Safe: two separate files
-- V5__add_price_column.sql  → only the ALTER TABLE
-- V6__backfill_price.sql    → only the UPDATE
```

### ✅ Use ddl-auto=validate in production

```properties
spring.jpa.hibernate.ddl-auto=validate
```

This is already set in this project's `application.properties`. It means the app will refuse to start if the database schema does not match the JPA entities. This acts as an automatic sanity check after every deployment.

### ✅ Use the expand-contract pattern for zero-downtime

When renaming or removing a column, never do it in a single destructive migration. Instead:

1. **Expand** – Add the new column alongside the old one
2. **Migrate** – Backfill data from old to new
3. **Contract** – Deploy new code that uses the new column, then drop the old one in a subsequent release

This allows old and new versions of your application to coexist during a rolling deployment.

### ✅ Use flyway_schema_history as an audit log

The `flyway_schema_history` table is automatically created and maintained by Flyway. It records every migration that has been applied, when it ran, how long it took, and whether it succeeded. In production, this table is your primary tool for answering "what state is this database in?".

```sql
SELECT version, description, installed_on, execution_time, success
FROM flyway_schema_history
ORDER BY installed_rank;
```

### ✅ Use flyway:repair only as a last resort

`mvn flyway:repair` removes failed migration entries from `flyway_schema_history`. It does **not** undo the partial SQL that was applied — you must do that manually first. Use it only after you have manually cleaned up the database. See [section 11](#11-recovery-walkthrough--escaping-a-dirty-state) for the full procedure.

Note that `flyway:repair` doesn't pick up Spring Boots `application.properties`, so it needs its own database configuration:
```bash
./mvnw flyway:repair \
  -Dflyway.url=jdbc:mysql://localhost:3311/mydatabase \
  -Dflyway.user=myuser \
  -Dflyway.password=secret
```

---

## 11. Recovery Walkthrough – Escaping a Dirty State

This section describes what to do if a migration has partially applied and Flyway is stuck.

### How you end up in a dirty state

You have a migration file that fails partway through. Because MySQL does not support transactional DDL, some statements in the file have already been committed to the database. Flyway marks the migration as `success = 0` in `flyway_schema_history`.

On the next startup, Flyway sees the failed entry and refuses to continue:

```
Found failed migration to version 5 (add price to order item).
Please restore backups and roll back database and code!
Alternatively, you may force validation to be skipped by setting:
  flyway.validateOnMigrate=false
```

**Do not set `validateOnMigrate=false`.** That just hides the problem. Fix it properly.

### Step-by-step recovery

#### 1. Identify what was and was not applied

Connect to MySQL and inspect the history table:

```sql
SELECT version, description, script, success, installed_on
FROM flyway_schema_history
ORDER BY installed_rank;
```

Find the row with `success = 0`. Note the version number — for example `5`.

#### 2. Inspect the failed migration file

Open the migration file that failed. Identify which statements ran successfully before the failure and which did not. For example, if the file was:

```sql
ALTER TABLE order_item ADD price DECIMAL(10,2) NOT NULL;  -- ← this committed and failed (MySQL error)
INSERT INTO order_item (name, price, order_id) VALUES ('Extra', 9.99, 1); -- ← never ran
```

The `ALTER TABLE` may have partially succeeded or not at all, depending on when MySQL raised the error.

#### 3. Manually undo the partial changes

Write compensating SQL to return the database to the state it was in before the migration started. For the example above:

```sql
-- Undo the column addition if it was applied
ALTER TABLE order_item DROP COLUMN price;
```

Run these statements directly against the database via a MySQL client. **Do not put them in a new migration file yet** — you need to clean up first, then re-attempt with a corrected migration.

#### 4. Remove the failed entry with flyway:repair

Now that the database is back to its pre-migration state, remove the failed entry from `flyway_schema_history`:

```bash
./mvnw flyway:repair \
  -Dflyway.url=jdbc:mysql://localhost:3311/mydatabase \
  -Dflyway.user=myuser \
  -Dflyway.password=secret
```
> Replace with the actual database credentials

This removes all `success = 0` rows from the history table. It does **not** touch your data or schema.

#### 5. Fix the migration and retry

Correct the migration file (or create a new one if the file has already been applied on other environments). In our example, the fix is to split the migration as described in the exercise:

- `V5__add_price_to_order_item.sql` → `ADD price DECIMAL(10,2) NULL`
- `V6__backfill_price.sql` → `UPDATE ... SET price = ...`
- `V7__make_price_not_null.sql` → `MODIFY price ... NOT NULL`

Start the application. Flyway will apply the corrected migration cleanly.

### Key takeaway

The root cause of every dirty state is a migration that tried to do too much at once. The best way to avoid recovery scenarios is to follow the expand-contract pattern and keep DDL and DML in separate files from the start.

---

*This tutorial was written to accompany a Spring Boot 4 + MySQL + Flyway demo project. All migration files go in `src/main/resources/db/migration/`.*

