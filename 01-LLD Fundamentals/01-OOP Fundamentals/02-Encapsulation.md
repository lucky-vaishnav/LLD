Let's continue with the next foundational concept.

## 🆕 LLD Topic #2: Encapsulation

This is one of the **most important OOP concepts for LLD**, because it directly affects how we design classes and control responsibilities.

---

## 1. What is Encapsulation?

**Encapsulation means keeping an object's data and the operations that modify that data together, while controlling how the outside world can access or modify it.**

In simple terms:

> **Don't let everyone directly change an object's internal state. Let the object control its own state.**

For example, instead of:

```ts
class BankAccount {
  balance: number;
}

const account = new BankAccount();

account.balance = -50000;
```

we have a problem.

Anyone can put the object into an invalid state.

A better design:

```ts
class BankAccount {
  private balance = 0;

  deposit(amount: number) {
    if (amount <= 0) {
      throw new Error("Invalid amount");
    }

    this.balance += amount;
  }

  withdraw(amount: number) {
    if (amount <= 0 || amount > this.balance) {
      throw new Error("Invalid withdrawal");
    }

    this.balance -= amount;
  }

  getBalance() {
    return this.balance;
  }
}
```

Now:

```ts
const account = new BankAccount();

account.deposit(1000);
account.withdraw(300);

console.log(account.getBalance());
```

But this is **not allowed**:

```ts
account.balance = -50000; // ❌
```

The `BankAccount` itself controls how its balance changes.

---

# 2. Why is this important in LLD?

Suppose we're designing a `ParkingSpot`.

### Poor design

```ts
class ParkingSpot {
  vehicle: Vehicle | null;
}
```

Other classes can do:

```ts
spot.vehicle = vehicle;
spot.vehicle = null;
```

Now anybody can modify the state.

Imagine:

```text
ParkingManager
PaymentService
TicketService
VehicleService
AdminService
```

all directly modifying `ParkingSpot.vehicle`.

Very quickly, we lose control over the object's state.

---

### Better design

```ts
class ParkingSpot {
  private vehicle: Vehicle | null = null;

  park(vehicle: Vehicle) {
    if (this.vehicle !== null) {
      throw new Error("Spot already occupied");
    }

    this.vehicle = vehicle;
  }

  removeVehicle() {
    if (this.vehicle === null) {
      throw new Error("Spot is already empty");
    }

    this.vehicle = null;
  }

  isAvailable(): boolean {
    return this.vehicle === null;
  }
}
```

Now the object itself guarantees:

```text
ParkingSpot
    │
    ├── park()
    │     └── validates state
    │
    ├── removeVehicle()
    │     └── validates state
    │
    └── isAvailable()
```

This is **encapsulation**.

---

# 3. Encapsulation ≠ Just `private`

This is a very important interview point.

A common answer is:

> "Encapsulation means making variables private."

That's **partially correct but incomplete**.

`private` is a mechanism that helps us achieve encapsulation.

The actual goal is:

> **Protect the object's invariants and control how its state changes.**

For example:

```ts
private balance: number;
```

by itself doesn't give us good encapsulation.

We also need controlled operations:

```ts
deposit()
withdraw()
```

and validation around them.

---

# 4. What is an invariant?

This is a useful LLD term.

An **invariant** is a condition that should always remain true for an object.

For a bank account:

```text
balance >= 0
```

For a parking spot:

```text
A spot can contain at most one vehicle.
```

For an order:

```text
A delivered order cannot be cancelled.
```

A well-designed class should try to **protect its own invariants**.

For example:

```ts
class Order {
  private status: "PLACED" | "SHIPPED" | "DELIVERED" | "CANCELLED" = "PLACED";

  cancel() {
    if (this.status === "DELIVERED") {
      throw new Error("Delivered order cannot be cancelled");
    }

    this.status = "CANCELLED";
  }
}
```

We don't allow another class to simply do:

```ts
order.status = "CANCELLED";
```

because that other class may not know the business rules.

---

# 5. Encapsulation vs Abstraction

These two are frequently confused.

### Encapsulation

Focuses on:

> **How do we protect and control internal state?**

Example:

```ts
private balance;
deposit();
withdraw();
```

### Abstraction

Focuses on:

> **What should the outside world know/use while hiding implementation details?**

Example:

```ts
paymentProcessor.pay();
```

The caller doesn't need to know whether internally we use:

```text
Stripe
Braintree
CyberSource
PayPal
```

We'll study **Abstraction** separately.

---

# 6. Interview-level example

Imagine an interviewer asks:

> Why shouldn't we expose class properties publicly?

Don't simply answer:

> "Because of encapsulation."

A stronger answer:

> Public mutable state allows any part of the system to change an object's state without enforcing its business rules. By controlling state changes through methods, the object can validate operations and maintain its invariants. This reduces coupling and makes the class easier to maintain and reason about.

That's the type of answer we want to develop for senior-level LLD interviews.

---

# 7. The mental model to remember

Think of a class as a **guardian of its own state**.

