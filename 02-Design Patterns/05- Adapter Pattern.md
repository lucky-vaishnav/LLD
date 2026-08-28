Let's continue with the **next LLD pattern**.

# Topic 5 — Adapter Pattern

This is an important pattern for backend development because you'll frequently integrate with:

* Third-party APIs
* Payment providers
* External SDKs
* Legacy systems
* Different internal services with incompatible interfaces

And it connects nicely with the kind of integrations you've worked on.

---

## 1. The problem

Imagine our application expects this interface:

```ts
interface PaymentGateway {
  pay(amount: number): void;
}
```

Our application wants to do:

```ts
paymentGateway.pay(100);
```

But suppose a third-party provider gives us:

```ts
class RazorpaySDK {
  makePayment(
    amountInPaise: number
  ): void {
    console.log(`Razorpay payment: ${amountInPaise}`);
  }
}
```

The interfaces don't match.

Our application expects:

```text
pay(amount)
```

but the external SDK provides:

```text
makePayment(amountInPaise)
```

We **cannot/shouldn't modify the third-party SDK**.

---

# 2. Adapter Pattern

The Adapter sits between them:

```text id="nq4n5e"
Our Application
      ↓
PaymentGateway
      ↓
   Adapter
      ↓
RazorpaySDK
```

The Adapter converts one interface into another interface that our application expects.

---

## 3. Code

Our application's contract:

```ts
interface PaymentGateway {
  pay(amount: number): void;
}
```

Third-party SDK:

```ts
class RazorpaySDK {
  makePayment(amountInPaise: number): void {
    console.log(
      `Razorpay payment: ${amountInPaise} paise`
    );
  }
}
```

Adapter:

```ts
class RazorpayAdapter implements PaymentGateway {
  constructor(
    private razorpay: RazorpaySDK
  ) {}

  pay(amount: number): void {
    const amountInPaise = amount * 100;

    this.razorpay.makePayment(amountInPaise);
  }
}
```

Now our application doesn't need to know anything about:

```text
RazorpaySDK
makePayment()
paise
```

It only knows:

```ts
PaymentGateway
```

---

# 4. Why is this useful?

Suppose tomorrow we add another provider:

```ts
class StripeSDK {
  charge(amountInCents: number): void {
    console.log(`Stripe charge: ${amountInCents}`);
  }
}
```

We create:

```ts
class StripeAdapter implements PaymentGateway {
  constructor(
    private stripe: StripeSDK
  ) {}

  pay(amount: number): void {
    const amountInCents = amount * 100;

    this.stripe.charge(amountInCents);
  }
}
```

Now:

```text id="n2k2wx"
                  PaymentGateway
                       ↑
                ┌──────┴──────┐
                ↓             ↓
       RazorpayAdapter   StripeAdapter
                ↓             ↓
         RazorpaySDK     StripeSDK
```

Our application continues using:

```ts
paymentGateway.pay(100);
```

It doesn't need to know which external provider is underneath.

---

# 5. Adapter vs Strategy

This is important because they can look very similar.

### Strategy

Different implementations represent **different algorithms/behaviors**.

```text id="l0l72u"
PaymentStrategy
      ↑
Card / UPI / Wallet
```

### Adapter

Different implementations represent **different interfaces that need to be made compatible**.

```text id="0n9r7r"
Our Interface
     ↑
 Adapter
     ↓
External Interface
```

So:

> **Strategy → change behavior.**

> **Adapter → make incompatible interfaces work together.**

---

# 6. Adapter vs Decorator

Another important distinction.

### Decorator

```text id="4pk2w0"
PaymentService
      ↓
Logging
      ↓
Retry
      ↓
Actual Payment
```

Purpose:

> **Add behavior.**

### Adapter

```text id="g8cv7h"
Application
     ↓
Expected Interface
     ↓
Adapter
     ↓
Third-party API
```

Purpose:

> **Convert/adapt an incompatible interface.**

---

# 7. Real-world backend example

Suppose your application has:

```ts
interface ReceiptProvider {
  getReceipt(tripId: string): Promise<Receipt>;
}
```

But an external Uber integration gives you:

```ts
uber.getTripReceipt(tripId)
```

and another provider gives:

```ts
provider.fetchReceipt(tripId)
```

You don't want your business logic to know these provider-specific APIs.

Instead:

```text id="x1qj84"
                    ReceiptProvider
                         ↑
                 ┌───────┴────────┐
                 ↓                ↓
          UberAdapter       OtherAdapter
                 ↓                ↓
             Uber API       Other API
```

Your business logic simply does:

```ts
receiptProvider.getReceipt(tripId);
```

This is a very common reason to use Adapter in real backend systems.

---

# 8. One important point

Adapter does **not necessarily mean third-party integration**.

It can also be used for:

* Legacy code
* Internal modules
* Different versions of an API
* Migrating from one implementation to another
* Wrapping a library with an interface suitable for your domain

The fundamental problem is always:

