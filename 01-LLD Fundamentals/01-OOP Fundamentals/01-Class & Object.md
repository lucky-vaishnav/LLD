## 🆕 LLD Topic #1: Class & Object

This is the foundation of almost every LLD discussion.

### 1. What is a Class?

A **class is a blueprint/template** that defines:

* **State** → data/properties
* **Behavior** → methods/functions

For example, in an e-commerce system:

```ts
class Order {
  id: string;
  amount: number;
  status: string;

  placeOrder() {
    // ...
  }

  cancelOrder() {
    // ...
  }
}
```

The class tells us what an `Order` **has** and what an `Order` **can do**.

---

### 2. What is an Object?

An **object is an actual instance of a class**.

```ts
const order1 = new Order();
const order2 = new Order();
```

Here:

```text
Order        → Class / blueprint
order1       → Object
order2       → Object
```

Both objects follow the structure defined by `Order`, but they can have different state.

```text
order1
  id     → ORD-101
  amount → 500
  status → PLACED

order2
  id     → ORD-102
  amount → 1200
  status → CANCELLED
```

---

## 3. The important LLD perspective

Don't think of a class simply as a way to organize code.

In LLD, we use classes to represent **responsibilities/entities in the system**.

For example, suppose we're designing a parking lot:

```text
ParkingLot
Vehicle
ParkingSpot
Ticket
Payment
```

Each class should have a **clear responsibility**.

For example:

```ts
class ParkingSpot {
  private vehicle: Vehicle | null = null;

  park(vehicle: Vehicle) {
    this.vehicle = vehicle;
  }

  removeVehicle() {
    this.vehicle = null;
  }

  isAvailable(): boolean {
    return this.vehicle === null;
  }
}
```

Notice something important:

`ParkingSpot` doesn't just contain data.

It also contains the **behavior associated with that data**.

Instead of doing:

```ts
spot.vehicle = vehicle;
```

we can do:

```ts
spot.park(vehicle);
```

This idea will become **very important when we learn Encapsulation and SOLID**.

---

## 4. Class vs Object — interview answer

If an interviewer asks:

> **What's the difference between a class and an object?**

A good answer:

> A class is a blueprint that defines the state and behavior of an entity, while an object is a concrete instance of that class with its own state. In LLD, classes are typically used to model entities or responsibilities, while objects represent the actual runtime instances interacting with each other.

That's much stronger than simply saying *"class is a blueprint and object is an instance."*

---

## 5. One important design question

When designing an LLD, you shouldn't blindly create a class for everything.

Ask:

> **Does this entity have meaningful state or behavior that deserves its own responsibility?**

For example:

```text
Order
Payment
Customer
Vehicle
ParkingSpot
```

are likely meaningful classes.

But creating something like:

```text
OrderStringHelper
OrderNumberData
OrderStatusContainer
```

without a real responsibility may unnecessarily complicate the design.

This connects directly to **Single Responsibility Principle**, which we'll study later.

---

### 🎯 Today's takeaway

Remember these three things:

```text
Class  → Defines state + behavior
Object → Runtime instance of a class
LLD    → Classes should represent meaningful responsibilities
```

And the most important mindset:

> **Don't design classes just because you can. Design them around responsibilities.**

That's enough for today's topic. Next time, we'll move to **Encapsulation**, where we'll take this `ParkingSpot` example and understand *why* controlling access to state matters in real LLD.