```text
                 Outside World
                       │
                       │
                 allowed operations
                       ↓
              ┌─────────────────┐
              │     Object      │
              │                 │
              │  private state  │
              │       │         │
              │       ↓         │
              │   validation    │
              │       │         │
              │       ↓         │
              │  state changes  │
              └─────────────────┘
```

Instead of:

```text
Outside → directly modify state ❌
```

we want:

```text
Outside → request operation → Object validates → Object changes state ✅
```

---

## 🎯 Today's key notes

Write these down:

> **Encapsulation = protecting an object's internal state and controlling how that state changes.**

And remember:

1. `private` helps implement encapsulation.
2. Encapsulation is **not just about access modifiers**.
3. Classes should protect their **invariants**.
4. Business rules should live close to the state they govern.
5. Encapsulation reduces uncontrolled coupling between classes.

### One interview question to think about

**If encapsulation is so useful, should every property in a class always be `private`?**

Think about that before our next topic. We can discuss the answer and then move to **Abstraction**.

---
### My Thought-

Yeah, for this question, if encapsulation is useful, should every property in the class always be private? So according to whatever we have discussed, according to me, yes, to achieve this type of thing, or we should do encapsulation for all the business functionality and the business rule, which is a good approach. And if we can do it and make all variables private, so it will be better to handle and control the property state or the object state, which is very good. But if there can be any cases where we want some variable to be publicly accessible, mostly I think we can do it for the read operations. Then in those cases, we can make it public. But once we make properties public, then we should be responsible for those variables. And while making any update or functionality, the project developer should know that these properties are public, and where else this property has been used. And there will be some type of impact analysis should be done whenever we update the public properties. So that's kind of checking and regression type of checking will be required, and the responsibility will be increased. But according to me, we should use encapsulation for most of the part of the project, which will be very easier to handle the project in a long term. But yeah, you can share your answer.

---
Yes — your reasoning is **mostly correct**, and you've identified the important trade-off. There is just one adjustment I'd make to your mental model.

### The key point

**Encapsulation does not mean every property must be `private`.**

It means:

> **The class should control access to the state according to the responsibility and invariants of that state.**

So `private` is a **tool for achieving encapsulation**, not the definition of encapsulation.

### Your read-only point is especially good

For example:

```ts
class BankAccount {
  private balance = 0;

  getBalance(): number {
    return this.balance;
  }
}
```

The outside world can **read** the balance but cannot arbitrarily modify it.

In TypeScript, you can also express this with a `readonly` property when appropriate:

```ts
class User {
  readonly id: string;

  constructor(id: string) {
    this.id = id;
  }
}
```

Here the ID can be publicly accessed:

```ts
user.id; // ✅
```

but cannot be reassigned:

```ts
user.id = "123"; // ❌
```

That's often preferable to making something mutable `public`.

---

### One thing I'd change from your answer

You said:

> "If we make properties public, developers should know where the property is used and do impact analysis."

That's true from a **maintenance perspective**, but ideally we shouldn't design our LLD around that burden.

Instead of:

```ts
class Order {
  public status: string;
}
```

and then telling developers:

> "Be careful when changing `status`."

we should ask:

> **Does the outside world actually need to modify `status` directly?**

Usually, the answer is no.

We would prefer:

```ts
class Order {
  private status: OrderStatus;

  ship() {
    // validate transition
    this.status = OrderStatus.SHIPPED;
  }

  cancel() {
    // validate transition
    this.status = OrderStatus.CANCELLED;
  }

  getStatus() {
    return this.status;
  }
}
```

Now the class itself owns the rules.

---

### Senior-level mental model

Don't think:

> **"Make everything private."**

Think:

> **"Expose the minimum necessary interface and keep ownership of business rules inside the appropriate class."**

So we can have:

| Access                      | Typical use                                   |
| --------------------------- | --------------------------------------------- |
| `private`                   | Internal state/business logic                 |
| `public` method             | Operations that others are allowed to perform |
| `public` read-only property | State that is safe to expose directly         |
| `protected`                 | Specialized inheritance cases; use carefully  |

And there's another important concept here:

**Encapsulation is about controlling behavior, not merely hiding data.**

That's why this:

```ts
order.cancel();
```

is much better LLD than:

```ts
order.status = "CANCELLED";
```

because the first says:

> "Order, please perform the cancellation."

The `Order` object then decides whether cancellation is valid.

---

### 🎯 Your answer in interview form

If the interviewer asks:

> **Should every property always be private?**

I'd answer:

> "Not necessarily. Encapsulation doesn't require every property to be private. The goal is to control access to state and ensure that the class can maintain its invariants. Internal mutable state should generally be private, while read-only or safely exposed state can be public when appropriate. More importantly, business operations should be exposed through well-defined methods rather than allowing callers to directly modify state."

**That's a strong senior-level answer.**

And you understood the core idea correctly. 👍

We can now move to **Abstraction**, which is the next OOP concept and is very important because it connects directly to **interfaces, dependency inversion, and design patterns**.

