# Cinema Pricing Engine — Design Architecture Tutorial

> **Learning goals:** SOLID principles · High cohesion & low coupling · Chain of Responsibility pattern · Logging strategy · Test-Driven Development (TDD)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Business Rules](#2-business-rules)
3. [Step 1 — The Naive Implementation](#3-step-1--the-naive-implementation)
4. [Step 2 — Extract `TicketPriceRule`](#4-step-2--extract-ticketpricerule)
5. [Step 3 — The Accumulator: `TicketPriceResult`](#5-step-3--the-accumulator-ticketpriceresult)
6. [Step 4 — Remaining Ticket Rules](#6-step-4--remaining-ticket-rules)
7. [Step 5 — The Ticket Engine](#7-step-5--the-ticket-engine)
8. [Step 6 — Order-Level Rules](#8-step-6--order-level-rules)
9. [Step 7 — Architecture Diagram](#9-step-7--architecture-diagram)
10. [Step 8 — Logging](#10-step-8--logging)
11. [Step 9 — TDD](#11-step-9--tdd)
12. [Step 10 — Exercises](#12-step-10--exercises)

---

## 1. Introduction

You are building a **cinema ticket pricing engine** for a movie theatre. The engine must calculate
the price of an order, where each order contains one or more tickets. Each ticket has a seat type,
and all tickets in the same order share a showing context (movie format, start time, duration).

The engine must be:

- **Easy to extend** — new rules should require only a new class, never a change to existing code
- **Traceable** — it must be possible to see exactly which rules fired and what they contributed
- **Testable** — each rule must be testable in isolation

### Package structure

```
priceengine/
├── package-info.java
├── orderRules/
│   ├── PriceEngine.java               ← interface: calculates full order price
│   ├── DefaultPriceEngine.java        ← implementation
│   ├── OrderPriceRule.java            ← interface: one order-level rule
│   ├── OrderReservationFeeRule.java   ← concrete rule
│   ├── BulkBookingDiscountRule.java   ← concrete rule
│   └── OrderRulesConfiguration.java  ← Spring wiring
├── ticketRules/
│   ├── TicketPriceEngine.java         ← interface: calculates one ticket price
│   ├── DefaultTicketPriceEngine.java  ← implementation
│   ├── TicketPriceRule.java           ← interface: one ticket-level rule
│   ├── TicketBasePriceRule.java       ← concrete rule
│   ├── MovieFormatPriceRule.java      ← concrete rule
│   ├── MovieDurationPriceRule.java    ← concrete rule
│   ├── SeatTypePriceRule.java         ← concrete rule
│   └── TicketRulesConfiguration.java ← Spring wiring
├── request/
│   ├── OrderPriceRequest.java         ← input: list of seat types + showing context
│   ├── TicketPriceRequest.java        ← input: one seat type + showing context
│   ├── ShowingContext.java            ← movie format, start time, duration
│   ├── SeatType.java                  ← enum: COWBOY, REGULAR, SOFA
│   └── MovieFormat.java               ← enum: TWO_D, THREE_D, IMAX
└── response/
    ├── OrderPriceResult.java          ← output: list of ticket results + order adjustments
    ├── TicketPriceResult.java         ← output: base price + ticket adjustments
    ├── OrderAdjustment.java           ← one order-level adjustment
    ├── TicketAdjustment.java          ← one ticket-level adjustment
    └── AdjustmentType.java            ← enum: SURCHARGE, DISCOUNT
```

---

## 2. Business Rules

### Ticket-level rules (applied per seat)

| Rule | Condition | Effect |
|---|---|---|
| Base price | Always | +100.00 |
| Movie format | `THREE_D` | +20.00 surcharge |
| Movie format | `IMAX` | +30.00 surcharge |
| Movie format | `TWO_D` | no adjustment |
| Movie duration | > 170 minutes | +15.00 surcharge |
| Seat type | `REGULAR` | +5.00 surcharge |
| Seat type | `SOFA` | +10.00 surcharge |
| Seat type | `COWBOY` | no adjustment |

### Order-level rules (applied to the whole order)

| Rule | Condition | Effect |
|---|---|---|
| Reservation fee | ≤ 5 tickets | +7% of ticket total |
| Bulk discount | > 10 tickets | −5% of ticket base total |

---

## 3. Step 1 — The Naive Implementation

Below is a naive implementation that calculates the order price in a single class.
**Read it carefully and identify the problems before reading the critique below.**

```java
public class NaivePriceCalculator {

    public BigDecimal calculate(List<String> seatTypes, int durationMinutes,
                                String movieFormat, int numberOfTickets) {
        BigDecimal total = BigDecimal.ZERO;

        for (String seat : seatTypes) {
            BigDecimal price = new BigDecimal("100.00");

            if (seat.equals("REGULAR")) {
                price = price.add(new BigDecimal("5.00"));
            } else if (seat.equals("SOFA")) {
                price = price.add(new BigDecimal("10.00"));
            }

            if (movieFormat.equals("THREE_D")) {
                price = price.add(new BigDecimal("20.00"));
            } else if (movieFormat.equals("IMAX")) {
                price = price.add(new BigDecimal("30.00"));
            }

            if (durationMinutes > 170) {
                price = price.add(new BigDecimal("15.00"));
            }

            total = total.add(price);
        }

        if (numberOfTickets <= 5) {
            total = total.add(total.multiply(new BigDecimal("0.07")));
        }

        if (numberOfTickets > 10) {
            total = total.subtract(total.multiply(new BigDecimal("0.05")));
        }

        return total;
    }
}
```

### What is wrong?

| Problem | Principle violated | Consequence |
|---|---|---|
| All rules live in one method | **SRP** — Single Responsibility | One reason to change becomes many |
| Adding a rule means editing this class | **OCP** — Open/Closed | Risk of breaking existing rules |
| `"REGULAR"`, `"THREE_D"`, `100.00`, `0.07` are magic strings/numbers | **DRY** | Change the value in one place, forget it in another |
| No breakdown returned — only a total | **Cohesion** | Impossible to show the customer what they are paying for |
| Cannot test one rule without triggering all others | **Testability** | A bug in one rule breaks tests for all rules |
| No logging | **Observability** | Impossible to trace why a price was calculated incorrectly in production |
| Strings instead of enums for `seatType` and `movieFormat` | **Type safety** | A typo compiles, but produces wrong prices silently |

---

## 4. Step 2 — Extract `TicketPriceRule`

The first step is to give each rule its own class. To do that, we need a shared contract — an interface.

```java
public interface TicketPriceRule {
    void apply(TicketPriceRequest request, TicketPriceResult result);
}
```

- `TicketPriceRequest` carries the input for one ticket (seat type + showing context)
- `TicketPriceResult` is the accumulator — rules write their contribution into it

The simplest rule is the base price — it has no condition, it always fires first:

```java
class TicketBasePriceRule implements TicketPriceRule {
    private final BigDecimal BASE_PRICE = new BigDecimal("100.00");

    @Override
    public void apply(TicketPriceRequest request, TicketPriceResult result) {
        result.setBasePrice(BASE_PRICE);
    }
}
```

> **SRP** — `TicketBasePriceRule` has exactly one reason to change: the base price changes.
>
> **OCP** — adding a new rule means writing a new class. `TicketBasePriceRule` is never touched again.

---

## 5. Step 3 — The Accumulator: `TicketPriceResult`

Each rule calls either `setBasePrice` (only `TicketBasePriceRule`) or `addAdjustment` (all other rules).
`TicketPriceResult` owns the logic for computing the final total — no one else does.

```java
public class TicketPriceResult {
    private TicketPriceRequest ticketPriceRequest;
    private List<TicketAdjustment> adjustments;
    private BigDecimal basePrice;

    private TicketPriceResult() {}

    public static TicketPriceResult of(TicketPriceRequest request) {
        TicketPriceResult r = new TicketPriceResult();
        r.ticketPriceRequest = request;
        r.basePrice = BigDecimal.ZERO;
        r.adjustments = new ArrayList<>();
        return r;
    }

    public void setBasePrice(BigDecimal amount) { this.basePrice = amount; }

    public void addAdjustment(AdjustmentType type, BigDecimal amount) {
        this.adjustments.add(new TicketAdjustment(type, amount));
    }

    public BigDecimal getTotal() {
        BigDecimal total = basePrice;
        for (TicketAdjustment adj : adjustments) {
            total = total.add(adj.getAmount()); // getAmount() negates DISCOUNT automatically
        }
        return total;
    }
}
```

`AdjustmentType` distinguishes surcharges from discounts:

```java
public enum AdjustmentType {
    SURCHARGE,
    DISCOUNT
}
```

`TicketAdjustment` automatically negates discounts so callers always use `BigDecimal::add`:

```java
public record TicketAdjustment(AdjustmentType type, BigDecimal amount) {
    public BigDecimal getAmount() {
        return type == AdjustmentType.DISCOUNT ? amount.negate() : amount;
    }
}
```

> **High cohesion** — `TicketPriceResult` owns everything about a ticket's price. The calculation
> logic for `getTotal()` lives here and nowhere else.

---

## 6. Step 4 — Remaining Ticket Rules

### `MovieFormatPriceRule`

```java
class MovieFormatPriceRule implements TicketPriceRule {
    private final BigDecimal THREE_D_SURCHARGE = new BigDecimal("20.00");
    private final BigDecimal IMAX_SURCHARGE    = new BigDecimal("30.00");

    @Override
    public void apply(TicketPriceRequest request, TicketPriceResult result) {
        BigDecimal surcharge = switch (request.showingContext().movieFormat()) {
            case TWO_D   -> BigDecimal.ZERO;
            case THREE_D -> THREE_D_SURCHARGE;
            case IMAX    -> IMAX_SURCHARGE;
        };
        if (surcharge.compareTo(BigDecimal.ZERO) > 0) {
            result.addAdjustment(AdjustmentType.SURCHARGE, surcharge);
        }
    }
}
```

### `MovieDurationPriceRule`

```java
class MovieDurationPriceRule implements TicketPriceRule {
    private final BigDecimal LONG_MOVIE_SURCHARGE = new BigDecimal("15.00");

    @Override
    public void apply(TicketPriceRequest request, TicketPriceResult result) {
        if (request.showingContext().duration() > 170) {
            result.addAdjustment(AdjustmentType.SURCHARGE, LONG_MOVIE_SURCHARGE);
        }
    }
}
```

### `SeatTypePriceRule`

```java
class SeatTypePriceRule implements TicketPriceRule {
    private final BigDecimal REGULAR_SEAT_SURCHARGE = new BigDecimal("5.00");
    private final BigDecimal VIP_SEAT_SURCHARGE     = new BigDecimal("10.00");

    @Override
    public void apply(TicketPriceRequest request, TicketPriceResult result) {
        BigDecimal surcharge = switch (request.seatType()) {
            case COWBOY  -> BigDecimal.ZERO;
            case REGULAR -> REGULAR_SEAT_SURCHARGE;
            case SOFA    -> VIP_SEAT_SURCHARGE;
        };
        if (surcharge.compareTo(BigDecimal.ZERO) > 0) {
            result.addAdjustment(AdjustmentType.SURCHARGE, surcharge);
        }
    }
}
```

### Why switch expressions matter here

The switch above is a **switch expression** — it must return a value, so the compiler enforces
exhaustiveness. If you add a new `SeatType` (e.g. `PREMIUM`) and forget to handle it here, the
code **will not compile**. This is a compile-time safety net you get for free.

Compare with a switch statement:

```java
// This compiles even if PREMIUM is missing — silent wrong prices
switch (request.seatType()) {
    case COWBOY  -> { /* nothing */ }
    case REGULAR -> result.addAdjustment(AdjustmentType.SURCHARGE, REGULAR_SEAT_SURCHARGE);
}
```

> **Prefer switch expressions over switch statements** in rules that handle enums.

---

## 7. Step 5 — The Ticket Engine

`DefaultTicketPriceEngine` holds a `List<TicketPriceRule>` and applies each rule in order.

```java
@Service
class DefaultTicketPriceEngine implements TicketPriceEngine {
    private static final Logger log = LoggerFactory.getLogger(DefaultTicketPriceEngine.class);
    private final List<TicketPriceRule> rules;

    public DefaultTicketPriceEngine(List<TicketPriceRule> rules) {
        this.rules = rules;
    }

    @Override
    public TicketPriceResult calculatePrice(TicketPriceRequest ticket) {
        TicketPriceResult result = TicketPriceResult.of(ticket);
        for (TicketPriceRule rule : rules) {
            rule.apply(ticket, result);
        }
        log.debug("Calculated ticket price: seatType={}, movieFormat={}, basePrice={}, finalPrice={}",
                ticket.seatType(),
                ticket.showingContext().movieFormat(),
                result.getBasePrice(),
                result.getTotal()
        );
        return result;
    }
}
```

The rules are registered as Spring beans in order via `TicketRulesConfiguration`:

```java
@Configuration
class TicketRulesConfiguration {

    @Bean @Order(1)
    TicketPriceRule ticketBasePriceRule() { return new TicketBasePriceRule(); }

    @Bean @Order(10)
    TicketPriceRule movieFormatPriceRule() { return new MovieFormatPriceRule(); }

    @Bean @Order(20)
    TicketPriceRule movieDurationPriceRule() { return new MovieDurationPriceRule(); }

    @Bean @Order(30)
    TicketPriceRule seatTypePriceRule() { return new SeatTypePriceRule(); }
}
```

### Why `@Order` spacing matters

Orders are spaced `1, 10, 20, 30` — not `1, 2, 3, 4`. This leaves room to insert a new rule
between two existing ones without renumbering everything. For example, a `LoyaltyCardRule` could
be inserted at `@Order(5)` between the base price and the format rule without touching anything
else.

### The pattern: Chain of Responsibility

This is the **Chain of Responsibility** pattern. Each rule in the chain handles a specific concern
and passes the result on to the next. The engine drives the chain:

```
Request → Rule 1 → Rule 2 → Rule 3 → Rule 4 → Result
```

Each rule:
- Is independent — it has no reference to any other rule
- Is replaceable — swap it out without affecting others
- Is optional — remove it by removing the `@Bean` declaration

---

## 8. Step 6 — Order-Level Rules

### Why some rules cannot live at the ticket level

`BulkBookingDiscountRule` applies a 5% discount when there are **more than 10 tickets**.
`OrderReservationFeeRule` adds a 7% fee when there are **5 or fewer tickets**.

Both rules need to know the **total number of tickets in the order** — information that does not
exist when processing a single `TicketPriceRequest`. Putting them at the ticket level would
require passing order-level data into a ticket-level object — that is **coupling that does not
belong there**.

The solution is a second rule interface that operates on the full order:

```java
public interface OrderPriceRule {
    void apply(OrderPriceRequest request, OrderPriceResult result);
}
```

```java
class OrderReservationFeeRule implements OrderPriceRule {
    private final BigDecimal RESERVATION_FEE_MULTIPLIER = new BigDecimal("0.07");

    @Override
    public void apply(OrderPriceRequest request, OrderPriceResult result) {
        if (request.seatTypes().size() <= 5) {
            result.addOrderAdjustment(
                    AdjustmentType.SURCHARGE,
                    result.getTicketTotal().multiply(RESERVATION_FEE_MULTIPLIER)
            );
        }
    }
}
```

```java
class BulkBookingDiscountRule implements OrderPriceRule {
    private final BigDecimal DISCOUNT_MULTIPLIER = new BigDecimal("0.05");

    @Override
    public void apply(OrderPriceRequest request, OrderPriceResult result) {
        if (request.seatTypes().size() > 10) {
            result.addOrderAdjustment(
                    AdjustmentType.DISCOUNT,
                    result.getTicketBaseTotal().multiply(DISCOUNT_MULTIPLIER)
            );
        }
    }
}
```

`DefaultPriceEngine` wires both levels together:

```java
@Component
public class DefaultPriceEngine implements PriceEngine {
    private static final Logger log = LoggerFactory.getLogger(DefaultPriceEngine.class);
    private final TicketPriceEngine ticketPriceEngine;
    private final List<OrderPriceRule> rules;

    public DefaultPriceEngine(TicketPriceEngine ticketPriceEngine, List<OrderPriceRule> rules) {
        this.ticketPriceEngine = ticketPriceEngine;
        this.rules = rules;
    }

    @Override
    public OrderPriceResult calculatePrice(OrderPriceRequest request) {
        OrderPriceResult result = new OrderPriceResult();

        for (SeatType seatType : request.seatTypes()) {
            TicketPriceRequest ticketRequest = TicketPriceRequest.of(seatType, request.showingContext());
            result.addTicketPrice(ticketPriceEngine.calculatePrice(ticketRequest));
        }

        rules.forEach(rule -> rule.apply(request, result));

        log.info("Price calculated: total={}, seats={}, format={}",
                result.getOrderTotal(),
                request.seatTypes().size(),
                request.showingContext().movieFormat()
        );
        return result;
    }
}
```

> **Low coupling** — `DefaultPriceEngine` knows nothing about individual ticket rules. It only
> depends on `TicketPriceEngine` (an interface) and `List<OrderPriceRule>` (interfaces). The two
> levels are completely independent of each other's implementation details.

---

## 9. Step 7 — Architecture Diagram

```
OrderPriceRequest
      │
      ▼
DefaultPriceEngine
      │
      ├── for each SeatType ──────────────────────────────────────────────┐
      │                                                                    │
      │         TicketPriceRequest                                         │
      │               │                                                    │
      │               ▼                                                    │
      │   DefaultTicketPriceEngine                                         │
      │               │                                                    │
      │         ┌─────┴──────────────────────────────┐                    │
      │         │             │             │         │                    │
      │         ▼             ▼             ▼         ▼                    │
      │  TicketBase    MovieFormat   MovieDuration  SeatType               │
      │  PriceRule     PriceRule     PriceRule      PriceRule              │
      │         │             │             │         │                    │
      │         └─────────────┴─────────────┴─────────┘                    │
      │                       │                                            │
      │               TicketPriceResult ◄─────────────────────────────────┘
      │
      ├── collects List<TicketPriceResult> into OrderPriceResult
      │
      ├── applies OrderPriceRules
      │         │
      │   ┌─────┴──────────────────────┐
      │   │                            │
      │   ▼                            ▼
      │  OrderReservation         BulkBooking
      │  FeeRule                  DiscountRule
      │   │                            │
      │   └─────────────┬──────────────┘
      │                 │
      │         OrderPriceResult
      │
      ▼
OrderPriceResult (returned to caller)
```

---

## 10. Step 8 — Logging

### Why logging matters in a rules engine

When a customer calls support and says "my price was wrong", you need to be able to answer:
- Which rules fired?
- What did each rule contribute?
- What was the final total?

Without logging, you are guessing. With the right logging strategy, you can replay the exact
calculation from the log.

### The three logging levels used in this engine

| Level | Where | What | Volume |
|---|---|---|---|
| `TRACE` | Inside each rule | Which rule fired, what it applied | Very high — once per rule per ticket |
| `DEBUG` | `DefaultTicketPriceEngine` | Per-ticket summary (base, adjustments, final) | High — once per ticket |
| `INFO`  | `DefaultPriceEngine` | Per-order summary (total, seat count, format) | Low — once per order |

### Example: rule-level `TRACE` logging

```java
class SeatTypePriceRule implements TicketPriceRule {
    private static final Logger log = LoggerFactory.getLogger(SeatTypePriceRule.class);

    @Override
    public void apply(TicketPriceRequest request, TicketPriceResult result) {
        BigDecimal surcharge = switch (request.seatType()) {
            case COWBOY  -> BigDecimal.ZERO;
            case REGULAR -> REGULAR_SEAT_SURCHARGE;
            case SOFA    -> VIP_SEAT_SURCHARGE;
        };
        if (surcharge.compareTo(BigDecimal.ZERO) > 0) {
            log.trace("Applying seat type surcharge: seatType={}, surcharge={}",
                    request.seatType(), surcharge);
            result.addAdjustment(AdjustmentType.SURCHARGE, surcharge);
        }
    }
}
```

### Example: engine-level `DEBUG` logging

```java
log.debug("Calculated ticket price: seatType={}, movieFormat={}, basePrice={}, finalPrice={}",
        ticket.seatType(),
        ticket.showingContext().movieFormat(),
        result.getBasePrice(),
        result.getTotal()
);
```

### Example: order-level `INFO` logging

```java
log.info("Price calculated: total={}, seats={}, format={}",
        result.getOrderTotal(),
        request.seatTypes().size(),
        request.showingContext().movieFormat()
);
```

### Controlling log levels in `application.properties`

```properties
# Production — only see order totals
logging.level.ek.osnb.designpattern.priceengine=INFO

# Development — see per-ticket summaries
logging.level.ek.osnb.designpattern.priceengine=DEBUG

# Deep debugging — see every rule firing
logging.level.ek.osnb.designpattern.priceengine=TRACE
```

> **High cohesion + low coupling in logging** — each class logs only what it owns.
> `SeatTypePriceRule` logs the seat surcharge. `DefaultTicketPriceEngine` logs the ticket total.
> `DefaultPriceEngine` logs the order total. No class logs on behalf of another.

### Why `TRACE` inside rules, not `DEBUG`?

In production with many concurrent orders, `DEBUG` on every rule firing for every ticket would
flood the log. `TRACE` is off by default in all environments and only enabled deliberately when
diagnosing a specific pricing problem. `INFO` stays on in production because one line per order
is acceptable volume.

---

## 11. Step 9 — TDD

Tests are written **bottom-up** — the simplest, most isolated units first. By the time you test
the full engine, every component it depends on is already verified.

### Test order

```
TicketBasePriceRuleTest
    MovieFormatPriceRuleTest
        MovieDurationPriceRuleTest
            SeatTypePriceRuleTest
                DefaultTicketPriceEngineTest  ← integration
                    OrderReservationFeeRuleTest
                        BulkBookingDiscountRuleTest
                            DefaultPriceEngineTest  ← full integration
```

All test classes live in:
`src/test/java/ek/osnb/designpattern/priceengine/`

The rule classes are package-private, so the tests must be in the **same package** as the rules.

---

### `TicketBasePriceRuleTest`

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

class TicketBasePriceRuleTest {

    private final TicketBasePriceRule rule = new TicketBasePriceRule();

    @Test
    void shouldSetBasePrice() {
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.REGULAR,
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );
        TicketPriceResult result = TicketPriceResult.of(request);

        rule.apply(request, result);

        assertThat(result.getBasePrice()).isEqualByComparingTo("100.00");
    }
}
```

---

### `MovieFormatPriceRuleTest`

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

class MovieFormatPriceRuleTest {

    private final MovieFormatPriceRule rule = new MovieFormatPriceRule();

    @Test
    void shouldNotAddSurchargeForTwoD() {
        TicketPriceResult result = applyRule(MovieFormat.TWO_D);

        assertThat(result.getAdjustments()).isEmpty();
    }

    @Test
    void shouldAddSurchargeForThreeD() {
        TicketPriceResult result = applyRule(MovieFormat.THREE_D);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("20.00");
    }

    @Test
    void shouldAddSurchargeForImax() {
        TicketPriceResult result = applyRule(MovieFormat.IMAX);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("30.00");
    }

    private TicketPriceResult applyRule(MovieFormat format) {
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.REGULAR,
                ShowingContext.of(format, LocalDateTime.now(), 120)
        );
        TicketPriceResult result = TicketPriceResult.of(request);
        rule.apply(request, result);
        return result;
    }
}
```

---

### `MovieDurationPriceRuleTest`

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

class MovieDurationPriceRuleTest {

    private final MovieDurationPriceRule rule = new MovieDurationPriceRule();

    @Test
    void shouldNotAddSurchargeForShortMovie() {
        TicketPriceResult result = applyRule(170);

        assertThat(result.getAdjustments()).isEmpty();
    }

    @Test
    void shouldAddSurchargeForLongMovie() {
        TicketPriceResult result = applyRule(171);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("15.00");
    }

    private TicketPriceResult applyRule(int duration) {
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.REGULAR,
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), duration)
        );
        TicketPriceResult result = TicketPriceResult.of(request);
        rule.apply(request, result);
        return result;
    }
}
```

---

### `SeatTypePriceRuleTest`

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

class SeatTypePriceRuleTest {

    private final SeatTypePriceRule rule = new SeatTypePriceRule();

    @Test
    void shouldNotAddSurchargeForCowboySeat() {
        TicketPriceResult result = applyRule(SeatType.COWBOY);

        assertThat(result.getAdjustments()).isEmpty();
    }

    @Test
    void shouldAddSurchargeForRegularSeat() {
        TicketPriceResult result = applyRule(SeatType.REGULAR);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("5.00");
    }

    @Test
    void shouldAddSurchargeForSofaSeat() {
        TicketPriceResult result = applyRule(SeatType.SOFA);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("10.00");
    }

    private TicketPriceResult applyRule(SeatType seatType) {
        TicketPriceRequest request = TicketPriceRequest.of(
                seatType,
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );
        TicketPriceResult result = TicketPriceResult.of(request);
        rule.apply(request, result);
        return result;
    }
}
```

---

### `DefaultTicketPriceEngineTest`

This is an **integration test** — all four rules are wired together and the combined result is
verified. No Spring context is needed; we wire the rules manually.

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

class DefaultTicketPriceEngineTest {

    private DefaultTicketPriceEngine engine;

    @BeforeEach
    void setUp() {
        engine = new DefaultTicketPriceEngine(List.of(
                new TicketBasePriceRule(),
                new MovieFormatPriceRule(),
                new MovieDurationPriceRule(),
                new SeatTypePriceRule()
        ));
    }

    @Test
    void shouldCalculateBasePriceOnly_forCowboyTwoDShortMovie() {
        // COWBOY + TWO_D + 120 min → base 100.00, no surcharges
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.COWBOY,
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );

        TicketPriceResult result = engine.calculatePrice(request);

        assertThat(result.getTotal()).isEqualByComparingTo("100.00");
    }

    @Test
    void shouldApplyAllSurcharges_forSofaImaxLongMovie() {
        // SOFA (+10) + IMAX (+30) + 180 min (+15) + base 100 = 155.00
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.SOFA,
                ShowingContext.of(MovieFormat.IMAX, LocalDateTime.now(), 180)
        );

        TicketPriceResult result = engine.calculatePrice(request);

        assertThat(result.getTotal()).isEqualByComparingTo("155.00");
    }

    @Test
    void shouldApplyThreeDAndRegularSurcharge() {
        // REGULAR (+5) + THREE_D (+20) + base 100 = 125.00
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.REGULAR,
                ShowingContext.of(MovieFormat.THREE_D, LocalDateTime.now(), 120)
        );

        TicketPriceResult result = engine.calculatePrice(request);

        assertThat(result.getTotal()).isEqualByComparingTo("125.00");
    }
}
```

---

### `OrderReservationFeeRuleTest`

```java
package ek.osnb.designpattern.priceengine.orderRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.OrderPriceRequest;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.OrderPriceResult;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.Collections;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

class OrderReservationFeeRuleTest {

    private final OrderReservationFeeRule rule = new OrderReservationFeeRule();

    @Test
    void shouldAddReservationFee_whenFiveOrFewerTickets() {
        // 3 tickets, ticket total = 300.00 → fee = 300 * 0.07 = 21.00
        OrderPriceRequest request = requestWithSeats(3);
        OrderPriceResult result = resultWithTicketTotal(new BigDecimal("300.00"), 3);

        rule.apply(request, result);

        assertThat(result.getOrderTotal()).isEqualByComparingTo("321.00");
    }

    @Test
    void shouldNotAddReservationFee_whenMoreThanFiveTickets() {
        // 6 tickets, ticket total = 600.00 → no fee
        OrderPriceRequest request = requestWithSeats(6);
        OrderPriceResult result = resultWithTicketTotal(new BigDecimal("600.00"), 6);

        rule.apply(request, result);

        assertThat(result.getOrderAdjustments()).isEmpty();
    }

    private OrderPriceRequest requestWithSeats(int count) {
        List<SeatType> seats = Collections.nCopies(count, SeatType.REGULAR);
        return OrderPriceRequest.of(seats, ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120));
    }

    private OrderPriceResult resultWithTicketTotal(BigDecimal total, int count) {
        OrderPriceResult result = new OrderPriceResult();
        ShowingContext ctx = ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120);
        BigDecimal perTicket = total.divide(new BigDecimal(count));
        for (int i = 0; i < count; i++) {
            TicketPriceRequest req = TicketPriceRequest.of(SeatType.REGULAR, ctx);
            TicketPriceResult ticket = TicketPriceResult.of(req);
            ticket.setBasePrice(perTicket);
            result.addTicketPrice(ticket);
        }
        return result;
    }
}
```

---

### `BulkBookingDiscountRuleTest`

```java
package ek.osnb.designpattern.priceengine.orderRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.OrderPriceRequest;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.OrderPriceResult;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.Collections;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

class BulkBookingDiscountRuleTest {

    private final BulkBookingDiscountRule rule = new BulkBookingDiscountRule();

    @Test
    void shouldApplyDiscount_whenMoreThanTenTickets() {
        // 11 tickets, base total = 1100.00 → discount = 1100 * 0.05 = 55.00 → total = 1045.00
        OrderPriceRequest request = requestWithSeats(11);
        OrderPriceResult result = resultWithBaseTotal(new BigDecimal("1100.00"), 11);

        rule.apply(request, result);

        assertThat(result.getOrderTotal()).isEqualByComparingTo("1045.00");
    }

    @Test
    void shouldNotApplyDiscount_whenTenOrFewerTickets() {
        // 10 tickets → no discount
        OrderPriceRequest request = requestWithSeats(10);
        OrderPriceResult result = resultWithBaseTotal(new BigDecimal("1000.00"), 10);

        rule.apply(request, result);

        assertThat(result.getOrderAdjustments()).isEmpty();
    }

    private OrderPriceRequest requestWithSeats(int count) {
        List<SeatType> seats = Collections.nCopies(count, SeatType.REGULAR);
        return OrderPriceRequest.of(seats, ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120));
    }

    private OrderPriceResult resultWithBaseTotal(BigDecimal total, int count) {
        OrderPriceResult result = new OrderPriceResult();
        ShowingContext ctx = ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120);
        BigDecimal perTicket = total.divide(new BigDecimal(count));
        for (int i = 0; i < count; i++) {
            TicketPriceRequest req = TicketPriceRequest.of(SeatType.REGULAR, ctx);
            TicketPriceResult ticket = TicketPriceResult.of(req);
            ticket.setBasePrice(perTicket);
            result.addTicketPrice(ticket);
        }
        return result;
    }
}
```

---

### `DefaultPriceEngineTest`

This is the **full integration test** — the complete two-level engine with all rules wired,
verified against a concrete order.

```java
package ek.osnb.designpattern.priceengine.orderRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.OrderPriceRequest;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.response.OrderPriceResult;
import ek.osnb.designpattern.priceengine.ticketRules.DefaultTicketPriceEngine;
import ek.osnb.designpattern.priceengine.ticketRules.MovieDurationPriceRule;
import ek.osnb.designpattern.priceengine.ticketRules.MovieFormatPriceRule;
import ek.osnb.designpattern.priceengine.ticketRules.SeatTypePriceRule;
import ek.osnb.designpattern.priceengine.ticketRules.TicketBasePriceRule;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

