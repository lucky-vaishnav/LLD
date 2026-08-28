# Topic 4 — Decorator Pattern

This is a very useful pattern for senior-level LLD because it teaches you how to **add behavior to an existing object without modifying its original class**.

It also connects strongly with **OCP (Open/Closed Principle)**, which you've already learned.

---

## 1. The problem

Imagine we have a coffee ordering system.

Initially:

```text
Coffee
→ ₹100
```

Now customers can add:

```text
Milk       + ₹20
Sugar      + ₹10
WhippedCream + ₹30
```

A naive implementation could become:

```ts
class Coffee {
  price() {
    return 100;
  }
}
```

Then we might start doing:

```ts
class Coffee {
  constructor(
    private milk: boolean,
    private sugar: boolean,
    private cream: boolean
  ) {}

  price() {
    let price = 100;

    if (this.milk) price += 20;
    if (this.sugar) price += 10;
    if (this.cream) price += 30;

    return price;
  }
}
```

Now imagine 20 different optional additions.

The class becomes increasingly complicated.

And every new option requires modifying `Coffee`.

---

# 2. Decorator's idea

Instead of modifying the original object, we **wrap it**.

```text
Coffee
  ↓
MilkDecorator
  ↓
SugarDecorator
  ↓
CreamDecorator
```

Each decorator adds something to the existing behavior.

For example:

```text
Base Coffee = ₹100

      ↓ add Milk

Milk Coffee = ₹120

      ↓ add Sugar

Milk + Sugar = ₹130

      ↓ add Cream

Milk + Sugar + Cream = ₹160
```

---

# 3. Common abstraction

We define:

```ts
interface Coffee {
  getPrice(): number;
}
```

The important thing is that **both the original object and decorators implement the same interface**.

---

## 4. Base object

```ts
class SimpleCoffee implements Coffee {
  getPrice(): number {
    return 100;
  }
}
```

---

## 5. Base Decorator

```ts
abstract class CoffeeDecorator implements Coffee {
  constructor(
    protected coffee: Coffee
  ) {}

  abstract getPrice(): number;
}
```

Notice something important:

```text
CoffeeDecorator
      ↓
contains Coffee
```

This is **composition**.

---

# 6. Concrete decorators

```ts
class MilkDecorator extends CoffeeDecorator {
  getPrice(): number {
    return this.coffee.getPrice() + 20;
  }
}
```

And:

```ts
class SugarDecorator extends CoffeeDecorator {
  getPrice(): number {
    return this.coffee.getPrice() + 10;
  }
}
```

And:

```ts
class CreamDecorator extends CoffeeDecorator {
  getPrice(): number {
    return this.coffee.getPrice() + 30;
  }
}
```

---

# 7. Now the interesting part

We can dynamically compose them:

```ts
const coffee = new SimpleCoffee();

const milkCoffee = new MilkDecorator(coffee);

const milkSugarCoffee =
  new SugarDecorator(milkCoffee);

const finalCoffee =
  new CreamDecorator(milkSugarCoffee);

console.log(finalCoffee.getPrice());
```

Result:

```text
100
 ↓
+20
 ↓
+10
 ↓
+30
 ↓
160
```

The structure is:

```text
CreamDecorator
      ↓
SugarDecorator
      ↓
MilkDecorator
      ↓
SimpleCoffee
```

Each decorator delegates to the object it wraps and adds its own behavior.

---

# 8. Why not inheritance?

This is an important LLD question.

We could theoretically create:

```text
MilkCoffee
SugarCoffee
MilkSugarCoffee
MilkSugarCreamCoffee
...
```

This becomes a **class explosion**.

With Decorator:

```text
Coffee
 ↓
Milk
 ↓
Sugar
 ↓
Cream
```

We compose behavior dynamically.

So:

> **Decorator favors composition over creating many subclasses.**

---

# 9. Connection with OCP

Remember OCP:

> **Open for extension, closed for modification.**

Our original:

```ts
SimpleCoffee
```

doesn't need to change.

If tomorrow we add:

```text
Caramel
Chocolate
ExtraShot
Vanilla
```

we create new decorators.

```ts
class CaramelDecorator extends CoffeeDecorator {
  getPrice(): number {
    return this.coffee.getPrice() + 25;
  }
}
```

Existing classes remain unchanged.

---

# 10. Decorator vs Strategy

This is another distinction I want you to think about.

### Strategy

Usually:

```text
Choose ONE behavior
```

