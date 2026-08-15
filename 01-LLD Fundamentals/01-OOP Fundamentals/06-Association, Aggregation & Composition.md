## 🆕 LLD Topic #6: Association, Aggregation & Composition

This is an important topic because it helps you correctly model **relationships between classes** in LLD.

These three terms are often confused, especially **aggregation vs composition**.

---

# 1. Association

**Association simply means that two classes are related or interact with each other.**

There is no strong ownership implied.

Example:

```text
Doctor ───────── Patient
```

A doctor treats a patient, and a patient may visit multiple doctors.

In code:

```ts
class Doctor {
  treat(patient: Patient) {
    // treat patient
  }
}

class Patient {
  name: string;
}
```

The important thing is:

> **Doctor and Patient can exist independently.**

If the doctor object is deleted, the patient doesn't disappear.

If the patient object is deleted, the doctor doesn't disappear.

That's association.

### Mental model

```text
A knows/uses B
```

---

# 2. Aggregation

Aggregation is a **stronger form of association** where one object represents a collection/group of other objects, but the contained objects can exist independently.

Think:

```text
Team
 ├── Player
 ├── Player
 └── Player
```

A `Team` has players, but players don't depend on the existence of that particular team.

A player can:

```text
Team A
   ↓
Player

Player leaves

Team B
   ↓
Player
```

The player still exists.

In code:

```ts
class Player {
  constructor(public name: string) {}
}

class Team {
  constructor(
    public players: Player[]
  ) {}
}
```

Now:

```ts
const player = new Player("Rahul");

const teamA = new Team([player]);
```

Later:

```ts
const teamB = new Team([player]);
```

The same `Player` can belong to another team.

So:

> **Aggregation = HAS-A relationship, but the child can exist independently.**

---

# 3. Composition

Composition is a **strong ownership relationship**.

The contained object is considered part of the lifecycle of the parent.

Classic example:

```text
House
 ├── Room
 ├── Room
 └── Room
```

A room is considered a part of that particular house.

Another very common LLD example:

```text
Order
 ├── OrderItem
 ├── OrderItem
 └── OrderItem
```

An `OrderItem` belongs to an order.

If the order is permanently removed, its order items generally don't have an independent business meaning.

---

# 4. Code example

```ts
class OrderItem {
  constructor(
    public productId: string,
    public quantity: number
  ) {}
}

class Order {
  private items: OrderItem[] = [];

  addItem(productId: string, quantity: number) {
    const item = new OrderItem(productId, quantity);

    this.items.push(item);
  }
}
```

Notice something important.

The `Order` is responsible for creating and managing its `OrderItem`s.

```text
Order
  │
  ├── creates OrderItem
  ├── owns OrderItem
  └── manages OrderItem
```

That's composition.

---

# 5. The critical difference

The easiest way to distinguish them is **lifecycle ownership**.

### Association

```text
A ───── B
```

Just a relationship.

Both can independently exist.

---

### Aggregation

```text
A ◇──── B
```

A contains/references B, but B can independently exist.

Example:

```text
Team ◇──── Player
```

---

### Composition

```text
A ◆──── B
```

A strongly owns B.

B's lifecycle is tied to A.

Example:

```text
Order ◆──── OrderItem
```

---

# 6. Very important: Don't overthink the UML symbols

In interviews, people sometimes become too focused on:

```text
◇ Aggregation
◆ Composition
```

The more important question is:

> **What is the ownership and lifecycle relationship between these objects?**

Ask:

### Question 1

Can B exist without A?

If yes → potentially **association/aggregation**.

If no → potentially **composition**.

### Question 2

Does A own the lifecycle of B?

If yes → **composition** is likely appropriate.

---

# 7. Real LLD example: Parking System

Let's apply this to something you're already familiar with.

Imagine:

```text
ParkingLot
 ├── ParkingFloor
 │     ├── ParkingSpot
 │     └── ParkingSpot
 │
 └── ParkingFloor
       ├── ParkingSpot
       └── ParkingSpot
```

We could model:

```text
ParkingLot ◆──── ParkingFloor
ParkingFloor ◆──── ParkingSpot
```

because these objects are conceptually components of the parking lot/floor.

But:

```text
ParkingSpot ───── Vehicle
```

is different.

A vehicle exists independently of the parking spot.

The vehicle can leave one spot and enter another:

```text
Vehicle
  │
  ├── ParkingSpot A
  │
  └── ParkingSpot B
```

So this is more naturally an **association**.

This is exactly why understanding relationships matters when designing an LLD.

---

# 8. Composition vs the Composition concept from yesterday

One subtle but important point:

Yesterday we discussed:

> **"Favor composition over inheritance."**

That's a somewhat broader OOP design principle.

Today we're discussing:

> **Composition as a class/object relationship.**

They're related but don't mean exactly the same thing.

Yesterday:

```text
PaymentService
      │
      │ HAS-A
      ↓
PaymentProcessor
```

We were talking about **object composition/delegation instead of inheritance**.

Today we're focusing more specifically on:

```text
Order
  ◆── OrderItem
```

where we're talking about **ownership and lifecycle**.

---

# 9. Interview question

If the interviewer asks:

> **"What's the difference between aggregation and composition?"**

A strong answer is:

> "Both represent a has-a relationship, but composition implies stronger ownership and lifecycle dependency. In aggregation, the contained object can exist independently of the container, whereas in composition, the contained object's lifecycle is generally managed by the parent."

Example:

```text
Aggregation:
Team ─── Player

Composition:
Order ─── OrderItem
```

That's enough. Don't give a five-minute definition unless the interviewer asks for more.

---

# 🎯 Today's takeaway

Keep this hierarchy in your notes:

```text
Association
    ↓
General relationship

Aggregation
    ↓
HAS-A
    ↓
Child can exist independently

Composition
    ↓
Strong HAS-A
    ↓
Parent owns child's lifecycle
```

And the question you should always ask when designing classes:

> **"Who owns whom, and can the child meaningfully exist without the parent?"**

---

### Progress so far

We've covered:

1. Class & Object
2. Encapsulation
3. Abstraction
4. Inheritance vs Composition
5. Polymorphism
6. **Association, Aggregation & Composition** ← Today

We're getting close to finishing the **core OOP/LLD fundamentals**. I'll explicitly tell you when we've completed that fundamentals phase, as requested.

