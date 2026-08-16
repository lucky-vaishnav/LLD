# 🆕 LLD Topic #12: Interface Segregation Principle (ISP)

The **I** in SOLID.

ISP is closely connected to what we just discussed with **LSP**, especially our payment example.

---

## 1. What is ISP?

The formal definition is:

> **Clients should not be forced to depend on interfaces they do not use.**

A simpler way to remember it:

> **Prefer small, focused interfaces over large, general-purpose interfaces.**

---

# 2. The classic problem

Suppose we create:

```ts
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}
```

Now we have:

```ts
class HumanWorker implements Worker {
  work() {}
  eat() {}
  sleep() {}
}
```

Fine.

But suppose we have:

```ts
class RobotWorker implements Worker {
  work() {}

  eat() {
    // Robot doesn't eat
  }

  sleep() {
    // Robot doesn't sleep
  }
}
```

The robot is being **forced to implement methods it doesn't need**.

That's an ISP violation.

---

# 3. Better design

Split the interface:

```ts
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}
```

Now:

```ts
class HumanWorker
  implements Workable, Eatable, Sleepable {

  work() {}
  eat() {}
  sleep() {}
}
```

But:

```ts
class RobotWorker implements Workable {
  work() {}
}
```

Now each class only depends on the capabilities it actually needs.

---

# 4. Why is this called "Interface Segregation"?

Because we're **segregating** one large interface into smaller interfaces.

Instead of:

```text id="j9e8r2"
             Worker
       ┌──────┼──────┐
     work    eat   sleep
```

we create:

```text id="8k2n4z"
Workable
   ↓
 work()

Eatable
   ↓
 eat()

Sleepable
   ↓
 sleep()
```

Classes can then choose what they support.

---

# 5. Very relevant backend example

Let's return to our payment system.

Suppose we initially create:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
  refund(transactionId: string): RefundResult;
  recurringPayment(amount: number): PaymentResult;
}
```

Now imagine:

```text id="t3w4h7"
Stripe
    → pay ✓
    → refund ✓
    → recurring ✓

CyberSource
    → pay ✓
    → refund ✓
    → recurring ✓

SomeProvider
    → pay ✓
    → refund ✗
    → recurring ✗
```

If we force `SomeProvider` to implement everything:

```ts
class SomeProvider implements PaymentProcessor {
  pay() {
    // works
  }

  refund() {
    throw new Error("Not supported");
  }

  recurringPayment() {
    throw new Error("Not supported");
  }
}
```

That's a warning sign.

We're forcing the implementation to depend on capabilities it doesn't have.

---

# 6. Apply ISP

Split the interface:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}

interface RefundProcessor {
  refund(transactionId: string): RefundResult;
}

interface RecurringPaymentProcessor {
  recurringPayment(amount: number): PaymentResult;
}
```

Now:

```text id="x8k4q1"
StripeProcessor
 ├── PaymentProcessor
 ├── RefundProcessor
 └── RecurringPaymentProcessor

CyberSourceProcessor
 ├── PaymentProcessor
 ├── RefundProcessor
 └── RecurringPaymentProcessor

SomeProvider
 └── PaymentProcessor
```

Much cleaner.

---

# 7. ISP and LSP are connected

Remember yesterday's example.

We had:

```ts
interface PaymentProcessor {
  pay();
  refund();
}
```

and a provider that couldn't refund.

That created an **LSP problem** because the implementation couldn't honor the full contract.

ISP helps us prevent that by designing smaller contracts:

```text id="c9x2k7"
PaymentProcessor
      ↓
     pay()

RefundProcessor
      ↓
    refund()
```

Now a provider only claims the capabilities it actually supports.

So:

> **ISP helps create better abstractions, which can also make LSP easier to satisfy.**

---

# 8. Another practical example: Repository

Imagine:

```ts
interface UserRepository {
  create(user: User): void;
  update(user: User): void;
  delete(id: string): void;
  findById(id: string): User;
  findAll(): User[];
  exportToCSV(): string;
}
```

Suppose your `UserService` only needs:

```text id="3e4q8h"
findById()
create()
```

Why should it depend on:

```text id="m2p7z0"
delete()
findAll()
exportToCSV()
```

if it doesn't use them?

A more focused design could be:

```ts
interface UserReader {
  findById(id: string): User;
}

interface UserWriter {
  create(user: User): void;
  update(user: User): void;
}

interface UserDeleter {
  delete(id: string): void;
}
```

Now consumers depend only on what they actually need.

---

# 9. ISP doesn't mean every interface should have one method

This is an important interview nuance.

Don't interpret ISP as:

> "Every interface should contain exactly one method."

That's **not** the principle.

For example:

```ts
interface UserReader {
  findById(id: string): User;
  findByEmail(email: string): User;
  findAll(): User[];
}
```

These methods are closely related.

That's a perfectly reasonable interface.

The question is:

> **Are these methods part of a cohesive capability that the client actually needs?**

If yes, they can belong together.

---

# 10. ISP and frontend/backend APIs

This principle also appears outside traditional OOP.

Imagine an API that returns:

```json
{
  "user": {},
  "payment": {},
  "trips": {},
  "notifications": {},
  "adminSettings": {},
  "reports": {}
}
```

for every client even though each client only needs a small portion.

That's conceptually similar to an overly broad contract.

Smaller, purpose-specific contracts can sometimes be better.

The same underlying idea applies:

> **Don't force consumers to depend on things they don't need.**

---

# 11. How to identify an ISP violation

Look for these signals:

### 🚩 Large interfaces

```text id="n4q6y1"
Interface
 ├── method A
 ├── method B
 ├── method C
 ├── method D
 ├── method E
 └── method F
```

### 🚩 Implementations throwing:

```ts id="h4k3t7"
throw new Error("Not supported");
```

### 🚩 Empty implementations:

```ts id="y6q8s2"
method() {}
```

### 🚩 Clients using only a small subset

```text id="8z0p1w"
Client A → methods A + B

Client B → methods C + D

Client C → method E
```

These are strong signals that the interface may be too broad.

---

# 12. ISP vs SRP

They sound similar, but they're applied at different levels.

### SRP

Focuses on:

> **Does this class have too many responsibilities?**

### ISP

Focuses on:

> **Does this interface expose too many unrelated/unneeded capabilities to its clients?**

Example:

```text id="7v3j1m"
SRP
Class responsibility
       ↓
"Does this class do too many unrelated things?"

ISP
Interface contract
       ↓
"Does this interface force clients
 to depend on things they don't need?"
```

---

# 13. Senior-level interview answer

If asked:

> **"What is the Interface Segregation Principle?"**

A strong answer:

> "ISP says that clients shouldn't be forced to depend on methods or capabilities they don't use. Instead of creating large interfaces covering many responsibilities, I prefer smaller, cohesive interfaces based on actual client requirements. This reduces unnecessary coupling and prevents implementations from having to provide unsupported behavior."

That's a good senior-level answer.

---

# 🎯 Today's key takeaway

Remember:

```text id="k7m4w2"
❌ Large interface
       ↓
Everyone depends on everything

✅ Focused interfaces
       ↓
Each client depends only
on what it actually needs
```

And the simplest mental model:

> **Don't force a class to promise capabilities it doesn't actually have.**

---

## 📚 Current Progress

We've now covered:

1. Class & Object
2. Encapsulation
3. Abstraction
4. Inheritance vs Composition
5. Polymorphism
6. Association / Aggregation / Composition
7. Dependency Injection
8. Coupling & Cohesion
9. SRP
10. OCP
11. LSP
12. **ISP ← Today**

Only **DIP — Dependency Inversion Principle** remains among the five SOLID principles.