class DefaultPriceEngineTest {

    private DefaultPriceEngine engine;

    @BeforeEach
    void setUp() {
        DefaultTicketPriceEngine ticketEngine = new DefaultTicketPriceEngine(List.of(
                new TicketBasePriceRule(),
                new MovieFormatPriceRule(),
                new MovieDurationPriceRule(),
                new SeatTypePriceRule()
        ));
        engine = new DefaultPriceEngine(ticketEngine, List.of(
                new OrderReservationFeeRule(),
                new BulkBookingDiscountRule()
        ));
    }

    @Test
    void shouldApplyReservationFee_forSmallOrder() {
        // 3 x COWBOY + TWO_D + 120 min
        // ticket total = 3 * 100.00 = 300.00
        // reservation fee = 300 * 0.07 = 21.00
        // order total = 321.00
        OrderPriceRequest request = OrderPriceRequest.of(
                List.of(SeatType.COWBOY, SeatType.COWBOY, SeatType.COWBOY),
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );

        OrderPriceResult result = engine.calculatePrice(request);

        assertThat(result.getOrderTotal()).isEqualByComparingTo("321.00");
    }

    @Test
    void shouldApplyBulkDiscount_forLargeOrder() {
        // 11 x COWBOY + TWO_D + 120 min
        // ticket total = 11 * 100.00 = 1100.00
        // bulk discount = 1100 * 0.05 = 55.00
        // order total = 1045.00
        OrderPriceRequest request = OrderPriceRequest.of(
                List.of(SeatType.COWBOY, SeatType.COWBOY, SeatType.COWBOY,
                        SeatType.COWBOY, SeatType.COWBOY, SeatType.COWBOY,
                        SeatType.COWBOY, SeatType.COWBOY, SeatType.COWBOY,
                        SeatType.COWBOY, SeatType.COWBOY),
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );

        OrderPriceResult result = engine.calculatePrice(request);

        assertThat(result.getOrderTotal()).isEqualByComparingTo("1045.00");
    }

