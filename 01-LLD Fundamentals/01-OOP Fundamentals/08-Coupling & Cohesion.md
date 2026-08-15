## 🆕 LLD Topic #8: Coupling & Cohesion

These two concepts are **fundamental to good LLD** and will become very important when we start SOLID.

---

# 1. What is Coupling?

**Coupling describes how strongly one class/module depends on another.**

Simple mental model:

> **How much does A need to know about B?**

### High coupling

```text
OrderService
     │
     ├── knows Stripe implementation
     ├── knows database implementation
     ├── knows email implementation
     └── knows SMS implementation
```

`OrderService` knows too much about other components.

That's **high coupling**.

### Low coupling

```text
OrderService
     │
     ├── PaymentProcessor
     ├── NotificationService
     └── OrderRepository
```

It depends on abstractions/contracts rather than concrete implementations.

That's **low coupling**.

---

# 2. Why is high coupling a problem?

Suppose:

```ts id="3sg0z0"
class PaymentService {
  private processor = new StripeProcessor();
}
```

Now `PaymentService` is coupled to Stripe.

If we change:

```text id="5t7lce"
Stripe → CyberSource
```

we need to modify `PaymentService`.

And if 10 other classes do the same thing:

```text id="m8l3j7"
BookingService
TripService
RefundService
OrderService
PaymentService
```

the change becomes expensive.

---

# 3. Low coupling

Instead:

```ts id="qz0r9u"
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

and:

```ts id="wy3j3k"
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

Now:

```text id="f1j7n0"
PaymentService
      ↓
PaymentProcessor
      ↑
      │
 ┌────┴─────┐
 │          │
Stripe   CyberSource
```

`PaymentService` doesn't care which implementation exists.

That's **loose coupling**.

---

# 4. What is Cohesion?

Cohesion is about something completely different.

**Cohesion describes how closely related the responsibilities inside a class/module are.**

Simple mental model:

> **Does this class have one focused purpose, or is it doing unrelated things?**

### High cohesion

```text
PaymentService

- processPayment()
- refundPayment()
- validatePayment()
```

These responsibilities are related to payment.

That's **high cohesion**.

### Low cohesion

Imagine:

```text
UserService

- createUser()
- sendEmail()
- calculateParkingFee()
- processPayment()
- generatePDF()
- resizeImage()
```

These responsibilities are unrelated.

That's **low cohesion**.

---

# 5. The ideal LLD

We generally want:

> **Low coupling + High cohesion**

Think:

```text id="2w9t5f"
        GOOD DESIGN

     High Cohesion
           +
      Low Coupling
           ↓
    Maintainable System
```

---

# 6. Let's connect this to everything we've learned

This is where the previous topics start making sense.

### Encapsulation

Protects internal state.

```text id="w4q0n8"
private state
     ↓
controlled behavior
```

### Abstraction

Hides unnecessary implementation details.

```text id="f2kqsk"
PaymentProcessor
```

### Polymorphism

Allows multiple implementations.

```text id="v0i3d7"
PaymentProcessor
   ├── Stripe
   ├── CyberSource
   └── Razorpay
```

### Dependency Injection

Allows us to provide implementations externally.

```text id="3v0my4"
PaymentService
      ↑
      │
PaymentProcessor
```

### Result

```text id="wmy1o9"
        PaymentService
              │
              ↓
      PaymentProcessor
              ↑
       ┌──────┼──────┐
       │      │      │
    Stripe CyberSource Razorpay

        LOW COUPLING
```

This is why these concepts aren't isolated definitions.

They work together to produce good designs.

---

# 7. Coupling example: Bad design

Imagine:

```ts id="r4h6iq"
class BookingService {
  book() {
    const payment = new StripePayment();
    payment.pay();

    const email = new GmailEmailService();
    email.send();

    const db = new MongoDatabase();
    db.save();
  }
}
```

`BookingService` knows:

* Stripe
* Gmail
* MongoDB

That's a lot of implementation knowledge.

Now changing any of those technologies potentially requires changing `BookingService`.

---

# 8. Better design

Define abstractions:

```ts id="q9b0e3"
interface PaymentProcessor {
  pay(): void;
}

interface NotificationService {
  send(): void;
}

interface BookingRepository {
  save(): void;
}
```

Then:

```ts id="n8rj7p"
class BookingService {
  constructor(
    private payment: PaymentProcessor,
    private notification: NotificationService,
    private repository: BookingRepository
  ) {}

  book() {
    this.payment.pay();
    this.repository.save();
    this.notification.send();
  }
}
```

Now `BookingService` focuses on:

> **Booking orchestration**

It doesn't care whether:

```text
Payment → Stripe/CyberSource
Notification → Email/SMS/Push
Repository → MongoDB/PostgreSQL
```

That's both:

**High cohesion + Low coupling.**

---

# 9. Coupling types

You don't need to memorize a huge list right now, but you should understand that coupling can vary.

For example:

```text
Tight coupling
    ↓
Concrete implementation dependency
    ↓
Abstraction dependency
    ↓
Loose coupling
```

Compare:

```ts id="03v5tg"
// Tight
private payment = new StripeProcessor();
```

vs.

```ts id="w5xq4y"
// Loose
constructor(
  private payment: PaymentProcessor
) {}
```

The second design doesn't eliminate dependency.

**That's important.**

You can never realistically have a system with zero dependencies.

The goal is:

> **Manage dependencies and keep them at the right abstraction boundaries.**

---

# 10. Cohesion and Single Responsibility

Cohesion is closely related to **Single Responsibility Principle**, which we'll study next when we enter SOLID.

For example:

```text id="9klg2t"
❌ Low cohesion

BookingService
 ├── book()
 ├── sendEmail()
 ├── generateInvoice()
 ├── resizeImage()
 └── calculateTax()
```

Better:

```text id="d4k5h9"
BookingService
    └── booking responsibilities

NotificationService
    └── notification responsibilities

InvoiceService
    └── invoice responsibilities

TaxService
    └── tax responsibilities
```

Each class has a focused responsibility.

That's **high cohesion**.

---

# 11. Interview question

If an interviewer asks:

> **"What is the difference between coupling and cohesion?"**

A strong answer:

> **"Coupling measures how strongly different components depend on each other, while cohesion measures how closely related the responsibilities within a component are. In general, we aim for low coupling between components and high cohesion within each component because that makes the system easier to change, test, and maintain."**

That's the answer I'd expect from a senior developer.

---

# 🎯 Today's final takeaway

Remember this simple formula:

```text
GOOD LLD
   ↓
High Cohesion
   +
Low Coupling
```

And:

```text
Coupling
→ Relationship BETWEEN components

Cohesion
→ Relationship WITHIN a component
```

That's probably the most useful way to remember the difference.

---

## 📚 LLD Fundamentals Progress

We've now covered:

1. Class & Object
2. Encapsulation
3. Abstraction
4. Inheritance vs Composition
5. Polymorphism
6. Association / Aggregation / Composition
7. Dependency Injection
8. **Coupling & Cohesion** ← Today

We're **very close to finishing the OOP/LLD fundamentals**.
