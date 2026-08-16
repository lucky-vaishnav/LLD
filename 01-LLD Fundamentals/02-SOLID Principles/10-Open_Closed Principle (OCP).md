## 🆕 LLD Topic #10: Open/Closed Principle (OCP)

The **O** in SOLID.

### 1. What is OCP?

> **Software entities should be open for extension but closed for modification.**

In simpler terms:

> **We should be able to add new behavior without repeatedly modifying existing, stable code.**

This is one of the most important principles for designing extensible LLDs.

---

# 2. A simple example

Suppose we have:

```ts
class PaymentService {
  processPayment(type: string, amount: number) {
    if (type === "stripe") {
      // Stripe logic
    }

    if (type === "cybersource") {
      // CyberSource logic
    }
  }
}
```

Now tomorrow we add Razorpay:

```ts
if (type === "razorpay") {
  // Razorpay logic
}
```

We have to **modify `PaymentService`**.

Then another provider:

```ts
if (type === "paypal") {
  // PayPal logic
}
```

Again, modify it.

This design isn't following OCP well.

---

# 3. Applying abstraction

We already know how to solve this.

Define an abstraction:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

Implementations:

```ts
class StripeProcessor implements PaymentProcessor {
  pay(amount: number) {
    // Stripe implementation
  }
}

class CyberSourceProcessor implements PaymentProcessor {
  pay(amount: number) {
    // CyberSource implementation
  }
}
```

And:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}

  processPayment(amount: number) {
    return this.processor.pay(amount);
  }
}
```

Now adding Razorpay means:

```ts
class RazorpayProcessor implements PaymentProcessor {
  pay(amount: number) {
    // Razorpay implementation
  }
}
```

We **extend the system** without modifying `PaymentService`.

That's the essence of OCP.

---

# 4. "Open for extension"

The system is open for:

```text
Stripe
CyberSource
Razorpay
PayPal
...
```

We can add implementations.

```text id="w4e5kz"
PaymentProcessor
      ↑
 ┌────┼──────────────┐
 │    │              │
Stripe CyberSource Razorpay
```

We are extending the system by adding new classes.

---

# 5. "Closed for modification"

This doesn't literally mean:

> **"Never modify existing code."**

That's an important interview clarification.

It means:

> **Once a component's behavior is stable, adding a new variation shouldn't require repeatedly changing that component.**

For example:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}

  processPayment(amount: number) {
    return this.processor.pay(amount);
  }
}
```

We don't need to touch this class every time we introduce another payment provider.

---

# 6. Another practical example: Discounts

Suppose we have:

```ts
class DiscountCalculator {
  calculate(type: string, amount: number) {
    if (type === "regular") {
      return amount * 0.05;
    }

    if (type === "premium") {
      return amount * 0.10;
    }

    if (type === "corporate") {
      return amount * 0.20;
    }
  }
}
```

Every new discount type requires modifying this class.

Instead:

```ts
interface DiscountStrategy {
  calculate(amount: number): number;
}
```

Implementations:

```ts
class RegularDiscount implements DiscountStrategy {
  calculate(amount: number) {
    return amount * 0.05;
  }
}

class PremiumDiscount implements DiscountStrategy {
  calculate(amount: number) {
    return amount * 0.10;
  }
}

class CorporateDiscount implements DiscountStrategy {
  calculate(amount: number) {
    return amount * 0.20;
  }
}
```

Then:

```ts
class DiscountCalculator {
  constructor(
    private strategy: DiscountStrategy
  ) {}

  calculate(amount: number) {
    return this.strategy.calculate(amount);
  }
}
```

Adding:

```text
FestivalDiscount
EmployeeDiscount
FirstOrderDiscount
```

doesn't require changing `DiscountCalculator`.

This is also the beginning of the **Strategy Pattern**, which we'll study later.

---

# 7. OCP and SRP are connected

Yesterday we learned **SRP**.

They solve different problems.

### SRP asks:

> **Does this class have too many different reasons to change?**

### OCP asks:

> **Can we add new variations without repeatedly modifying stable existing code?**

For example:

```text
PaymentService
```

could have one responsibility:

> Process payments.

That's SRP.

But if every new payment provider requires modifying it:

```text
Stripe → modify
CyberSource → modify
Razorpay → modify
PayPal → modify
```

then we have an extensibility problem.

OCP helps us design:

```text
PaymentService
      ↓
PaymentProcessor
      ↑
 ┌────┼─────┬──────┐
Stripe Cyber Razorpay
```

---

# 8. OCP does NOT mean "never use if/else"

This is another important nuance.

You shouldn't see:

```ts
if (...)
```

and immediately say:

> "OCP violation!"

For example:

```ts
if (user.isAdmin) {
  // ...
}
```

may be completely fine.

OCP becomes particularly valuable when you have **a growing set of variations/implementations**.

For example:

```text
Payment providers
Notification channels
Pricing strategies
Shipping methods
Authentication providers
File storage providers
```

These are good candidates for abstraction + polymorphism.

---

# 9. The key design signal

Whenever you see code like:

```ts
if (type === "A") {
   ...
} else if (type === "B") {
   ...
} else if (type === "C") {
   ...
}
```

don't automatically refactor it.

Instead ask:

> **"Is this list of types expected to grow, and will each type have different behavior?"**

If yes, consider:

```text
Interface
   ↓
Multiple implementations
   ↓
Polymorphism
   ↓
Extension without modifying core logic
```

That's where OCP becomes useful.

---

# 10. Senior-level interview answer

If asked:

> **"What is the Open/Closed Principle?"**

A strong answer:

> "The Open/Closed Principle says that a component should be open for extension but closed for modification. In practice, when I have behavior that is expected to have multiple variations, I use abstractions and polymorphism so that new implementations can be added without repeatedly changing stable business logic. It doesn't mean existing code can never be modified; rather, it reduces the need to modify stable code for every new variation."

That's the level of answer I'd recommend for you.

---

## 🎯 Today's takeaway

Remember:

```text
OCP
 ↓
Open for extension
Closed for modification
 ↓
New behavior
   → Add implementation
   → Avoid modifying stable core logic
```

And the common LLD approach:

```text
Abstraction
     ↓
Polymorphism
     ↓
New implementations
     ↓
Existing code remains unchanged
```

### Progress

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
10. **OCP ← Today**

Next in SOLID is **Liskov Substitution Principle (LSP)**.
