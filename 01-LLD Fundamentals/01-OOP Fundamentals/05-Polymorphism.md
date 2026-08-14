Since we've covered **Class/Object → Encapsulation → Abstraction → Inheritance vs Composition**, the next topic should be another core OOP concept before we move into SOLID.

## 🆕 LLD Topic #5: Polymorphism

Polymorphism is extremely important in LLD because it is what allows the abstraction/composition designs we've discussed to actually work cleanly.

---

# 1. What is Polymorphism?

The word means:

> **"One interface, multiple implementations."**

In practical terms:

> We can interact with different objects through the same interface, while each object provides its own implementation.

Our notification example is perfect.

```ts
interface Notification {
  send(message: string): void;
}
```

Different implementations:

```ts
class EmailNotification implements Notification {
  send(message: string) {
    console.log("Sending Email:", message);
  }
}

class SmsNotification implements Notification {
  send(message: string) {
    console.log("Sending SMS:", message);
  }
}

class PushNotification implements Notification {
  send(message: string) {
    console.log("Sending Push:", message);
  }
}
```

All three provide:

```text
send()
```

but they implement it differently.

---

# 2. The important part

Now consider:

```ts
function sendNotification(
  notification: Notification,
  message: string
) {
  notification.send(message);
}
```

We can pass any implementation:

```ts
sendNotification(
  new EmailNotification(),
  "Hello"
);

sendNotification(
  new SmsNotification(),
  "Hello"
);

sendNotification(
  new PushNotification(),
  "Hello"
);
```

The function doesn't need:

```ts
if (type === "email") {
   // ...
} else if (type === "sms") {
   // ...
} else if (type === "push") {
   // ...
}
```

Instead, it simply says:

```text
"Whatever object you give me,
if you satisfy Notification,
I'll call send()."
```

**That is polymorphism.**

---

# 3. Why is this powerful in LLD?

Imagine tomorrow we add:

```text
WhatsAppNotification
SlackNotification
TeamsNotification
```

We implement:

```ts
class WhatsAppNotification implements Notification {
  send(message: string) {
    // WhatsApp implementation
  }
}
```

We don't need to change:

```ts
sendNotification()
```

That's a major benefit.

The code that **uses** the abstraction doesn't need to know about every implementation.

---

# 4. Without polymorphism

Imagine this:

```ts
function sendNotification(
  type: string,
  message: string
) {
  if (type === "email") {
    // Email logic
  } else if (type === "sms") {
    // SMS logic
  } else if (type === "push") {
    // Push logic
  }
}
```

Every new notification type requires modifying this function.

Over time:

```text
Email
SMS
Push
WhatsApp
Slack
Teams
...
```

The conditional logic keeps growing.

This is exactly the kind of design we want to avoid in LLD.

---

# 5. Polymorphism + Abstraction

These two concepts work closely together.

### Abstraction

Defines **what** an object can do:

```ts
interface Notification {
  send(message: string): void;
}
```

### Polymorphism

Allows different objects to provide **different implementations** of that behavior:

```text
Notification
    │
    ├── Email → send()
    ├── SMS   → send()
    └── Push  → send()
```

So:

> **Abstraction gives us the common contract.
> Polymorphism lets different implementations fulfill that contract.**

This is a very useful interview distinction.

---

# 6. Polymorphism + Composition

Remember our previous discussion:

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

Now:

```ts
const service = new NotificationService(
  new EmailNotification()
);
```

or:

```ts
const service = new NotificationService(
  new SmsNotification()
);
```

or:

```ts
const service = new NotificationService(
  new PushNotification()
);
```

The `NotificationService` remains unchanged.

This gives us:

```text
Abstraction
     ↓
Notification interface
     ↓
Polymorphism
     ↓
Email / SMS / Push
     ↓
Composition
     ↓
NotificationService uses Notification
```

This combination appears **everywhere in real LLD designs**.

---

# 7. Runtime polymorphism