```text
PaymentService
      ↓
PaymentStrategy
      ↓
UPI
```

### Decorator

Usually:

```text
Add/compose MULTIPLE behaviors
```

```text
Coffee
 ↓
Milk
 ↓
Sugar
 ↓
Cream
```

So:

> **Strategy = replace/select behavior**

> **Decorator = wrap and add behavior**

---

# 11. Real-world examples

Decorator is useful for things like:

### HTTP middleware

```text
Request
 ↓
Authentication
 ↓
Logging
 ↓
Rate Limiting
 ↓
Controller
```

### File/Data processing

```text
Data
 ↓
Compression
 ↓
Encryption
 ↓
Caching
```

### Logging

```text
Service
 ↓
LoggingDecorator
 ↓
MetricsDecorator
 ↓
RetryDecorator
```

This is why the pattern is particularly relevant to backend development.

---

# 🧠 Your turn

Let's use a backend example rather than coffee.

### Problem: API Request Processing

We have:

```text
PaymentService
```

Initially it simply processes a payment.

But depending on the requirement, we may want to add:

```text
Logging
Metrics
Retry
Authorization
```

And these should be **composable**.

For example:

```text
PaymentService
    ↓
Logging
    ↓
Metrics
    ↓
Retry
```

### Questions

**Q1.** What should the common interface be?

**Q2.** What would be the base/original component?

**Q3.** What would the decorators be?

**Q4.** Why is Decorator better here than creating subclasses such as:

```text
LoggingPaymentService
MetricsPaymentService
LoggingMetricsPaymentService
LoggingRetryPaymentService
...
```

**Q5.** How is Decorator different from Strategy?

Answer in your own words first. **Don't write code yet.** I'll review your answer and correct it in the same notes/interview format we've been using.

---
### My Thoughts-
The coffee example which you shared, so I am not able to use that example to answer this question because for coffee we have an abstract method get price, which we will be using to get the price. But for this payment-related API processing, we have multiple behaviors which we want, and we can use the decorator, but it will be a general decorator, let's say for API request processing or payment API request processing. And then by using this general function, we can create or implement different different decorators for logging, payment, retry, authorization, and those all. So I think according to that. And so it is different from strategy because for strategy there is one behavior, but for doing it we have multiple approaches. But here we have different behavior which we want and we need to handle it. So that's why we should use decorator. And just for payment service-related payment, we can use strategy pattern with different type of payments, which will be one will be used, but the login, retry, authorization, if shared or the required functionality, which all or some can be needed according to the implementation. That is why decorator is preferred here, I think.

---
Yes — your **core understanding is correct**, especially your distinction between **Strategy = select one behavior** and **Decorator = compose multiple behaviors**. There is one important refinement around the API example.

## Your answer reviewed

### Q1. Common interface — ⚠️

You correctly identified that we need a **general abstraction**.

For this example, it could be:

```ts
interface PaymentProcessor {
  process(payment: Payment): Promise<void>;
}
```

The key is that **the original PaymentProcessor and every decorator implement the same interface**.

---

### Q2. Base component — ✅

The original implementation could be:

```text
PaymentProcessor
      ↓
ActualPaymentProcessor
```

It performs the actual payment processing.

---

### Q3. Decorators — ✅

You correctly identified:

```text
PaymentProcessor
      ↑
 ┌────┼──────────────┐
 ↓    ↓      ↓       ↓
Logging Metrics Retry Authorization
```

Each decorator **wraps another `PaymentProcessor`**.

For example:

```text
LoggingDecorator
      ↓
RetryDecorator
      ↓
AuthorizationDecorator
      ↓
ActualPaymentProcessor
```

This is the key idea of Decorator.

---

## Q4. Why not create many subclasses? — This is the important reason

You didn't explicitly answer this, but your explanation leads to it.

Suppose we use inheritance:

```text
PaymentService
 ├── LoggingPaymentService
 ├── RetryPaymentService
 ├── AuthorizationPaymentService
 ├── LoggingRetryPaymentService
 ├── LoggingAuthorizationPaymentService
 ├── RetryAuthorizationPaymentService
 └── LoggingRetryAuthorizationPaymentService
```

As combinations increase, the number of classes can explode.

With Decorator:

```text
ActualPaymentService
        ↓
Authorization
        ↓
Retry
        ↓
Logging
```

We can compose them dynamically.

**That's one of the strongest reasons to use Decorator.**

---

