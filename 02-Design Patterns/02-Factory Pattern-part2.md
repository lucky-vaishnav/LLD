# 🧠 Design Patterns — Factory Pattern, Part 2

## Simple Factory vs Factory Method

You already understand the basic Factory idea. Now let's understand **Factory Method**, because interviewers often ask:

> "What's the difference between Factory and Factory Method?"

---

# 1. Simple Factory — what we learned yesterday

We had:

```text
NotificationFactory
        ↓
   create(type)
        ↓
 ┌──────┼──────┐
Email   SMS   Push
```

Example:

```ts
class NotificationFactory {
  static create(type: string): Notification {
    switch (type) {
      case "EMAIL":
        return new EmailNotification();

      case "SMS":
        return new SmsNotification();

      case "PUSH":
        return new PushNotification();

      default:
        throw new Error("Unsupported notification type");
    }
  }
}
```

The Factory itself contains the selection logic.

```text
Factory
 ├── knows Email
 ├── knows SMS
 └── knows Push
```

This is commonly called a **Simple Factory**.

> Note: "Simple Factory" is not one of the original GoF 23 patterns. It's a common design technique.

---

# 2. The problem with the Simple Factory

Suppose we keep adding:

```text
Email
SMS
Push
WhatsApp
Slack
Teams
Telegram
...
```

Our Factory becomes:

```ts
switch (type) {
  case "EMAIL":
  case "SMS":
  case "PUSH":
  case "WHATSAPP":
  case "SLACK":
  case "TEAMS":
  // ...
}
```

The Factory itself becomes increasingly responsible for knowing every concrete implementation.

So we ask:

> **Can we move the creation decision to the subclasses themselves?**

That's where **Factory Method** comes in.

---

# 3. Factory Method — core idea

The key idea:

> **Define a method for creating an object, but let subclasses decide which concrete object to create.**

Instead of:

```text
Factory
 ├── creates Email
 ├── creates SMS
 └── creates Push
```

we get:

```text
NotificationCreator
        ↑
 ┌──────┼──────────┐
 │      │          │
EmailCreator  SmsCreator  PushCreator
 │              │          │
creates         creates    creates
Email           SMS        Push
```

The creation logic is distributed to the appropriate subclass.

---

# 4. Example

First, our product abstraction:

```ts
interface Notification {
  send(message: string): void;
}
```

Implementations:

```ts
class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`Email: ${message}`);
  }
}

class SmsNotification implements Notification {
  send(message: string): void {
    console.log(`SMS: ${message}`);
  }
}

class PushNotification implements Notification {
  send(message: string): void {
    console.log(`Push: ${message}`);
  }
}
```

Now create the **Creator**:

```ts
abstract class NotificationCreator {
  abstract createNotification(): Notification;

  send(message: string): void {
    const notification = this.createNotification();

    notification.send(message);
  }
}
```

Notice something interesting:

`NotificationCreator` knows:

```text
"I need a Notification."
```

but doesn't know:

```text
Email
SMS
Push
```

---

# 5. Concrete creators

```ts
class EmailNotificationCreator extends NotificationCreator {
  createNotification(): Notification {
    return new EmailNotification();
  }
}
```

```ts
class SmsNotificationCreator extends NotificationCreator {
  createNotification(): Notification {
    return new SmsNotification();
  }
}
```

```ts
class PushNotificationCreator extends NotificationCreator {
  createNotification(): Notification {
    return new PushNotification();
  }
}
```

Now:

```text
NotificationCreator
        ↑
 ┌──────┼──────────┐
 │      │          │
Email  SMS        Push
Creator Creator   Creator
 │      │          │
 ↓      ↓          ↓
Email  SMS        Push
```

---

# 6. The important part

Look at this:

```ts
abstract class NotificationCreator {
  abstract createNotification(): Notification;

  send(message: string): void {
    const notification = this.createNotification();
    notification.send(message);
  }
}
```

