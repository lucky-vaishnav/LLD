# 🆕 LLD Topic #13: Dependency Inversion Principle (DIP)

The **D** in SOLID.

This is especially important because we've already learned **Dependency Injection**, and DIP is closely related to it—but they are **not the same thing**.

---

## 1. What is DIP?

The formal definition has two parts:

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

And:

> **Abstractions should not depend on details. Details should depend on abstractions.**

That sounds complicated, so let's break it down.

---

# 2. High-level vs Low-level

Imagine:

```text id="m9j6p2"
OrderService
     ↓
StripePayment
```

`OrderService` represents **business logic**.

So it's a **high-level module**.

`StripePayment` is an implementation/detail.

So it's a **low-level module**.

Currently:

```text id="3v7c8k"
OrderService
      ↓
StripePayment
```

The high-level business logic directly depends on a specific technical implementation.

That's what DIP tries to avoid.

---

# 3. Apply an abstraction

Create:

```ts id="z6c8y2"
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

Then:

```text id="g2n8x5"
          PaymentProcessor
                ↑
                │
        ┌───────┴───────┐
        │               │
StripeProcessor   CyberSourceProcessor
```

Now:

```ts id="1u6b4q"
class OrderService {
  constructor(
    private paymentProcessor: PaymentProcessor
  ) {}

  createOrder(amount: number) {
    this.paymentProcessor.pay(amount);

    // order business logic
  }
}
```

Now the dependency direction is:

```text id="7j4xq9"
        OrderService
             ↓
     PaymentProcessor
        (abstraction)
             ↑
       StripeProcessor
```

Both high-level and low-level code depend on the abstraction.

That's DIP.

---

# 4. The key difference from DI

This is **very important for interviews**.

### Dependency Injection

A technique:

> **How do we provide the dependency?**

Example:

```ts id="q9x2y7"
constructor(
  private processor: PaymentProcessor
) {}
```

We're injecting the dependency.

---

### Dependency Inversion Principle

A design principle:

> **How should dependencies be structured?**

Instead of:

```text id="z7h2k4"
High-level
    ↓
Low-level
```

we want:

```text id="f3x9m1"
High-level
    ↓
Abstraction
    ↑
Low-level
```

So:

```text id="8j5m0q"
DIP
 ↓
Design principle

DI
 ↓
Implementation technique
```

**DI can be used to implement a DIP-friendly design, but DI itself is not DIP.**

---

# 5. Real backend example

Suppose you have:

```ts id="4q8k2m"
class BookingService {
  private payment = new CyberSourcePayment();
}
```

This is tightly coupled.

Now imagine:

```text id="x4p7s1"
BookingService
      ↓
CyberSourcePayment
```

If the business later says:

> "We want to support another payment provider."

You have to modify `BookingService`.

Instead:

```ts id="d6q1v8"
interface PaymentGateway {
  charge(amount: number): PaymentResult;
}
```

Then:

```ts id="r4k8z2"
class BookingService {
  constructor(
    private paymentGateway: PaymentGateway
  ) {}

  book() {
    this.paymentGateway.charge(100);
  }
}
```

Implementations:

```text id="5n7q3p"
PaymentGateway
      ↑
 ┌────┼───────────┐
 │    │           │
Cyber Stripe   Razorpay
```

Now `BookingService` is independent of the concrete provider.

---

# 6. Why is DIP useful?

### 1. Loose coupling

Business logic doesn't know implementation details.

### 2. Easier testing

You can inject:

```ts id="m8y4r2"
MockPaymentGateway
```

instead of the real provider.

### 3. Easier replacement

```text id="n7k1x9"
CyberSource
     ↓
Razorpay
```

without changing the business logic.

### 4. Better maintainability

Changes to infrastructure are isolated.

---

# 7. Another example: Database

Bad:

```ts id="3t8m5y"
class UserService {
  private db = new MongoDB();
}
```

Now:

```text id="1z5r7p"
UserService
     ↓
MongoDB
```

Your business logic directly depends on MongoDB.

Better:

```ts id="h6w9q3"
interface UserRepository {
  findById(id: string): User;
  save(user: User): void;
}
```

Then:

```ts id="e2p4n8"
class UserService {
  constructor(
    private repository: UserRepository
  ) {}
}
```

Implementations:

```text id="u7k3v5"
          UserRepository
                ↑
        ┌───────┴────────┐
        │                │