    @Test
    void shouldCalculateCorrectly_forMixedOrderWithNoAdjustments() {
        // 6 x mixed seats + TWO_D + 120 min (no reservation fee, no bulk discount)
        // COWBOY(100) + REGULAR(105) + SOFA(110) + COWBOY(100) + REGULAR(105) + SOFA(110) = 630.00
        OrderPriceRequest request = OrderPriceRequest.of(
                List.of(SeatType.COWBOY, SeatType.REGULAR, SeatType.SOFA,
                        SeatType.COWBOY, SeatType.REGULAR, SeatType.SOFA),
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );

        OrderPriceResult result = engine.calculatePrice(request);

        assertThat(result.getOrderTotal()).isEqualByComparingTo("630.00");
    }
}
```

---

## 12. Step 10 — Exercises

---

### Exercise 1 — Add a `PREMIUM` seat type

**What to build:** Add a new `SeatType` value `PREMIUM` that applies a surcharge of **+20.00**.
The compiler will guide you — the exhaustive switch in `SeatTypePriceRule` will fail to compile
until you handle the new value.

**Failing test — add this first:**

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

class PremiumSeatTypeTest {

    private final SeatTypePriceRule rule = new SeatTypePriceRule();

    @Test
    void shouldAddSurchargeForPremiumSeat() {
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.PREMIUM,
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120)
        );
        TicketPriceResult result = TicketPriceResult.of(request);

        rule.apply(request, result);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("20.00");
    }
}
```