The base class controls the **workflow**:

```text
create
 ↓
send
```

But it delegates the **creation decision**:

```text
createNotification()
```

to subclasses.

This is the heart of Factory Method.

---

# 7. Simple Factory vs Factory Method

This is the most important comparison.

| Simple Factory                     | Factory Method                                |
| ---------------------------------- | --------------------------------------------- |
| Separate factory object            | Creation method belongs to creator hierarchy  |
| Usually uses `switch`/conditions   | Uses polymorphism/subclassing                 |
| Factory knows concrete types       | Concrete creators decide what to create       |
| Simpler                            | More extensible/flexible                      |
| Good for straightforward selection | Useful when creation varies across subclasses |

### Simple Factory

```text
Client
  ↓
Factory
  ↓
Email / SMS / Push
```

### Factory Method

```text
Client
  ↓
Creator
  ↓
createNotification()
       ↑
 ┌─────┼─────┐
Email  SMS   Push
Creator Creator Creator
```

---

# 8. Why is it called "Factory Method"?

Because:

```ts
createNotification()
```

is the **factory method**.

It's a method whose purpose is to create the product.

The interesting part is that the method is defined at the abstraction/base level:

```ts
abstract createNotification(): Notification;
```

but the concrete implementation is supplied by subclasses.

That's where **polymorphism** comes into play.

---

# 9. Connection with OCP

Suppose we add:

```text
WhatsAppNotification
```

With Simple Factory:

```text
NotificationFactory
       ↓
modify switch
```

With Factory Method:

```text
WhatsAppNotificationCreator
       ↓
createNotification()
       ↓
WhatsAppNotification
```

The existing creators don't need to change.

This gives us a stronger application of **OCP**.

---

# 10. But there's a trade-off

Factory Method introduces more classes.

For three notification types:

```text
Simple Factory:

NotificationFactory
Email
SMS
Push
```

Factory Method:

```text
NotificationCreator
EmailCreator
SmsCreator
PushCreator

Email
SMS
Push
```

That's significantly more structure.

So don't automatically think:

> Factory Method is better than Simple Factory.

Instead:

> **Use the simplest design that handles the required variation.**

For a small application:

**Simple Factory may be enough.**

For a framework/library or a system where different subclasses need different creation behavior:

**Factory Method may be more appropriate.**

---

# 🧠 Very important interview distinction

If asked:

> "Does Factory Method mean there is always an abstract class?"

No.

Factory Method can be implemented using an interface/base abstraction depending on the design.

The important concept is:

> **Object creation is delegated to an overridable/implementable factory method, allowing different implementations to decide what concrete product gets created.**

---

# 11. Connect it to something you already know

Remember your earlier answer:

> "For different account types, we can use inheritance if their behavior is genuinely different."

Factory Method also uses **polymorphism**, but here we're specifically using polymorphism for **object creation**.

That's the connection:

```text
Polymorphism
     ↓
different subclass behavior
     ↓
Factory Method
     ↓
different subclass chooses what to create
```

---

```ts
interface Notification {
  send(message: string): void;
}

class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`Sending Email: ${message}`);
  }
}

class SmsNotification implements Notification {
  send(message: string): void {
    console.log(`Sending SMS: ${message}`);
  }
}

class PushNotification implements Notification {
  send(message: string): void {
    console.log(`Sending Push: ${message}`);
  }
}


// Creator
abstract class NotificationCreator {
  // Factory Method
  abstract createNotification(): Notification;

  // Common workflow
  sendNotification(message: string): void {
    const notification = this.createNotification();

    notification.send(message);
  }
}


// Concrete Creators
class EmailNotificationCreator extends NotificationCreator {
  createNotification(): Notification {
    return new EmailNotification();
  }
}

class SmsNotificationCreator extends NotificationCreator {
  createNotification(): Notification {
    return new SmsNotification();
  }
}

class PushNotificationCreator extends NotificationCreator {
  createNotification(): Notification {
    return new PushNotification();
  }
}


// Actual usage

const emailCreator = new EmailNotificationCreator();

emailCreator.sendNotification("Welcome!");
// Sending Email: Welcome!


const smsCreator = new SmsNotificationCreator();

smsCreator.sendNotification("Your OTP is 1234");
// Sending SMS: Your OTP is 1234


const pushCreator = new PushNotificationCreator();

pushCreator.sendNotification("You have a new message");
// Sending Push: You have a new message
```