> **"These two components need to work together, but their interfaces don't match."**

---

# 🧠 Your turn

Let's use a realistic backend problem.

### Problem

Our application has:

```ts
interface NotificationProvider {
  send(message: string, recipient: string): void;
}
```

But an external SMS provider gives us:

```ts
class SmsVendor {
  sendSms(phoneNumber: string, text: string): void {
    // external implementation
  }
}
```

And another vendor gives:

```ts
class AnotherSmsVendor {
  dispatch(number: string, message: string): void {
    // external implementation
  }
}
```

### Questions

**Q1.** What should be the Adapter interface/contract?

**Q2.** What are the Adapters?

**Q3.** What would the relationship between Adapter and the external vendor look like?

**Q4.** Why shouldn't our `NotificationService` directly depend on `SmsVendor`?

**Q5.** What is the difference between **Adapter vs Strategy** in this example?

Answer in your own words first, and I'll review it like the previous patterns.

---
### My Thoughts-
So for that, our interface or the contract of the adapter will be something like NotificationAdapter, and the adapter will be implementing this NotificationAdapter interface, like SMS vendor adapter or some other vendor adapter implement this NotificationAdapter interface. And the relation will be like that we will be using the same functions which we used to use, and without changing the external vendor functions, and we should not change it, and we will not be able to change it with our existing old code. This will work like a middleware to just use our existing functions without modifying or changing function name. And Notification Service should not depend on SMS because this service should work as an orchestration, and different type of SMS messaging can be supported by this service. And when we add any new SMS vendor, then Notification Service should not be impacted. No change will be required. So this will achieve the similar responsibility functionality and open to extend as well. And you can say that composition also is there. And difference between adapter versus strategy is that strategy is about one functionality, which can be done using different implementation. So there is only one behavior, and we have the option to choose which behavior to choose. And with adapter, this is all about working with the legacy system or the external API, external SDK, so that our existing code can use same existing contract without adding new contract or new function names.

---
Yes — **your understanding is correct**, and you're making the right connections between Adapter, Strategy, OCP, composition, and external integrations. A few terminology refinements will make your answer stronger in interviews.

## Q1. Adapter contract — ⚠️ Small correction

You said:

> "NotificationAdapter interface"

You *can* name it that way, but conceptually the important interface is actually the **interface expected by our application**.

For example:

```ts
interface NotificationProvider {
  send(message: string, recipient: string): void;
}
```

Then:

```text
NotificationProvider
        ↑
 ┌──────┴────────┐
 ↓               ↓
SmsVendorAdapter  OtherVendorAdapter
```

The Adapter implements **our application's expected interface**.

---

## Q2. Concrete Adapters — ✅

Exactly:

```text
SmsVendorAdapter
AnotherSmsVendorAdapter
```

For example:

```ts
class SmsVendorAdapter implements NotificationProvider {
  constructor(
    private vendor: SmsVendor
  ) {}

  send(message: string, recipient: string): void {
    this.vendor.sendSms(recipient, message);
  }
}
```

The adapter translates:

```text
Our application:
send(message, recipient)

        ↓ Adapter

Vendor:
sendSms(phoneNumber, text)
```

---

## Q3. Adapter ↔ Vendor relationship — ✅

Your "middleware" analogy is useful, although in an interview I'd use **translation layer** or **compatibility layer** rather than middleware.

```text
Our Application
      ↓
NotificationProvider
      ↓
SmsVendorAdapter
      ↓
SmsVendor
```

The Adapter translates our application's contract into the vendor's API.

And importantly:

> **We don't modify the external vendor.**

---

## Q4. Why shouldn't NotificationService depend directly on SmsVendor? — ✅

Your answer is correct.

Without Adapter:

```text
NotificationService
       ↓
    SmsVendor
```

Now adding:

```text
AnotherSmsVendor
```

could require changes in `NotificationService`.

With Adapter:

```text
NotificationService
       ↓
NotificationProvider
       ↑
 ┌─────┴──────┐
 ↓            ↓
SMS Adapter   Other Adapter
```

Now the service depends on **our abstraction**, not the external implementation.

That's a good application of **DIP** and helps with **OCP**.

---

# Q5. Adapter vs Strategy — ✅ Very good

Your distinction is correct.

I'd make it even sharper:

### Strategy

> **Different implementations provide different ways of performing the same behavior/algorithm.**

```text
PaymentStrategy
      ↑
Card / UPI / Wallet
```

We choose the behavior we want.

### Adapter

> **Different implementations translate incompatible external interfaces into the interface our application expects.**

```text
Our Interface
      ↑
   Adapter
      ↓
External Interface
```

We aren't primarily choosing a different algorithm. We're making two incompatible interfaces **compatible**.

---

# One correction to your statement

You said:

> "with adapter ... without adding new contract or new function names."

I'd phrase this as:

> **"The application keeps using its existing contract, while the Adapter translates that contract into whatever API the external system provides."**

Because the external vendor **can absolutely have different function names**.