<details>
<summary>Show solution</summary>

**1. Add `PREMIUM` to `SeatType.java`:**

```java
public enum SeatType {
    COWBOY,
    REGULAR,
    SOFA,
    PREMIUM
}
```

**2. Handle `PREMIUM` in `SeatTypePriceRule.java`:**

```java
private final BigDecimal PREMIUM_SEAT_SURCHARGE = new BigDecimal("20.00");

BigDecimal surcharge = switch (request.seatType()) {
    case COWBOY  -> BigDecimal.ZERO;
    case REGULAR -> REGULAR_SEAT_SURCHARGE;
    case SOFA    -> VIP_SEAT_SURCHARGE;
    case PREMIUM -> PREMIUM_SEAT_SURCHARGE;
};
```

The compiler will tell you exactly where else `SeatType` is switched on — fix each one.

</details>

---

### Exercise 2 — Weekend surcharge

**What to build:** Add a new `TicketPriceRule` called `WeekendSurchargePriceRule` that adds
**+10.00** when the showing is on a Saturday or Sunday.
Use `request.showingContext().startTime().getDayOfWeek()` to determine the day.

**Failing test — add this first:**

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.time.DayOfWeek;
import java.time.LocalDateTime;
import java.time.temporal.TemporalAdjusters;

import static org.assertj.core.api.Assertions.assertThat;