### The flow

For Email:

```text
emailCreator
     ↓
EmailNotificationCreator
     ↓
createNotification()
     ↓
EmailNotification
     ↓
send()
```

For SMS:

```text
smsCreator
     ↓
SmsNotificationCreator
     ↓
createNotification()
     ↓
SmsNotification
     ↓
send()
```

The important part is that the **common workflow lives in the parent**:

```ts
sendNotification()
```

while the **creation decision is delegated to the subclass**:

```ts
createNotification()
```

So when we call:

```ts
emailCreator.sendNotification(...)
```

the parent method executes:

```ts
const notification = this.createNotification();
```

but because of **polymorphism**, the actual implementation that runs is:

```ts
EmailNotificationCreator.createNotification()
```

That's the key idea behind **Factory Method**.


---
# 🎯 Interview question for you

Let's test whether you've actually understood the difference.

Imagine we have:

```text
PaymentProcessor
     ↑
 ┌───┼────┐
Card UPI Wallet
```

We currently have:

```ts
PaymentFactory.create(type)
```

with a `switch`.

Now imagine the business says:

> "Different payment providers are going to be implemented as independent modules by different teams. Each module should control how its payment processor is created, and the core payment workflow should remain unchanged."

### Your questions:

**Q1.** Would you prefer Simple Factory or Factory Method here? Why?

**Q2.** In Factory Method, what would be the **Product**?

**Q3.** What would be the **Creator**?

**Q4.** What would the concrete creators be?

**Q5.** Where is polymorphism being used?

Don't worry about code yet. Explain it in your own words, and I'll review it like we've been doing.

--------
Yes. Your answer to **Q1 is correct**. Because different teams/modules independently provide payment implementations and the core workflow should remain unchanged, **Factory Method** is a good fit.

---

## Q1. Simple Factory or Factory Method?

### Interview answer

> I would prefer the **Factory Method pattern** because different payment providers can be developed independently, and each implementation can control how its payment processor is created. The core payment workflow can remain unchanged while new providers can be added through new concrete creators.

### Short notes

```text
Independent implementations
        +
Creation logic varies
        +
Core workflow should remain stable
        ↓
Factory Method
```

---

## Q2. What is the Product?

The **Product** is the abstraction/interface representing the object that the Factory Method creates.

```ts
interface PaymentProcessor {
  pay(amount: number): void;
}
```

So:

```text
PaymentProcessor
      ↑
 ┌────┼────────┐
Card  UPI     Wallet
```

### Interview answer

> `PaymentProcessor` is the Product abstraction. Concrete products such as `CardPaymentProcessor`, `UpiPaymentProcessor`, and `WalletPaymentProcessor` implement it.

### Notes

```text
Product
→ PaymentProcessor

Concrete Products
→ CardPaymentProcessor
→ UpiPaymentProcessor
→ WalletPaymentProcessor
```

---

## Q3. What is the Creator?

The **Creator** is the abstraction that declares the Factory Method.

For example:

```ts
abstract class PaymentProcessorCreator {
  abstract createProcessor(): PaymentProcessor;

  processPayment(amount: number): void {
    const processor = this.createProcessor();
    processor.pay(amount);
  }
}
```

### Interview answer

> `PaymentProcessorCreator` is the Creator. It defines the Factory Method for creating a `PaymentProcessor` and can also contain the common payment workflow.