MongoUserRepository   PostgresUserRepository
```

Now:

```text id="k4q8m2"
UserService
     ↓
UserRepository
     ↑
Mongo / PostgreSQL
```

That's DIP.

---

# 8. The "inversion" part

Why is it called **Dependency Inversion**?

Originally:

```text id="j7p3w1"
High-level business logic
          ↓
Low-level implementation
```

The business logic depends directly on the implementation.

After applying DIP:

```text id="r8k2v6"
High-level business logic
          ↓
      Abstraction
          ↑
Low-level implementation
```

The dependency direction is effectively inverted.

Instead of the high-level module saying:

> "I depend on MongoDB."

it says:

> "I need something that provides UserRepository behavior."

Then MongoDB/PostgreSQL implementations satisfy that abstraction.

---

# 9. DIP is NOT "everything needs an interface"

This is an important senior-level point.

You shouldn't create:

```text id="e8j2m4"
UserServiceInterface
UserServiceImpl
UserServiceFactory
UserServiceProvider
```

for every single class just because you've learned DIP.

That's over-engineering.

DIP is particularly useful at **important architectural boundaries**, such as:

```text id="q1m7x5"
Business logic
      ↓
Database
External API
Payment provider
Message queue
File storage
Email provider
```

These are areas where implementation details are likely to vary.

---

# 10. DIP + DI + Abstraction + Polymorphism

Now look at how everything we've learned comes together.

```text id="t7p3k9"
                    Abstraction
                         │
                         ↓
                  PaymentGateway
                         ↑
               ┌─────────┼─────────┐
               │         │         │
            Stripe    CyberSource Razorpay
               │         │         │
               └─────────┼─────────┘
                         │
                         ↓
                 Dependency Injection
                         │
                         ↓
                  BookingService
```

### Abstraction

Defines the contract.

### Polymorphism

Allows different implementations.

### DIP

Keeps business logic independent from implementation details.

### DI

Provides the implementation to the business logic.

### Result

**Loose coupling + replaceability + testability.**

This is the core of a lot of good backend LLD.

---

# 11. DIP vs OCP

These also work together.

### OCP

> Add new implementations without modifying stable business logic.

### DIP

> Business logic should depend on abstractions rather than concrete implementations.

For example:

```text id="w8n3k2"
                  PaymentGateway
                        ↑
        ┌───────────────┼───────────────┐
        │               │               │
      Stripe        CyberSource      Razorpay
                        ↑
                        │
                  BookingService
```

DIP gives us the dependency structure.

OCP gives us the ability to extend it.

---

# 12. Senior-level interview answer

If asked:

> **"What is the Dependency Inversion Principle?"**

A strong answer:

> "DIP says that high-level business logic should not depend directly on low-level implementation details. Both should depend on abstractions, and the concrete implementations should depend on those abstractions. For example, a BookingService should depend on a PaymentGateway interface rather than directly on CyberSource. This keeps business logic decoupled from infrastructure and makes implementations easier to replace and test."

Then if the interviewer asks:

> "Is DIP the same as dependency injection?"

Answer:

> "No. DIP is a design principle about dependency direction, while dependency injection is a technique for supplying dependencies from outside. DI is commonly used to implement a design that follows DIP."

That's an excellent distinction to remember.

---

# 🎯 Today's key takeaway

The simplest mental model:

```text id="6g8p4r"
❌ Bad

Business Logic
      ↓
Concrete Implementation


✅ Better

Business Logic
      ↓
  Abstraction
      ↑
Concrete Implementation
```

And:

> **High-level business rules should not be tightly coupled to low-level technical details.**

---

# 🎉 SOLID Completed

We've now covered all five SOLID principles:

| Principle   | Core idea                                            |
| ----------- | ---------------------------------------------------- |
| **S — SRP** | One reason to change                                 |
| **O — OCP** | Extend without repeatedly modifying stable code      |
| **L — LSP** | Subtypes must honor the abstraction's contract       |
| **I — ISP** | Don't force clients to depend on unused capabilities |
| **D — DIP** | Depend on abstractions, not concrete details         |
