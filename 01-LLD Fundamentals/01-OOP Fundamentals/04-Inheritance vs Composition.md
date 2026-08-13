## 🆕 LLD Topic #4: Inheritance vs Composition

This is a **very important LLD topic**, especially for senior-level interviews.

The basic question is:

> **When should one class inherit from another, and when should we compose objects together?**

A lot of poor LLD designs come from overusing inheritance.

---

# 1. Inheritance

Inheritance represents an **"is-a" relationship**.

For example:

```text
Vehicle
  ↑
  │
  ├── Car
  ├── Bike
  └── Truck
```

In TypeScript:

```ts
class Vehicle {
  move() {
    console.log("Moving");
  }
}

class Car extends Vehicle {
  drive() {
    console.log("Driving");
  }
}
```

Now:

```ts
const car = new Car();

car.move();  // inherited
car.drive(); // own behavior
```

The `Car` **is a** `Vehicle`.

That's a legitimate inheritance relationship.

---

# 2. Composition

Composition means a class **contains or uses another object** to accomplish its behavior.

This represents a **"has-a" relationship**.

For example:

```text
Car
 │
 ├── Engine
 └── Transmission
```

```ts
class Engine {
  start() {
    console.log("Engine started");
  }
}

class Car {
  constructor(private engine: Engine) {}

  start() {
    this.engine.start();
  }
}
```

Here:

```text
Car HAS an Engine
```

We're using composition.

---

# 3. The simplest way to remember

```text
Inheritance → IS-A

Composition → HAS-A
```

Examples:

```text
Car IS-A Vehicle
Dog IS-A Animal

Car HAS-A Engine
Order HAS-A Payment
ParkingLot HAS-A ParkingSpot
```

---

# 4. Why inheritance can become dangerous

Consider this design:

```ts
class Bird {
  fly() {
    console.log("Flying");
  }
}

class Sparrow extends Bird {}

class Penguin extends Bird {}
```

At first it looks reasonable:

```text
Bird
 ├── Sparrow
 └── Penguin
```

But:

```ts
const penguin = new Penguin();

penguin.fly();
```

What happens?

A penguin can't fly.

We've created an inheritance relationship that doesn't correctly represent the behavior.

This leads us toward an important SOLID principle:

**Liskov Substitution Principle (LSP).**

We'll study that later.

---

# 5. Another common problem: inheritance creates tight coupling

Suppose:

```ts
class PaymentService {
  processPayment() {}
}

class StripePaymentService extends PaymentService {
  processStripePayment() {}
}
```

Later you need:

```text
Stripe
Braintree
CyberSource
PayPal
```

You could create:

```text
PaymentService
   │
   ├── StripePaymentService
   ├── BraintreePaymentService
   ├── CyberSourcePaymentService
   └── PayPalPaymentService
```

This can work, but you're coupling the child classes to the parent implementation.

Composition often gives us more flexibility.

---

# 6. Composition approach

Remember our abstraction from yesterday:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

Now:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}

  pay(amount: number) {
    return this.processor.pay(amount);
  }
}
```

And:

```ts
class StripeProcessor implements PaymentProcessor {
  pay(amount: number) {
    // Stripe implementation
  }
}

class BraintreeProcessor implements PaymentProcessor {
  pay(amount: number) {
    // Braintree implementation
  }
}
```

Now we can do:

```ts
const stripe = new StripeProcessor();

const paymentService = new PaymentService(stripe);
```

or:

```ts
const braintree = new BraintreeProcessor();

const paymentService = new PaymentService(braintree);
```

The `PaymentService` doesn't need to change.

This is one of the biggest advantages of composition.

---

# 7. Composition gives us replaceability

Imagine:

```text
PaymentService
      │
      ↓
PaymentProcessor
      │
      ├── StripeProcessor
      ├── BraintreeProcessor
      └── CyberSourceProcessor
