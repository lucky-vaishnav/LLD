# 🚀 Design Patterns — Topic 1

## Factory Pattern

This is a good first pattern for you because you've already encountered the exact problem in our **Payment System** and **Food Ordering System** exercises.

You don't need to learn it as an isolated definition—we'll connect it to those problems.

---

# 1. The problem Factory solves

Imagine our payment system:

```text
PaymentService
      ↓
Which processor?
      ↓
UPI
Credit Card
Wallet
Net Banking
```

Suppose we write:

```ts
class PaymentService {
  process(type: string, amount: number) {
    if (type === "UPI") {
      const processor = new UpiPaymentProcessor();
      processor.pay(amount);
    } else if (type === "CARD") {
      const processor = new CardPaymentProcessor();
      processor.pay(amount);
    } else if (type === "WALLET") {
      const processor = new WalletPaymentProcessor();
      processor.pay(amount);
    }
  }
}
```

There are problems here.

### Problem 1 — Tight coupling

`PaymentService` knows about:

```text
UpiPaymentProcessor
CardPaymentProcessor
WalletPaymentProcessor
```

### Problem 2 — OCP violation

Every time we add:

```text
NetBanking
```

we modify `PaymentService`.

### Problem 3 — Object creation and business logic are mixed

`PaymentService` is doing two different things:

```text
1. Decide which implementation
2. Process payment
```

That's where Factory can help.

---

# 2. What is Factory Pattern?

The basic idea:

> **Move object-creation/selection logic into a separate component.**

Instead of:

```text
PaymentService
    ↓
creates UPI/Card/Wallet
```

we introduce:

```text
PaymentService
      ↓
PaymentFactory
      ↓
creates appropriate processor
```

So:

```text
                PaymentService
                      ↓
                PaymentFactory
                      ↓
              PaymentProcessor
                 ↑    ↑    ↑
                UPI Card Wallet
```

---

# 3. First define the abstraction

```ts
interface PaymentProcessor {
  pay(amount: number): void;
}
```

Implementations:

```ts
class UpiPaymentProcessor implements PaymentProcessor {
  pay(amount: number): void {
    console.log(`Paid ${amount} using UPI`);
  }
}

class CardPaymentProcessor implements PaymentProcessor {
  pay(amount: number): void {
    console.log(`Paid ${amount} using Card`);
  }
}

class WalletPaymentProcessor implements PaymentProcessor {
  pay(amount: number): void {
    console.log(`Paid ${amount} using Wallet`);
  }
}
```

---

# 4. Create the Factory

```ts
class PaymentProcessorFactory {
  static create(type: string): PaymentProcessor {
    switch (type) {
      case "UPI":
        return new UpiPaymentProcessor();

      case "CARD":
        return new CardPaymentProcessor();

      case "WALLET":
        return new WalletPaymentProcessor();

      default:
        throw new Error("Unsupported payment type");
    }
  }
}
```

Now the creation logic is centralized.

---

# 5. PaymentService becomes simpler

```ts
class PaymentService {
  process(type: string, amount: number): void {
    const processor =
      PaymentProcessorFactory.create(type);

    processor.pay(amount);
  }
}
```

Now:

```text
PaymentService
      ↓
"I need a payment processor"
      ↓
Factory
      ↓
UPI / Card / Wallet
```

`PaymentService` doesn't need to know how each processor is created.

---

# 6. What exactly did Factory improve?

Before:

```text
PaymentService
 ├── knows UPI
 ├── knows Card
 ├── knows Wallet
 └── creates them
```

After:

```text
PaymentService
      ↓
PaymentFactory
      ↓
creates implementation
```

So we separate:

> **Business operation** from **object creation/selection**.

That's the core idea.

---

# 7. Factory vs Dependency Injection

This is very important because we've already learned DI.

### Dependency Injection

Someone **outside** provides the dependency.

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

The service doesn't create it.

---

### Factory

The Factory **creates/selects** the appropriate implementation.

```ts
const processor =
  PaymentProcessorFactory.create("UPI");
```

So:

```text
DI
→ "Give me the dependency."

Factory
→ "I'll create/select the appropriate dependency."
```

They can also work together.

---

# 8. Factory does NOT mean every `new` needs a Factory

This is very important.

Don't do:

```text
new Customer()
       ↓
CustomerFactory
       ↓
new Customer()
```

just because you've learned Factory.

If object creation is simple and there's no meaningful variation:

```ts
const customer = new Customer();
```

is perfectly fine.

Factory becomes useful when:

* creation logic is complex
* there are multiple implementations
* the concrete type is determined dynamically
* you want to centralize creation/selection
* callers shouldn't know the concrete class

---

# 9. Factory and OCP

There's a subtle point here.

You might notice:

```ts
switch(type) {
   case "UPI":
   case "CARD":
   case "WALLET":
}
```

If we add Net Banking, the **Factory itself** must change.

So Factory doesn't magically mean:

> "No code ever changes."

Instead, we are **isolating the change**.

Without Factory:

```text
PaymentService
   ↓
changes whenever new processor added
```

With Factory:

```text
PaymentService
   ↓
stable

Factory
   ↓
changes when new type is registered
```

That's much better separation.

Later, when we learn more advanced approaches like **Factory Method**, **Abstract Factory**, or registry-based designs, you'll see how this can evolve further.

---

# 10. Factory vs Factory Method

Don't confuse these.

### Simple Factory

What we just did:

```text
PaymentProcessorFactory
        ↓
create(type)
```

It's commonly called **Simple Factory** or **Factory Pattern**, although terminology varies.

### Factory Method

Factory Method is a formal **GoF creational design pattern** with a different structure, where object creation is delegated to subclasses/overridable factory methods.

