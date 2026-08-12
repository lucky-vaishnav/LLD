## 🆕 LLD Topic #3: Abstraction

Abstraction is especially important for LLD because it leads directly into **interfaces, dependency inversion, loose coupling, and design patterns**.

---

## 1. What is Abstraction?

**Abstraction means exposing only the essential behavior of an object while hiding the implementation details.**

The key question is:

> **What does the caller need to know, and what can remain hidden?**

For example, when you use a payment system:

```ts
paymentService.pay(1000);
```

The caller doesn't need to know whether internally it uses:

```text
PaymentService
      ↓
 ┌───────────────┐
 │ Payment logic │
 └───────────────┘
      ↓
 ┌───────────────┐
 │ API call      │
 │ authentication│
 │ retries       │
 │ logging       │
 │ response      │
 └───────────────┘
```

The caller only cares:

> **"Make a payment."**

That's abstraction.

---

# 2. Real-world example

Think about driving a car.

You interact with:

```text
steering wheel
accelerator
brake
gear
```

You don't need to understand:

```text
fuel injection
engine combustion
transmission internals
ECU logic
```

The car exposes the **essential interface** while hiding the implementation.

That's abstraction.

---

# 3. Abstraction in code

Suppose we have:

```ts
class PaymentService {
  pay(amount: number) {
    // validate payment
    // create request
    // call payment gateway
    // handle response
    // retry if necessary
  }
}
```

The consumer simply does:

```ts
paymentService.pay(1000);
```

The consumer doesn't need to know how `pay()` works internally.

So:

```text
Consumer
   │
   │ pay()
   ↓
PaymentService
   │
   ├── validation
   ├── API request
   ├── retry
   ├── logging
   └── response handling
```

The implementation is hidden behind the abstraction.

---

# 4. Where abstraction becomes REALLY important in LLD

Now imagine our system supports multiple payment providers:

```text
CyberSource
Braintree
Stripe
PayPal
```

A bad design might be:

```ts
class PaymentService {
  pay(provider: string, amount: number) {
    if (provider === "stripe") {
      // Stripe implementation
    } else if (provider === "braintree") {
      // Braintree implementation
    } else if (provider === "cybersource") {
      // CyberSource implementation
    }
  }
}
```

This creates tight coupling.

Now let's introduce an abstraction:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

Then implementations:

```ts
class StripeProcessor implements PaymentProcessor {
  pay(amount: number): PaymentResult {
    // Stripe-specific implementation
  }
}

class BraintreeProcessor implements PaymentProcessor {
  pay(amount: number): PaymentResult {
    // Braintree-specific implementation
  }
}

class CyberSourceProcessor implements PaymentProcessor {
  pay(amount: number): PaymentResult {
    // CyberSource-specific implementation
  }
}
```

Now the higher-level service can depend on the abstraction:

```ts
class PaymentService {
  constructor(private processor: PaymentProcessor) {}

  pay(amount: number) {
    return this.processor.pay(amount);
  }
}
```

And:

```ts
const processor = new StripeProcessor();

const paymentService = new PaymentService(processor);

paymentService.pay(1000);
```

The important thing is:

**`PaymentService` doesn't care which payment provider it is using.**

It only knows:

```text
PaymentProcessor
      │
      └── pay()
```

That's abstraction.

---

# 5. Abstraction vs Encapsulation

This is a **very common interview question**.

### Encapsulation

> **How do we protect and control the internal state?**

Example:

```ts
class BankAccount {
  private balance = 0;

  deposit(amount: number) {
    // controlled state change
  }
}
```

We're protecting `balance`.

---

### Abstraction

> **How do we hide implementation complexity and expose only what consumers need?**

Example:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

The consumer doesn't care whether the implementation is Stripe, Braintree, or CyberSource.

---

### Easy way to remember

Think:

```text
Encapsulation → Protect the internals
Abstraction   → Hide the complexity
```

Or:

> **Encapsulation is about controlling access.
> Abstraction is about controlling what is exposed.**

They're related, but they're not the same thing.

---

# 6. Abstraction doesn't necessarily mean interfaces

Another important interview point.

You might hear:

> "Abstraction means using interfaces."

Not exactly.

Interfaces are **one mechanism** for achieving abstraction.

You can also achieve abstraction through:

* abstract classes
* public methods hiding internal implementation
* modules/APIs
* well-designed classes

For example:

```ts
class CoffeeMachine {
  makeCoffee() {
    this.grindBeans();
    this.boilWater();
    this.brew();
  }

  private grindBeans() {}
  private boilWater() {}
  private brew() {}
}
```

The user sees:

```ts
machine.makeCoffee();
```

They don't need to know the internal steps.

That's abstraction even without an interface.

---

# 7. The LLD mindset

Whenever you're designing a class, ask:

### What should the consumer know?

```text
PaymentProcessor
    ↓
pay()
refund()
```

### What should the consumer NOT need to know?

```text
authentication
HTTP calls
request formatting
retry mechanism
provider-specific errors
logging
```

This separation is extremely useful for designing maintainable systems.

---

# 8. Why abstraction helps us

Good abstraction gives us:

**Loose coupling**

```text
PaymentService
      ↓
PaymentProcessor
      ↓
Stripe / Braintree / CyberSource
```

instead of:

```text
PaymentService
      ↓
Stripe implementation
      ↓
Braintree implementation
      ↓
CyberSource implementation
```

It also makes the system easier to:

* extend
* test
* replace implementations
* maintain
* reason about

And this is where abstraction starts connecting with **SOLID**, particularly the **Dependency Inversion Principle**.

We'll get there later.

---

## 🎯 Today's key takeaway

Write this in your notes:

> **Abstraction means exposing the essential behavior while hiding unnecessary implementation details.**

And remember:

```text
Encapsulation
→ Protect internal state
→ Control how state changes

Abstraction
→ Hide implementation complexity
→ Expose essential behavior
```

### One interview question to think about

Suppose we have:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

and:

```ts
class StripeProcessor implements PaymentProcessor {}
class BraintreeProcessor implements PaymentProcessor {}
```

**Why is this better than simply creating one `PaymentService` class containing all Stripe and Braintree logic?**

Think about that question. It will prepare you for the next major step: **Inheritance vs Composition**, and eventually **SOLID principles**.