class WeekendSurchargePriceRuleTest {

    private final WeekendSurchargePriceRule rule = new WeekendSurchargePriceRule();

    @Test
    void shouldAddSurcharge_onSaturday() {
        LocalDateTime saturday = LocalDateTime.now()
                .with(TemporalAdjusters.nextOrSame(DayOfWeek.SATURDAY));
        TicketPriceResult result = applyRule(saturday);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("10.00");
    }

    @Test
    void shouldAddSurcharge_onSunday() {
        LocalDateTime sunday = LocalDateTime.now()
                .with(TemporalAdjusters.nextOrSame(DayOfWeek.SUNDAY));
        TicketPriceResult result = applyRule(sunday);

        assertThat(result.getAdjustments()).hasSize(1);
        assertThat(result.getAdjustments().getFirst().amount()).isEqualByComparingTo("10.00");
    }

    @Test
    void shouldNotAddSurcharge_onWeekday() {
        LocalDateTime monday = LocalDateTime.now()
                .with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY));
        TicketPriceResult result = applyRule(monday);

        assertThat(result.getAdjustments()).isEmpty();
    }

    private TicketPriceResult applyRule(LocalDateTime startTime) {
        TicketPriceRequest request = TicketPriceRequest.of(
                SeatType.REGULAR,
                ShowingContext.of(MovieFormat.TWO_D, startTime, 120)
        );
        TicketPriceResult result = TicketPriceResult.of(request);
        rule.apply(request, result);
        return result;
    }
}
```

<details>
<summary>Show solution</summary>

**1. Create `WeekendSurchargePriceRule.java` in `ticketRules/`:**

```java
package ek.osnb.designpattern.priceengine.ticketRules;