That's actually the reason we need the Adapter:

```text
Our code:
send()

Vendor:
sendSms()

Another vendor:
dispatch()
```

Our business code continues using:

```text
send()
```

while each Adapter handles the translation.

---

# 🧠 Your final notes

### Adapter Pattern

> **Adapter allows incompatible interfaces to work together by translating an existing application contract into the interface expected by another component, such as a legacy system, third-party API, or SDK.**

### Core structure

```text
Application
    ↓
Expected Interface
    ↑
  Adapter
    ↓
External/Legacy Interface
```

### Main purpose

> **Make incompatible interfaces compatible without modifying the existing application code or external component.**

### Strategy vs Adapter

```text
Strategy
→ Different ways to perform a behavior.

Adapter
→ Different interface that needs translation.
```

And one particularly good point from your answer:

> **"When we add a new vendor, NotificationService should not change."**

Exactly. That's one of the practical reasons this pattern is so useful in backend integrations.

You've now covered **Factory → Strategy → Observer → Decorator → Adapter**. These patterns are starting to connect with the OOP/SOLID concepts we learned earlier, which is exactly what we want at the senior LLD level.


### Q1

> **In the Adapter Pattern, if multiple adapters such as `UberReceiptAdapter` and `OtherReceiptAdapter` implement the same `ReceiptProvider` interface, how does `receiptProvider.getReceipt(tripId)` know which external provider to call? Is `receiptProvider` actually the specific adapter object selected by the application?**

Yes — **your understanding is correct**, and this is an important point that I glossed over in the earlier example.

## Answer: Exactly. ✅

`receiptProvider` is a **reference to whichever concrete Adapter has been selected/injected**.

For example:

```ts
interface ReceiptProvider {
  getReceipt(tripId: string): Promise<Receipt>;
}
```

Uber's external API:

```ts
class UberAPI {
  getTripReceipt(tripId: string) {
    // Uber API call
  }
}
```

Uber Adapter:

```ts
class UberReceiptAdapter implements ReceiptProvider {
  constructor(
    private uberApi: UberAPI
  ) {}

  async getReceipt(tripId: string): Promise<Receipt> {
    return this.uberApi.getTripReceipt(tripId);
  }
}
```

Another provider:

```ts
class OtherReceiptAdapter implements ReceiptProvider {
  constructor(
    private otherApi: OtherAPI
  ) {}

  async getReceipt(tripId: string): Promise<Receipt> {
    return this.otherApi.fetchReceipt(tripId);
  }
}
```

Now imagine we select Uber:

```ts
const receiptProvider: ReceiptProvider =
  new UberReceiptAdapter(uberApi);
```

Then:

```ts
receiptProvider.getReceipt("TRIP-101");
```

actually executes:

```text
receiptProvider
      ↓
UberReceiptAdapter
      ↓
UberAPI.getTripReceipt()
```

If we instead inject the other provider:

```ts
const receiptProvider: ReceiptProvider =
  new OtherReceiptAdapter(otherApi);
```

the **same code**:

```ts
receiptProvider.getReceipt("TRIP-101");
```

becomes:

```text
receiptProvider
      ↓
OtherReceiptAdapter
      ↓
OtherAPI.fetchReceipt()
```

---

# This is where Polymorphism + DI come in

This is very similar to what we discussed with Strategy.

The business code only knows:

```ts
ReceiptProvider
```

The actual implementation is decided outside:

```text
                ReceiptProvider
                      ↑
             ┌────────┴────────┐
             ↓                 ↓
      UberAdapter       OtherAdapter
             ↓                 ↓
          Uber API        Other API
```

Then DI supplies the appropriate object:

```ts
ReceiptService(
  receiptProvider
)
```

So inside the service:

```ts
await receiptProvider.getReceipt(tripId);
```

The service **doesn't need to know whether it's Uber or another provider**.

---

## ⚠️ But there's one important question

You might now ask:

> **"Okay, but who decides whether to create `UberReceiptAdapter` or `OtherReceiptAdapter`?"**

That's a **separate responsibility**.

For example, some higher-level configuration/factory/DI container can decide:

```ts
const receiptProvider =
  provider === "UBER"
    ? new UberReceiptAdapter(uberApi)
    : new OtherReceiptAdapter(otherApi);
```

Then it passes that object into:

```ts
new ReceiptService(receiptProvider);
```

So the architecture becomes:

```text
Provider selection/configuration
          ↓
    UberAdapter
          ↓
   ReceiptService
          ↓
receiptProvider.getReceipt()
```

### 🧠 The key point for your notes

> **Adapter does not decide which provider to use. Adapter only translates the application's expected interface into a specific provider's interface.**

And:

> **Polymorphism allows the application to call `receiptProvider.getReceipt()` regardless of which concrete Adapter was injected.**

This distinction is important because otherwise Adapter can start looking like Factory or Strategy. **Adapter = interface translation; selection of the adapter is a separate concern.**