# Q5. Decorator vs Strategy — ✅ Your understanding is good

Your explanation was:

> Strategy has one behavior with multiple approaches, whereas Decorator has different behaviors that can be added as required.

I'd refine it slightly:

### Strategy

> **Choose/replace one algorithm or behavior.**

```text
PaymentService
      ↓
ONE strategy
      ↓
UPI / Card / Wallet
```

### Decorator

> **Add or compose additional behavior around an existing object.**

```text
PaymentService
      ↓
Retry
      ↓
Logging
      ↓
Metrics
      ↓
Actual Payment
```

And importantly, **multiple decorators can be combined**.

---

# 🔥 Your observation about the coffee example

You said you couldn't directly map the coffee example to this API example.

That's actually a **good observation**.

The coffee example is demonstrating:

```text
same operation
   ↓
getPrice()
   ↓
each decorator modifies the result
```

The API example is demonstrating:

```text
same operation
   ↓
process()
   ↓
each decorator adds behavior around that operation
```

So the actual Decorator concept isn't specifically:

> "Modify a price."

It's:

> **Wrap an object implementing the same contract and add behavior before/after/delegating to the wrapped object.**

For example:

```ts
class LoggingDecorator implements PaymentProcessor {
  constructor(
    private processor: PaymentProcessor
  ) {}

  async process(payment: Payment) {
    console.log("Payment started");

    await this.processor.process(payment);

    console.log("Payment completed");
  }
}
```

Then:

```text
LoggingDecorator
      ↓
PaymentProcessor
```

The decorator itself is a `PaymentProcessor`, which is why we can keep wrapping.

---

# 🧠 One correction to your wording

You said:

> "Decorator is for different behavior which we want."

That's **almost right**, but make it more precise for interviews:

> **Decorator is used when we want to dynamically add or combine responsibilities/behaviors around an existing object without modifying its implementation.**

That's the key sentence I'd put in your notes.

### And your mental model should now be:

```text
Strategy
→ Choose ONE interchangeable behavior.

Decorator
→ Add/compose MULTIPLE optional behaviors.

Factory
→ Decide/create an object.
```

That's a very useful three-pattern distinction for LLD interviews.

### Question

> **Can we consider polymorphism and composition as the major building blocks of the Decorator Pattern, where multiple decorators can be composed in different combinations to add optional behaviors?**

Yes. And I think you mean **Decorator Pattern** (not regulatory pattern). Your thinking is **mostly correct**, with one important refinement.

### Answer: **Yes, with a qualification.** ✅

Decorator heavily relies on:

**1. Polymorphism**

The original component and every decorator implement the **same interface**.

```text id="5d2s0f"
PaymentProcessor
      ↑
 ┌────┴────────┐
Actual       Decorator
Processor       ↑
           Logging / Retry / Metrics
```

This allows the caller to treat everything as a `PaymentProcessor`.

---

**2. Composition**

Each decorator **contains/wraps another `PaymentProcessor`**:

```ts id="w4x7bz"
class LoggingDecorator implements PaymentProcessor {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

So we can compose:

```text id="i0e8n6"
Logging
   ↓
Retry
   ↓
Metrics
   ↓
Actual Payment
```

And change the combination:

```text id="i4e3nq"
Retry
   ↓
Logging
   ↓
Actual Payment
```

or:

```text id="b7jhx8"
Metrics
   ↓
Authorization
   ↓
Retry
   ↓
Actual Payment
```

without changing the underlying `ActualPaymentProcessor`.

### The important qualification

Don't say:

> **"Decorator = polymorphism + composition."**

That describes **how it is commonly implemented**, but not **what the pattern is for**.

Better interview answer:

> **"Decorator is typically implemented using polymorphism and composition. The common interface allows the original component and decorators to be treated uniformly, while composition allows decorators to wrap and combine behaviors dynamically."**

And then explain the **purpose**:

> **"The purpose is to add responsibilities or behaviors to an object dynamically without modifying its original implementation or creating many subclasses."**

### 🧠 Best note

```text id="5w8u8g"
Decorator
   ↓
Purpose:
Add/compose optional behaviors dynamically

Implementation:
Polymorphism + Composition

Benefit:
Avoid modifying existing class
Avoid subclass/class explosion
Allow different decorator combinations
```

Your **"different permutations/combinations"** observation is particularly important. That's one of the major advantages of Decorator over inheritance: **you can compose behaviors at runtime instead of creating a separate class for every combination.**