import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.AdjustmentType;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.math.BigDecimal;
import java.time.DayOfWeek;

class WeekendSurchargePriceRule implements TicketPriceRule {
    private static final Logger log = LoggerFactory.getLogger(WeekendSurchargePriceRule.class);
    private final BigDecimal WEEKEND_SURCHARGE = new BigDecimal("10.00");

    @Override
    public void apply(TicketPriceRequest request, TicketPriceResult result) {
        DayOfWeek day = request.showingContext().startTime().getDayOfWeek();
        if (day == DayOfWeek.SATURDAY || day == DayOfWeek.SUNDAY) {
            log.trace("Applying weekend surcharge: day={}, surcharge={}", day, WEEKEND_SURCHARGE);
            result.addAdjustment(AdjustmentType.SURCHARGE, WEEKEND_SURCHARGE);
        }
    }
}
```

**2. Register the rule in `TicketRulesConfiguration.java`:**

```java
@Bean @Order(25)
TicketPriceRule weekendSurchargePriceRule() { return new WeekendSurchargePriceRule(); }
```

`@Order(25)` slots it between `MovieDurationPriceRule` (20) and `SeatTypePriceRule` (30) with
no renumbering needed — this is why `@Order` values are spaced by 10.

</details>

---

### Exercise 3 — Loyalty card discount

**What to build:** Add a new `OrderPriceRule` called `LoyaltyCardDiscountRule` that applies a
**−10%** discount on the ticket total when the order contains a loyalty card holder.

To support this, you will need to add a `boolean loyaltyCard` field to `OrderPriceRequest`.

**Failing test — add this first:**

```java
package ek.osnb.designpattern.priceengine.orderRules;