The example we've been discussing is generally called **runtime polymorphism**.

The exact implementation that gets executed depends on the actual object provided at runtime.

For example:

```ts
let notification: Notification;

notification = new EmailNotification();

notification.send("Hello");
```

Email implementation runs.

Later:

```ts
notification = new SmsNotification();

notification.send("Hello");
```

SMS implementation runs.

The variable is still:

```ts
Notification
```

but the actual object is different.

That's the important idea.

---

# 8. Another real LLD example: Payment

This is perhaps even closer to your backend work.

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

Implementations:

```text
PaymentProcessor
       │
       ├── StripeProcessor
       ├── BraintreeProcessor
       └── CyberSourceProcessor
```

Then:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}

  process(amount: number) {
    return this.processor.pay(amount);
  }
}
```

`PaymentService` doesn't care which provider is being used.

That's polymorphism in a practical production-style design.

---

# 9. A common interview mistake

Don't define polymorphism simply as:

> "Same method with different behavior."

That's not wrong, but it's incomplete.

A better senior-level answer:

> **Polymorphism allows objects of different concrete types to be treated through a common abstraction while each type provides its own implementation of the behavior.**

And then give the `PaymentProcessor` or `Notification` example.

That's much stronger.

---

# 10. Four OOP concepts we've covered

You should now start seeing how these concepts connect:

```text
              OOP
               │
     ┌─────────┼─────────┐
     │         │         │
Encapsulation Abstraction Polymorphism
     │         │         │
Protect      Hide       Same contract,
state        details    different behavior
               │
               ↓
          Composition
       uses abstractions
```

And inheritance can participate in polymorphism, but **polymorphism doesn't require class inheritance**. TypeScript interfaces + implementations are enough.

---

## 🎯 Today's key takeaway

Remember this:

> **Polymorphism = same contract/interface, different implementations.**

For example:

```text
Notification
   │
   ├── Email → send() → Email behavior
   ├── SMS   → send() → SMS behavior
   └── Push  → send() → Push behavior
```

And the big LLD benefit:

> **The code that uses the abstraction doesn't need to know which concrete implementation it is dealing with.**

### One interview question for you

Suppose we have:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

and:

```ts
class StripeProcessor implements PaymentProcessor {}
class CyberSourceProcessor implements PaymentProcessor {}
```

If `PaymentService` receives a `PaymentProcessor` in its constructor, **how does this design help us add a new `RazorpayProcessor` later without modifying `PaymentService`?**

--
### My Thoughts-
So, yes, if payment service receive a payment processor in its constructor, so how does this design help us to add new payment processor next time in the payment service. So, basically we have because payment service is using abstraction via payment processor, in the payment service constructor, we are using payment processor. And this payment processor constructor value can be changed at runtime, like say, for the processor, at implementation more any change. When we will be calling the payment service, we will be using the razorpay processor object and payment service will receive a constructor because this constructor payment processor is the abstracted interface.

---
Yes. **Your core understanding is correct.** 👍

You're describing **dependency injection + abstraction + polymorphism** working together.

The only correction is that the **constructor itself doesn't change at runtime**. What changes is the **concrete implementation/object that we inject into the constructor**.

### Your interview answer

You could say:

> "PaymentService depends on the PaymentProcessor abstraction rather than a concrete payment provider. When we introduce Razorpay, we only need to implement the PaymentProcessor interface in RazorpayProcessor and inject that implementation. PaymentService doesn't need to change because it already works with the common abstraction. This gives us loose coupling and polymorphism."

That's a **very good LLD answer**.

And you've now connected several concepts we've learned:

```text
Abstraction
    ↓
PaymentProcessor interface
    ↓
Polymorphism
    ↓
Stripe / CyberSource / Razorpay
    ↓
Dependency Injection
    ↓
PaymentService
    ↓
Loose Coupling
```
hink about the answer rather than just memorizing it. This will reinforce today's concept.

