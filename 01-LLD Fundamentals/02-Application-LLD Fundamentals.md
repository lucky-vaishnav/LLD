# 🧠 LLD Fundamentals — Application #2

## Design a Parking Payment System

This is also close to real backend systems, so it's a good exercise for you.

### Problem

Design a **Payment System** that supports:

```text
Credit Card
Debit Card
UPI
Wallet
```

The system should:

1. Process a payment for an amount.
2. Support multiple payment methods.
3. Allow adding a new payment method later without modifying the core payment service.
4. Each payment method can have its own implementation.
5. The payment service should be easy to unit test.
6. Don't worry about actual gateway/API integration yet.

---

## 🎤 Interview question

Don't write code initially.

Design it at the class/interface level and explain your reasoning.

### Q1. Abstraction

What interface or abstract class would you create?

For example, would you have something like:

```text
PaymentMethod
```

or something else?

And what contract should it expose?

---

### Q2. Implementations

What concrete classes would you create for:

```text
Credit Card
Debit Card
UPI
Wallet
```

---

### Q3. PaymentService

What should `PaymentService` depend on?

Would you make it depend directly on:

```text
CreditCardPayment
UPIPayment
...
```

or on an abstraction?

Why?

---

### Q4. New payment method

Tomorrow we add:

```text
Net Banking
```

What exactly would you need to change in your design?

The ideal answer should demonstrate **OCP**.

---

### Q5. Important design question

Suppose the caller says:

```text
paymentMethod = "UPI"
amount = 500
```

**Who should decide which concrete implementation (`UPIPayment`) should be used?**

Would you put this decision inside:

* `PaymentService`
* a Factory
* somewhere else

And **why?**

This is the most interesting question of today's exercise.

---

### Q6. Testing

How would you unit-test `PaymentService` without actually making a payment?

Think about the principles we've already learned.

---

Take your time and answer these in your own words, just like you did for the Notification System. **Don't worry about code yet.** I'll review your design afterward and turn the final design into concise notes for you.

---
### My Thought-
So for this problem for the payment methods, so when we talk about the payment methods, according to me, there should be three responsibilities majorly for now. One will be the payment method related action, second will be the payment flow related action like pay, and third will be the refund. And we know that we should not create one interface to just to overload it. And we can follow that interface segregation principle. So we should have these three interfaces. For payment flow, let's say there will be a pay function. For refund interface, so there will be a refund function. For payment method interface, so there will be an action like add, delete, and update. And those will be implemented by different payment methods flow. And there will be, according to me, three services as well. For payment flow, so it will be payment services. Refund, there will be refund service. And payment method, there there will be payment method service. And these services will be not be dependent on the hardcoded objects. So those payment method related objects will be passed by using our dependency inversion principle, and we will be injecting those required objects to these services using dependency injection. And which to use, what to use, and how we switch or how to decide which one we are using like credit card, debit card, UPI. For that, we can use one is we can filter it out using the API level, and second is we can do it on the controller level as the request will come to those components first. And according to that, we can create the related dependencies. And yeah, for testing purpose as well, this will be a good approach because those services for payment, refund, and payment method, we can use mock payments as well, mock refunds as well, and mock payment method actions as well. And for the extend purpose, yeah, we can support the new payment method as well in the future. We do not require any change in this payment refund and payment method related services. We just need to extend the existing interface or the related classes which are implementing those interfaces. So yeah. And yeah, give you my thought and share your input. Share in a way that it will be useful for my notes as well.

---
Your thinking is **good**, especially around ISP, separating payment/refund responsibilities, DIP, DI, testing, and extensibility. But there is one **important design issue** in your approach that I'd want you to correct for senior-level LLD.

# 📝 Review — Payment System

## 1. Your three interfaces — mostly correct

You proposed:

```text
PaymentMethod
Payment
Refund
```

This is a good observation from **ISP**.

For example:

```text
PaymentProcessor
    → pay()

RefundProcessor
    → refund()

PaymentMethod
    → add()
    → update()
    → delete()
```

The important thing is that these represent **different capabilities**.

A payment provider that supports payment but doesn't support refunds shouldn't be forced to implement `refund()`.

So this is a good application of ISP.

---

# ⚠️ 2. But `PaymentMethod` needs clarification

You proposed:

```text
PaymentMethod
 ├── add()
 ├── update()
 └── delete()
```

Here I'd pause and ask:

> **What exactly are we adding/updating/deleting?**

If we're talking about:

```text
Credit Card
Debit Card
UPI
Wallet
```

then these are **payment methods/types**.

But `add/update/delete` sounds more like **managing a user's saved payment instrument**.

For example:

```text
User's saved cards
    ↓
Add Card
Update Card
Delete Card
```

That's a different responsibility from:

```text
Payment processing
    ↓
Charge ₹500
```

So I would separate these concepts.

### Better:

```text
PaymentProcessor
    → pay()

RefundProcessor
    → refund()

PaymentMethodRepository / PaymentMethodService
    → add()
    → update()
    → delete()
```

And don't automatically assume the same classes need to implement all three.

This is actually **SRP + ISP working together**.

---

# 3. The bigger design issue: where to select UPI/Card/etc.?

You said:

> "We can filter it at API level or controller level and create the related dependency."

This is the part I'd change.

### ❌ I wouldn't put payment-method selection in the controller.

For example:

```ts
if (type === "upi") {
   processor = new UpiProcessor();
}

if (type === "card") {
   processor = new CardProcessor();
}
```

Now your controller knows about concrete payment implementations.