### Notes

```text
Creator
→ PaymentProcessorCreator

Factory Method
→ createProcessor()
```

---

## Q4. What are the Concrete Creators?

They are the subclasses that implement the Factory Method and decide which concrete Product to create.

```ts
class CardPaymentCreator extends PaymentProcessorCreator {
  createProcessor(): PaymentProcessor {
    return new CardPaymentProcessor();
  }
}

class UpiPaymentCreator extends PaymentProcessorCreator {
  createProcessor(): PaymentProcessor {
    return new UpiPaymentProcessor();
  }
}

class WalletPaymentCreator extends PaymentProcessorCreator {
  createProcessor(): PaymentProcessor {
    return new WalletPaymentProcessor();
  }
}
```

### Interview answer

> Concrete creators such as `CardPaymentCreator`, `UpiPaymentCreator`, and `WalletPaymentCreator` implement the Factory Method and return their corresponding concrete payment processor.

### Notes

```text
PaymentProcessorCreator
        ↑
 ┌──────┼─────────┐
Card   UPI      Wallet
Creator Creator  Creator
  ↓      ↓          ↓
Card    UPI       Wallet
Processor Processor Processor
```

---

## Q5. Where is polymorphism being used?

This is the **most important question**.

There are actually **two levels of polymorphism** here.

### 1. Product polymorphism

```ts
const processor: PaymentProcessor =
  creator.createProcessor();
```

The variable is of type:

```text
PaymentProcessor
```

but the actual object can be:

```text
CardPaymentProcessor
UpiPaymentProcessor
WalletPaymentProcessor
```

---

### 2. Factory Method polymorphism

The Creator has:

```ts
abstract createProcessor(): PaymentProcessor;
```

Different subclasses implement it differently:

```text
CardPaymentCreator
       ↓
creates CardPaymentProcessor

UpiPaymentCreator
       ↓
creates UpiPaymentProcessor

WalletPaymentCreator
       ↓
creates WalletPaymentProcessor
```

### Interview answer

> Polymorphism is used in the Factory Method itself because the base Creator defines `createProcessor()`, while each concrete Creator provides a different implementation. It is also used through the `PaymentProcessor` abstraction, allowing the workflow to work with different concrete processors without knowing their types.

---

# ⭐ One combined interview answer

If an interviewer asks:

> **"Explain how Factory Method works in your payment system."**

You can say:

> "I would define `PaymentProcessor` as the Product abstraction, with concrete implementations such as Card, UPI, and Wallet processors. `PaymentProcessorCreator` would be the Creator and would define a Factory Method such as `createProcessor()`, along with any common payment workflow. Concrete creators like `CardPaymentCreator` and `UpiPaymentCreator` would override that method and create their respective processors. This uses polymorphism both for selecting the concrete creator and for working with the common `PaymentProcessor` abstraction. This design allows independent payment modules to provide their own creation logic while keeping the core workflow unchanged."

That's a **very good senior-level Factory Method explanation**.

### 🧠 Remember this structure

```text
FACTORY METHOD

Creator
  │
  ├── Factory Method
  │       ↓
  │   Product
  │
  └── Common Workflow
          ↑
     Concrete Creators
          │
          ↓
    Concrete Products
```

And the one-line definition for your notes:

> **Factory Method defines an object-creation method in a Creator abstraction and lets subclasses decide which concrete Product to create.**
---
### Question - 
still seems like coder or developer still sending which object to use according to notification type so i think its still dependent on input , caller don't know what notification is send is false statement then from you.. so this pattern don't give 100% abstraction , so what is the actual use case of factory method just for code structure , good code structure or if multiple teams working on multiple implementation then only i think?

---
Yes — **your criticism is correct**, and I want to correct the earlier statement clearly.

I overstated the idea that Factory Method means *"the caller doesn't need to know which notification to use."* **That is not the defining benefit of Factory Method.**

