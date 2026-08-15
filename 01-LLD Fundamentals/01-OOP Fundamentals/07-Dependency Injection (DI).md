## 🆕 LLD Topic #7: Dependency Injection (DI)

This is an important topic because you've already encountered it several times in our examples, especially:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

We've been using DI without formally discussing **what it actually is and why it matters**.

---

# 1. What is Dependency Injection?

First understand **dependency**.

If `PaymentService` needs a `PaymentProcessor` to work:

```text
PaymentService
      ↓
PaymentProcessor
```

then `PaymentProcessor` is a **dependency** of `PaymentService`.

Now consider this design:

```ts
class PaymentService {
  private processor: PaymentProcessor;

  constructor() {
    this.processor = new StripeProcessor();
  }
}
```

The problem is:

**`PaymentService` creates its own dependency.**

Now `PaymentService` is tightly coupled to `StripeProcessor`.

---

## 2. Dependency Injection

Instead, we give the dependency to `PaymentService` from outside:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

Then:

```ts
const service = new PaymentService(
  new StripeProcessor()
);
```

The dependency is **injected** into the class.

That's Dependency Injection.

### Simple definition

> **Dependency Injection means providing an object's dependencies from outside instead of having the object create them itself.**

---

# 3. Why is this useful?

Consider:

```ts
class PaymentService {
  constructor() {
    this.processor = new StripeProcessor();
  }
}
```

Now imagine we want CyberSource:

```text
PaymentService → StripeProcessor
```

We're tightly coupled.

But with DI:

```text
              PaymentProcessor
                     ↑
          ┌──────────┼──────────┐
          │          │          │
       Stripe    CyberSource   Razorpay
          │          │          │
          └──────────┼──────────┘
                     ↓
              PaymentService
```

`PaymentService` doesn't care which implementation it receives.

---

# 4. DI + Abstraction

This is where today's topic connects directly to our previous topics.

We define:

```ts
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

Then:

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

Now we inject:

```ts
new PaymentService(
  new StripeProcessor()
);
```

or:

```ts
new PaymentService(
  new CyberSourceProcessor()
);
```

or:

```ts
new PaymentService(
  new RazorpayProcessor()
);
```

This gives us:

```text
Abstraction
     ↓
PaymentProcessor
     ↓
Dependency Injection
     ↓
PaymentService receives implementation
     ↓
Loose coupling
```

---

# 5. Why is DI very useful for testing?

This is one of the biggest practical benefits.

Suppose the real payment processor makes an external API call.

You don't want your unit test to actually call Stripe/CyberSource.

So we create:

```ts
class MockPaymentProcessor implements PaymentProcessor {
  pay(amount: number) {
    return {
      success: true
    };
  }
}
```

Then:

```ts
const service = new PaymentService(
  new MockPaymentProcessor()
);
```

Now the test can run without the real payment provider.

That's much easier because the dependency can be replaced.

---

# 6. Three common forms of DI

### Constructor Injection

This is the most common and usually the preferred approach.

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

The dependency is available when the object is created.

---

### Setter Injection

Dependency is provided later:

```ts
class PaymentService {
  private processor!: PaymentProcessor;

  setProcessor(processor: PaymentProcessor) {
    this.processor = processor;
  }
}
```

Usage:

```ts
service.setProcessor(new StripeProcessor());
```

This can be useful when the dependency is optional or can legitimately change.

But it also means the object can exist in an incomplete state.

---

### Method Injection

Dependency is supplied only for a particular operation:

```ts
class PaymentService {
  process(
    processor: PaymentProcessor,
    amount: number
  ) {
    return processor.pay(amount);
  }
}
```

This makes sense when the dependency is needed only for a specific operation.

---

# 7. Why constructor injection is usually preferred

Consider:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

The object **cannot be created without its required dependency**.

That's good because the object is always in a valid state.

Compare:

```ts
const service = new PaymentService();

// processor not set yet 😕
```

with constructor injection:

```ts
const service = new PaymentService(
  new StripeProcessor()
);
```

The dependency requirement is explicit.

---

# 8. DI vs Dependency Inversion

These are related but **not the same thing**.

This distinction is important for interviews.

### Dependency Injection

A **technique**:

> Provide dependencies from outside.

Example:

```ts
constructor(processor: PaymentProcessor)
```

### Dependency Inversion Principle

A **SOLID principle**:

> High-level modules should not depend directly on low-level modules; both should depend on abstractions.

We'll study DIP properly when we start SOLID.

So:

```text
Dependency Injection
        ↓
Technique

Dependency Inversion Principle
        ↓
Design principle
```

DI is often one way to implement a design that follows DIP.

---

# 9. Real backend example

Imagine your Node.js application has:

```text
BookingService
     ↓
PaymentService
     ↓
PaymentProcessor
```

Instead of:

```ts
class PaymentService {
  private processor = new CyberSourceProcessor();
}
```

you can have:

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

Then your application composition/root layer decides:

```ts
const paymentProcessor = new CyberSourceProcessor();

const paymentService = new PaymentService(
  paymentProcessor
);

const bookingService = new BookingService(
  paymentService
);
```

Notice an important architectural idea:

> **The classes focus on their business responsibility; object creation/wiring can happen somewhere else.**

This becomes especially powerful in larger applications.

---

# 10. Dependency Injection vs Dependency Inversion vs Inversion of Control

You'll hear all three terms.

For now, keep them separate:

```text
Dependency Injection
→ How dependency is provided

Dependency Inversion
→ How dependencies should be structured

Inversion of Control
→ Control of object creation/execution is moved away
  from the class itself
```

We'll revisit these when we study SOLID and architecture.

---

# 🎯 Today's takeaway

The most important definition:

> **Dependency Injection means a class receives its dependencies from outside instead of creating them itself.**

Remember this contrast:

### ❌ Tightly coupled

```ts
class PaymentService {
  private processor = new StripeProcessor();
}
```

### ✅ Dependency Injection

```ts
class PaymentService {
  constructor(
    private processor: PaymentProcessor
  ) {}
}
```

And the benefits:

```text
DI
 ↓
Loose coupling
 ↓
Replaceable implementations
 ↓
Easier testing
 ↓
Better extensibility
```

---

### Your LLD fundamentals progress

We've now covered:

1. Class & Object
2. Encapsulation
3. Abstraction
4. Inheritance vs Composition
5. Polymorphism
6. Association / Aggregation / Composition
7. **Dependency Injection** ← Today