That creates unnecessary coupling.

Instead:

```text
Controller
    ↓
PaymentService
    ↓
PaymentProcessor abstraction
    ↑
Factory / Registry
    ↓
UPI / Card / Wallet
```

The controller should generally say:

> "Process this payment."

It shouldn't need to know how the appropriate processor is constructed.

---

# ⭐ 4. This is where Factory Pattern naturally enters

This is the missing piece I wanted you to discover from the exercise.

Suppose the request contains:

```json
{
  "paymentMethod": "UPI",
  "amount": 500
}
```

We can have:

```text
PaymentProcessorFactory
          ↓
   "UPI" → UpiProcessor
   "CARD" → CardProcessor
   "WALLET" → WalletProcessor
```

Then:

```text
Controller
    ↓
PaymentService
    ↓
Factory
    ↓
PaymentProcessor
    ↓
UPI / Card / Wallet
```

The exact implementation-selection mechanism can vary, but **centralizing the selection/construction logic** is cleaner than spreading `if/else` across controllers/services.

We'll formally learn the **Factory Pattern** later.

---

# 5. Your PaymentService idea is correct

You correctly said the service shouldn't depend on hardcoded implementations.

Good:

```ts
class PaymentService {
    constructor(
        private processor: PaymentProcessor
    ) {}
}
```

Bad:

```ts
class PaymentService {
    private processor = new UpiProcessor();
}
```

The first gives us:

```text
PaymentService
      ↓
PaymentProcessor
      ↑
 ┌────┼─────┐
UPI  Card  Wallet
```

This demonstrates:

* DIP
* DI
* Abstraction
* Polymorphism
* Low coupling

---

# 6. Do we really need three services?

You proposed:

```text
PaymentService
RefundService
PaymentMethodService
```

This **can be a good design**, but don't make the split automatically just because we have three interfaces.

For example:

```text
PaymentService
    → payment workflow

RefundService
    → refund workflow

PaymentMethodService
    → saved payment method management
```

These are genuinely different business responsibilities, so separating them makes sense.

But the principle is:

> **Split services based on business responsibility, not simply based on the number of interfaces.**

That's an important distinction.

---

# 7. Your testing approach — ✅ Correct

You said:

> "We can use mock payments, mock refunds, and mock payment methods."

Exactly.

For example:

```text
PaymentService
      ↓
PaymentProcessor
      ↑
MockPaymentProcessor
```

During testing:

```ts
const mockProcessor = new MockPaymentProcessor();

const service = new PaymentService(mockProcessor);
```

Now you can test the business logic without calling:

```text
Stripe
UPI Gateway
Bank
Wallet provider
```

This is one of the practical benefits of **DIP + DI**.

---

# 8. Adding Net Banking

You correctly said:

> "We don't need to modify PaymentService."

Exactly.

Add:

```text
NetBankingProcessor
        ↓
PaymentProcessor
```

and register it with whatever selection mechanism we're using.

The core business logic remains unchanged.

That's **OCP**.

---

# 9. Your design after refinement

I'd structure the conceptual design like this:

```text
                       PaymentService
                             │
                             ↓
                    PaymentProcessor
                             ↑
               ┌─────────────┼─────────────┐
               │             │             │
          CardProcessor  UpiProcessor  WalletProcessor
               │             │             │
               └─────────────┼─────────────┘
                             ↑
                    Factory / Registry
                             ↑
                        Controller
```

And separately:

```text
RefundService
      ↓
RefundProcessor
      ↑
 ┌────┼──────────┐
Card  UPI      Wallet
```

And saved payment methods:

```text
PaymentMethodService
      ↓
PaymentMethodRepository
      ↓
Database
```

**Notice something important:** these are not necessarily the same objects.

A `Card` as a saved payment instrument and a `CardProcessor` that processes a payment are different concepts.

That's an important LLD modeling distinction.

---

# 🧠 Notes for you

### Payment System — LLD

> Separate different business capabilities instead of creating one large interface.

```text
PaymentProcessor
    → pay()

RefundProcessor
    → refund()

PaymentMethodService
    → manage saved payment methods
```

`PaymentService` should depend on the `PaymentProcessor` abstraction rather than concrete implementations.

```text
PaymentService
      ↓
PaymentProcessor
      ↑
Card / UPI / Wallet / NetBanking
```

Use **DI** to provide dependencies and **DIP** to keep high-level business logic independent of concrete payment providers.

When the payment method is determined dynamically, avoid putting concrete implementation selection inside controllers. A **Factory/Registry** can centralize the selection.

Testing becomes easier because real payment processors can be replaced with mocks.

---

# ⭐ One thing I particularly liked in your answer

You didn't just say:

> "We'll use SOLID."

You actually identified:

```text
ISP → separate contracts
DIP → depend on abstractions
DI → provide implementations
OCP → add new payment methods
Mocking → testability
```

That's exactly the transition we're looking for: **principle → actual design decision**.

The next level for you is becoming comfortable with **where each decision belongs**—Controller vs Service vs Factory vs Domain object vs Repository.

---

## 🎯 Fundamentals progress

I'd say we're **very close** to completing the fundamentals application phase, but I don't want to declare it yet.

Our next exercise should test a different area: **object modeling + state + encapsulation**, rather than another service/interface-heavy problem.

Once we've done that and I think you've demonstrated the core fundamentals, I'll explicitly mark:

> 🎉 **LLD Fundamentals Completed**

Then we'll move into **Design Patterns** and start learning them through real LLD problems rather than studying patterns in isolation.