```

At runtime, we can provide different implementations.

Even better, during testing:

```ts
class MockPaymentProcessor implements PaymentProcessor {
  pay(amount: number) {
    return { success: true };
  }
}
```

Then:

```ts
const service = new PaymentService(
  new MockPaymentProcessor()
);
```

We don't need the real payment provider.

This makes the system much easier to test.

---

# 8. Composition also allows behavior to change dynamically

This is a major difference.

With inheritance:

```text
Class behavior
     ↓
mostly determined by class hierarchy
```

With composition:

```text
Object
  ↓
can delegate to another object
  ↓
behavior can be changed/replaced
```

For example, imagine a ride booking system:

```ts
interface PricingStrategy {
  calculatePrice(distance: number): number;
}
```

Implementations:

```ts
class NormalPricing implements PricingStrategy {}

class SurgePricing implements PricingStrategy {}

class CorporatePricing implements PricingStrategy {}
```

Then:

```ts
class Ride {
  constructor(
    private pricingStrategy: PricingStrategy
  ) {}

  calculateFare(distance: number) {
    return this.pricingStrategy.calculatePrice(distance);
  }
}
```

Now the pricing behavior is composed into the `Ride`.

This is actually the **Strategy Pattern**.

We'll study it properly later.

---

# 9. So should we NEVER use inheritance?

**No.**

Inheritance is perfectly valid when the relationship genuinely represents a stable **is-a** relationship and the child can safely substitute for the parent.

For example:

```text
Vehicle
 ├── Car
 ├── Bike
 └── Truck
```

can be reasonable if all vehicles satisfy the expected `Vehicle` behavior.

Inheritance is also useful when:

* there is a genuine subtype relationship
* shared behavior/state is meaningful
* the hierarchy is stable
* substitutability is maintained

The mistake is using inheritance merely to **reuse code**.

---

# 10. Very important interview point

A common junior-level thought is:

> "I have common code, so I'll create a parent class."

That's not always the right reason.

Code reuse alone doesn't justify inheritance.

Ask:

> **Is this really an IS-A relationship?**

If not, composition is often a better choice.

For example:

```text
Car HAS-A Engine
```

not:

```text
Car IS-A Engine ❌
```

So:

```ts
class Car {
  constructor(private engine: Engine) {}
}
```

is better than:

```ts
class Car extends Engine {} // ❌
```

---

# 11. Composition vs Inheritance

Keep this mental model:

|                         | Inheritance     | Composition                     |
| ----------------------- | --------------- | ------------------------------- |
| Relationship            | IS-A            | HAS-A                           |
| Coupling                | Usually tighter | Usually looser                  |
| Reuse                   | Through parent  | Through delegation              |
| Flexibility             | Lower           | Higher                          |
| Runtime behavior change | Limited         | Easier                          |
| Testing                 | Can be harder   | Usually easier                  |
| Best use                | Genuine subtype | Replaceable behavior/components |

---

# 12. Senior-level interview answer

If the interviewer asks:

> **"Why do you prefer composition over inheritance?"**

Don't say:

> "Composition is better."

Instead:

> "I prefer composition when the relationship is not a true subtype relationship or when I need interchangeable behavior. Composition reduces coupling to a class hierarchy and allows behavior to be changed or replaced independently. I use inheritance when there is a genuine stable IS-A relationship and the subtype can safely satisfy the parent contract."

That's a strong answer.

---

## 🎯 Today's key takeaway

Remember:

```text
Inheritance
    ↓
IS-A
    ↓
Use for genuine subtype relationships

Composition
    ↓
HAS-A / USES-A
    ↓