We'll learn that separately.

For now:

```text
Simple Factory
→ central object creation/selection

Factory Method
→ subclasses decide which object to create
```

Don't worry about implementing Factory Method yet.

---

# 11. Interview answer

If the interviewer asks:

> **"What is the Factory Pattern?"**

A good answer:

> "Factory Pattern encapsulates object creation logic and provides the appropriate implementation based on some input or context. It prevents client code from being tightly coupled to concrete classes. For example, instead of PaymentService directly creating UPI, Card, or Wallet processors, a PaymentFactory can create the appropriate PaymentProcessor implementation."

Then add:

> "It is useful when object creation involves multiple implementations or dynamic selection, but I wouldn't introduce a Factory for simple object creation where there is no meaningful variation."

That's a **senior-level answer** because you explain not only what it is, but **when not to use it**.

---

# 🧠 Connect this with our previous problems

Remember your earlier answer?

You said:

> "Who should decide which implementation to use?"

We identified that as the point where Factory could help.

So now you can connect:

```text
Payment request
      ↓
paymentMethod = UPI
      ↓
💡 Factory
      ↓
UpiPaymentProcessor
```

And in the Food Ordering System:

```text
paymentMethod
      ↓
Factory
      ↓
UPI / Card / Wallet
```

This is exactly why I intentionally mentioned the pattern earlier without teaching it.

---

# 🎯 Your first Factory exercise

Let's test your understanding.

Imagine we have:

```text
Notification
    ↑
 ┌──┼────┐
Email SMS Push
```

and the API receives:

```text
notificationType = "SMS"
```

### Question:

**Without writing code initially**, tell me:

1. Where would you put the object creation/selection logic?
2. What would the Factory return?
3. What should `NotificationService` know about Email/SMS/Push?
4. If tomorrow we add WhatsApp, what part of the design changes?
5. How is this different from the Dependency Injection approach we used earlier?

Answer in your own words. Then I'll review it, and after that we'll do the **code implementation + Factory interview notes**.

---
### My Thought-
Yeah, for this problem, you shared about the notification, so we can follow the same factory pattern for it because we have multiple types of notification, email, SMS, and push. And in the future as well, we can have more notifications. So we can move the object creation logic into the factory pattern, and notification service will only be responsible for the sending the notification with the appropriate object, and the object creation logic will be moved to the factory pattern. And this is how we can achieve the single responsibility principle as well. And also, we can achieve the open/closed principle because factory part can be open for the extension, but notification service will be the same whenever we have the new notification type.

---
Exactly. 👍 Your understanding of **why** we'd introduce the Factory is correct. You connected it to the right SOLID principles rather than just saying "Factory creates objects."

There are a couple of refinements worth adding to your notes.

# 📝 Review

### 1. Factory owns creation/selection — ✅

You said:

> "We can move the object creation logic into the factory."

Correct.

Conceptually:

```text
NotificationService
        ↓
NotificationFactory
        ↓
 ┌──────┼──────┐
Email   SMS   Push
```

The Factory decides:

```text
"Given SMS, which Notification implementation should I create?"
```

---

### 2. NotificationService should not know concrete implementations — ✅

Ideally:

```text
NotificationService
        ↓
    Notification
```

It shouldn't have:

```ts
new EmailNotification()
new SmsNotification()
new PushNotification()
```

Instead:

```text
NotificationService
        ↓
Factory
        ↓
Notification abstraction
```

So the service focuses on the **notification workflow**, not construction.

---

### 3. SRP — ✅, but phrase it carefully

You said this helps achieve SRP.

Correct, but don't say:

> "Factory is responsible for SRP."

Better:

> **Separating object creation from notification processing gives each component a more focused responsibility.**

```text
Factory
→ creation/selection

NotificationService
→ notification workflow

Email/SMS/Push
→ channel-specific sending
```

That's SRP in practice.

---

### 4. OCP — mostly correct, with an important nuance

You said:

> "Factory can be open for extension, but NotificationService will be the same."

The second part is exactly right:

```text
Add WhatsApp
      ↓
NotificationService
      ↓
No change
```

But the Factory itself, with a simple `switch`, **will need modification**:

```ts
switch (type) {
  case "EMAIL":
  case "SMS":
  case "PUSH":
  case "WHATSAPP": // new change
}
```

So don't claim that a basic Factory gives us **perfect OCP**.

Instead:

> **Factory localizes the change to the object-creation layer and keeps the client/business service independent of concrete implementations.**

Later we'll learn approaches that can make registration more extensible.

---

# 5. Factory vs DI — your next important connection

Let's make this distinction very clear.

### DI

```text
Outside component
       ↓
provides dependency
       ↓
NotificationService
```

Example:

```ts
new NotificationService(new SmsNotification());
```

The caller decides the implementation.

---

### Factory

```text
NotificationService
       ↓
Factory
       ↓
"SMS"
       ↓
SmsNotification
```

The Factory decides/creates the implementation based on the required type.

So:

> **DI answers "How does the dependency get into the object?"**

> **Factory answers "Which object should be created?"**

They can absolutely be used together.

---

# 🧠 Your notes

> **Factory Pattern:** Encapsulates object creation/selection logic so client/business code doesn't need to know about concrete implementations.

```text
Client / Service
      ↓
   Factory
      ↓
Concrete implementation
```

### Benefits

* Separates creation from business logic
* Reduces coupling to concrete classes
* Centralizes implementation selection
* Makes adding implementations less invasive
* Works well with abstraction + polymorphism
* Can complement Dependency Injection

### Don't use Factory when:

> Object creation is simple, there's only one implementation, and there's no meaningful creation/selection logic.

---