import ek.osnb.designpattern.priceengine.request.MovieFormat;
import ek.osnb.designpattern.priceengine.request.OrderPriceRequest;
import ek.osnb.designpattern.priceengine.request.SeatType;
import ek.osnb.designpattern.priceengine.request.ShowingContext;
import ek.osnb.designpattern.priceengine.request.TicketPriceRequest;
import ek.osnb.designpattern.priceengine.response.OrderPriceResult;
import ek.osnb.designpattern.priceengine.response.TicketPriceResult;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

class LoyaltyCardDiscountRuleTest {

    private final LoyaltyCardDiscountRule rule = new LoyaltyCardDiscountRule();

    @Test
    void shouldApplyDiscount_whenLoyaltyCardPresent() {
        // ticket total = 200.00 → discount = 200 * 0.10 = 20.00 → total = 180.00
        OrderPriceRequest request = requestWithLoyaltyCard(true, 2);
        OrderPriceResult result = resultWithTicketTotal(new BigDecimal("200.00"), 2);

        rule.apply(request, result);

        assertThat(result.getOrderTotal()).isEqualByComparingTo("180.00");
    }

    @Test
    void shouldNotApplyDiscount_whenNoLoyaltyCard() {
        OrderPriceRequest request = requestWithLoyaltyCard(false, 2);
        OrderPriceResult result = resultWithTicketTotal(new BigDecimal("200.00"), 2);

        rule.apply(request, result);

        assertThat(result.getOrderAdjustments()).isEmpty();
    }