Prefer when behavior/components should be replaceable
```

And one principle worth remembering:

> **Favor composition over inheritance — but don't treat it as an absolute rule.**

The important thing is **choosing based on the relationship and design requirements**, not blindly following a rule.

### One question for you

Consider:

```text
class NotificationService
class EmailSender
class SmsSender
class PushSender
```

Would you design `EmailSender`, `SmsSender`, and `PushSender` using **inheritance from `NotificationService`**, or would you use **composition + abstraction**?

Think through **why**, not just which one. That's exactly the kind of reasoning LLD interviews test.

---
### My Thought-
So, what do, email, SMS, and push is a type of notification. So, we can use inheritance here from those notification service because these are the notification, but the implementation is different. And if you use the composition, only notification for email, SMS, and push. So it will not, will be able to properly compose because like send, so we will need to decide that what to send, email, SMS, push. So it will not support properly the composition. We can do it, but for here the inheritance support will be better with the different implementation of email, SMS, and push sender.

---
Yes — **your conclusion is reasonable**, but there is one important distinction I'd like you to make.

You said:

> Email, SMS, and Push are types of Notification, so inheritance makes sense.

**Exactly.** If our domain model genuinely considers them different types of notifications, inheritance can be appropriate.

But I'd separate **two different concepts** here:

### 1. Notification as the abstraction

We could have:

```ts
interface Notification {
  send(message: string): void;
}
```

Then:

```text
Notification
   ├── EmailNotification
   ├── SmsNotification
   └── PushNotification
```

That's a valid **polymorphic design**.

The caller can simply do:

```ts
notification.send(message);
```

without knowing which notification it is.

That's actually **abstraction + polymorphism** working together.

---

### 2. NotificationService as the orchestrator

I'd be careful about doing:

```text
NotificationService
   ↑
   ├── EmailSender
   ├── SmsSender
   └── PushSender
```

because `NotificationService` sounds more like a **service/orchestrator**, while Email/SMS/Push are different notification implementations.

A cleaner design could be:

```text
                Notification
                     ↑
          ┌──────────┼──────────┐
          │          │          │
       Email       SMS        Push
          │          │          │
          └──────────┼──────────┘
                     ↑
            NotificationService
```

Where `NotificationService` depends on the abstraction:

```ts
class NotificationService {
  constructor(
    private notification: Notification
  ) {}

  send(message: string) {
    this.notification.send(message);
  }
}
```

Then:

```ts
const email = new EmailNotification();

const service = new NotificationService(email);
service.send("Hello");
```

Or:

```ts
const sms = new SmsNotification();

const service = new NotificationService(sms);
service.send("Hello");
```

That's actually **composition + abstraction**, while the notification implementations themselves use polymorphism.

So composition doesn't necessarily mean we have to manually write:

```ts
if (type === "email") ...
else if (type === "sms") ...
```

That would indeed be a poor composition design.

Instead, we can **inject the appropriate implementation**.

---

### The deeper lesson

This is the important part I want you to take away from today's topic:

**Inheritance and composition are not necessarily mutually exclusive.**

A good LLD can use both:

```text
                    Notification
                         ↑
              ┌──────────┼──────────┐
              │          │          │
            Email       SMS        Push
              │          │          │
              └──────────┼──────────┘
                         │
                  NotificationService
                         │
                  HAS-A Notification
```

So:

* `Email IS-A Notification` → **Inheritance / polymorphism**
* `NotificationService HAS-A Notification` → **Composition**
* `NotificationService` depends on `Notification` → **Abstraction**

This is actually a very good example of how the concepts we've learned so far start fitting together.

And your instinct that **we shouldn't introduce composition just for the sake of avoiding inheritance** is correct. 👍


A concise interview answer would be:

> **Email, SMS, and Push are different types of notifications, so I would model them using a common `Notification` abstraction with polymorphic implementations. `NotificationService` can then use composition by depending on the `Notification` interface rather than a concrete implementation. This gives us both a natural IS-A relationship and the flexibility of composition.**

In short:

```text
Email/SMS/Push IS-A Notification
        → Polymorphism / inheritance

NotificationService HAS-A Notification
        → Composition

Notification interface
        → Abstraction
```

That's the **best answer** for this particular example.