### The important distinction

Factory Method does **not** necessarily hide the *business decision* of which product is needed.

It hides/delegates the **creation mechanism**.

So if the application receives:

```text
notificationType = EMAIL
```

some part of the application still has to decide:

> "This request requires Email."

Factory Method doesn't magically remove that decision.

---

# So what is the actual use case?

The real problem Factory Method solves is:

> **A base workflow is stable, but the concrete object used by that workflow needs to vary. We want subclasses to control that creation without changing the base workflow.**

For example, imagine a document-processing system:

```text
DocumentProcessor
        │
        └── process()
              │
              ├── validate()
              ├── createParser()   ← Factory Method
              └── parse()
```

Different processors:

```text
PDFProcessor
    ↓
creates PDFParser

WordProcessor
    ↓
creates WordParser

ExcelProcessor
    ↓
creates ExcelParser
```

The **workflow** is common:

```text
validate
   ↓
create parser
   ↓
parse
```

But the parser varies.

That is where Factory Method becomes useful.

---

# And yes — it is partly about structure

You said:

> "So this pattern don't give 100% abstraction, so is the actual use case just good code structure?"

**Yes, partly.**

Design patterns are not magic abstraction mechanisms.

Factory Method gives you:

### 1. Separation of creation from workflow

```text
Base Creator
   ↓
common workflow

Subclass
   ↓
creation decision
```

### 2. Polymorphic creation

The base workflow can call:

```ts
this.createProduct();
```

without knowing which concrete product will be created.

### 3. Easier extension

A new product can often be introduced through a new creator subclass without modifying the common workflow.

### 4. Framework/library extensibility

This is one of the **strongest real-world use cases**.

A framework can define:

```ts
abstract class Framework {
  abstract createConnection(): Connection;

  start() {
    const connection = this.createConnection();
    connection.connect();
    // common framework logic...
  }
}
```

Different implementations provide:

```text
PostgresFramework
    ↓
PostgresConnection

MongoFramework
    ↓
MongoConnection
```

The framework controls the workflow, while the implementation controls what gets created.

---

# What Factory Method does NOT solve

It does **not** necessarily solve:

> "The system receives `EMAIL` and automatically knows everything without anyone deciding."

That's a **different problem**.

For that, you might use:

```text
Factory
Registry
Dependency Injection container
Configuration
Strategy
Routing
```

depending on the architecture.

---

# And your "multiple teams" point

That's **one possible reason**, but not the definition.

Multiple teams/modules can benefit because each module can provide its own Creator:

```text
Core framework
      ↓
NotificationCreator
      ↑
 ┌────┼─────────┐
Team A   Team B   Team C
  ↓        ↓        ↓
Email     SMS      Push
```

But you **don't need multiple teams** to justify Factory Method.

It can be useful even in one team if:

```text
common workflow
       +
varying product creation
       +
polymorphic creators
```

exists.

---

# 🧠 The easiest way to remember it

Don't think:

> **Factory Method = hide which object the caller wants.** ❌

Think:

> **Factory Method = let subclasses decide how/which product to create while the parent controls the common workflow.** ✅

And compare:

```text
Simple Factory
→ "Give me an object based on this type."

Factory Method
→ "I have a common workflow, but subclasses decide which object that workflow creates."
```

---

## One more important point for your LLD learning

Your instinct here is **exactly what I want you to develop**.

You didn't just accept:

> "Factory Method = abstraction = good."

You challenged:

> "But the developer still has to decide the type, so what problem did we actually solve?"

That's the right LLD mindset.

**We should not use Factory Method for our Notification example just to make the design look sophisticated.** For that simple `EMAIL/SMS/PUSH` selection problem, a Simple Factory or another selection mechanism is likely more appropriate.

So I would **not consider the Notification example a good primary example for teaching Factory Method**. The Document Processor / Parser example is much better for understanding the actual pattern.