    private OrderPriceRequest requestWithLoyaltyCard(boolean loyaltyCard, int count) {
        List<SeatType> seats = java.util.Collections.nCopies(count, SeatType.REGULAR);
        return new OrderPriceRequest(
                seats,
                ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120),
                loyaltyCard
        );
    }

    private OrderPriceResult resultWithTicketTotal(BigDecimal total, int count) {
        OrderPriceResult result = new OrderPriceResult();
        ShowingContext ctx = ShowingContext.of(MovieFormat.TWO_D, LocalDateTime.now(), 120);
        BigDecimal perTicket = total.divide(new BigDecimal(count));
        for (int i = 0; i < count; i++) {
            TicketPriceRequest req = TicketPriceRequest.of(SeatType.REGULAR, ctx);
            TicketPriceResult ticket = TicketPriceResult.of(req);
            ticket.setBasePrice(perTicket);
            result.addTicketPrice(ticket);
        }
        return result;
    }
}
```

<details>
<summary>Show solution</summary>

**1. Add `loyaltyCard` to `OrderPriceRequest.java`:**

```java
public record OrderPriceRequest(List<SeatType> seatTypes, ShowingContext showingContext, boolean loyaltyCard) {
    public static OrderPriceRequest of(List<SeatType> seatTypes, ShowingContext showingContext) {
        return new OrderPriceRequest(seatTypes, showingContext, false);
    }
}
```

**2. Create `LoyaltyCardDiscountRule.java` in `orderRules/`:**

```java
package ek.osnb.designpattern.priceengine.orderRules;

import ek.osnb.designpattern.priceengine.request.OrderPriceRequest;
import ek.osnb.designpattern.priceengine.response.AdjustmentType;
import ek.osnb.designpattern.priceengine.response.OrderPriceResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.math.BigDecimal;

class LoyaltyCardDiscountRule implements OrderPriceRule {
    private static final Logger log = LoggerFactory.getLogger(LoyaltyCardDiscountRule.class);
    private final BigDecimal LOYALTY_DISCOUNT_MULTIPLIER = new BigDecimal("0.10");

    @Override
    public void apply(OrderPriceRequest request, OrderPriceResult result) {
        if (request.loyaltyCard()) {
            BigDecimal discount = result.getTicketTotal().multiply(LOYALTY_DISCOUNT_MULTIPLIER);
            log.trace("Applying loyalty card discount: discount={}", discount);
            result.addOrderAdjustment(AdjustmentType.DISCOUNT, discount);
        }
    }
}
```

**3. Register in `OrderRulesConfiguration.java`:**

```java
@Bean @Order(20)
OrderPriceRule loyaltyCardDiscountRule() { return new LoyaltyCardDiscountRule(); }
```

</details>

