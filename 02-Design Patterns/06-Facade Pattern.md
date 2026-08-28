Let's move to the **next LLD pattern**.

# Topic 6 — Facade Pattern

This one is relatively simple, but **very useful in backend design** and often gets confused with **Service classes, Adapter, and orchestration**.

---

## 1. The problem

Imagine a **Trip Completion** flow.

When a trip is completed, we need to perform several operations:

```text
Complete Trip
   ├── Validate trip
   ├── Calculate fare
   ├── Process payment
   ├── Generate receipt
   ├── Send notification
   └── Update analytics
```

Without a Facade, the caller might need to know about all these services:

```ts
tripValidator.validate(trip);

fareService.calculate(trip);

paymentService.process(trip);

receiptService.generate(trip);

notificationService.send(trip);

analyticsService.track(trip);
```

The caller now has knowledge of **many internal components and the sequence of the workflow**.

That's the problem Facade tries to solve.

---

# 2. Facade Pattern

We create a **simple entry point** that hides the complexity of the underlying subsystem.

```text
                    TripCompletionFacade
                            ↓
       ┌────────────┬───────┼──────────┬──────────┐
       ↓            ↓       ↓          ↓          ↓
   Validator      Fare   Payment    Receipt   Notification
```

The caller simply does:

```ts
tripCompletionFacade.completeTrip(trip);
```

The Facade handles the interaction with the underlying services.

---

# 3. Basic code

Our existing services:

```ts
class TripValidator {
  validate(trip: Trip) {
    // validation
  }
}

class FareService {
  calculate(trip: Trip) {
    // calculate fare
  }
}

class PaymentService {
  process(trip: Trip) {
    // payment
  }
}

class ReceiptService {
  generate(trip: Trip) {
    // receipt
  }

class NotificationService {
  send(trip: Trip) {
    // notification
  }
}
```

Now the Facade:

```ts
class TripCompletionFacade {
  constructor(
    private validator: TripValidator,
    private fareService: FareService,
    private paymentService: PaymentService,
    private receiptService: ReceiptService,
    private notificationService: NotificationService
  ) {}

  completeTrip(trip: Trip) {
    this.validator.validate(trip);

    this.fareService.calculate(trip);

    this.paymentService.process(trip);

    this.receiptService.generate(trip);

    this.notificationService.send(trip);
  }
}
```

Caller:

```ts
tripCompletionFacade.completeTrip(trip);
```

That's it.

The caller doesn't need to know:

```text
TripValidator
FareService
PaymentService
ReceiptService
NotificationService
```

---

# 4. What is the Facade actually doing?

This is the most important part.

Facade is **not necessarily implementing the business logic of all these services**.

It's primarily providing:

> **A simplified interface to a complex subsystem.**

Think:

```text
Complex subsystem
      ↓
     Facade
      ↓
Simple interface
```

---

# 5. Real-world analogy

Think about an ATM.

You want:

```text
Withdraw ₹10,000
```

You don't directly interact with:

```text
Account validation
Balance system
Fraud system
Bank ledger
Cash dispenser
Transaction logging
```

You interact with:

```text
ATM
 ↓
Withdraw
```

The ATM provides a **simplified interface over a complex subsystem**.

That's the basic idea of Facade.

---

# 6. Facade vs Service

This is particularly important for you.

A Service might contain:

```text
Business responsibility
```

For example:

```text
PaymentService
→ payment-related business logic
```

Facade is more about:

```text
Simplifying access to multiple components
and coordinating their interaction.
```

So:

```text
Service
→ owns a particular responsibility

Facade
→ provides a simplified entry point over multiple components
```

---

# 7. Facade vs Adapter

You just learned Adapter, so this distinction is important.

### Adapter

```text
Our Interface
      ↓
   Adapter
      ↓
External/Legacy Interface
```

Purpose:

> **Make incompatible interfaces compatible.**

### Facade

```text
Complex Subsystem
      ↓
    Facade
      ↓
Simple Interface
```

Purpose:

> **Hide complexity and provide a simpler interface.**

---

# 8. Facade vs Decorator

### Decorator

Adds behavior:

```text
Payment
 ↓
Logging
 ↓
Retry
 ↓
Payment
```

### Facade

Hides complexity:

```text
             Facade
                ↓
    ┌───────────┼───────────┐
    ↓           ↓           ↓
 Payment      Receipt    Notification
```

So:

```text
Decorator → Add behavior

Facade → Hide complexity
```

---

# 9. Does Facade violate SRP?

This is a good senior-level question.

At first glance:

```ts
TripCompletionFacade
```

is interacting with many things.

But its responsibility isn't:

```text
Payment
Receipt
Notification
Fare calculation
```

Its responsibility is:

> **Provide a simplified interface for the trip-completion workflow.**

The underlying services still own their respective responsibilities.

So it can still have a valid single responsibility.

---

# 10. One important observation

Facade **doesn't necessarily mean you must create a class called `Facade`**.

You may already have something like:

```text
CheckoutService
OrderService
TripService
PaymentWorkflow
```

which acts as a Facade over several internal components.

The **pattern is about the role**, not the class name.

---

# 🧠 Your turn

Let's use a realistic example.

### Parking Payment System

Suppose completing a parking payment requires:

```text
1. Validate parking session
2. Calculate parking fee
3. Apply discount
4. Process payment
5. Generate receipt
6. Send email
7. Update parking session
```

We already have separate components:

```text
SessionValidator
FeeCalculator
DiscountService
PaymentService
ReceiptService
EmailService
SessionService
```

### Questions

**Q1.** What problem would a Facade solve here?

**Q2.** What would you call the Facade?

**Q3.** What should the caller need to know after introducing the Facade?

**Q4.** Should the Facade contain all the actual payment, receipt, discount, etc. business logic? Why or why not?

**Q5.** What's the difference between **Facade vs Adapter**?

**Q6.** What's the difference between **Facade vs Service**?

Answer in your own words. Then I'll review it and give you the **interview + notes version**.

---
### My Thoughts-
So the problem facade will solve is that understanding the actual flow or interacting with different component of the functional flow, we can do it. But using facade we can make it simplified and more maintainable as well, and it can be reusable as well, because if we call this component in multiple core section, it will be repeating things. We can create a facade for it, and for this payment parking payment system, the facade can be Parking Payment Process, a generalized name which will have all the six, seven functionalities triggered. So the caller did not need to know anything, just facade function like payment parking payment process, something like that, and the rest will be handled by that facade. And facade will not actually have the business logic in it. It will be the interaction point for the business logic, which is important for the business, and it will be just an interaction point. And so actually adapter is totally different from facade, I will say, because adapter is making incompatible interfaces into the current compatible interface. And facade is just an interaction point with the business functionality or business rules in a combination. Service, different service have one responsibility and should have one responsibility and should do the orchestration about one functionality of the business behavior. And facade, we can say that, is a group of the flow, the service edge, and can be used as an interaction point, and it will be very maintainable and it will be very easy to use it. Yes, at first glance it looks like it is looking like not applying the single responsibility functionality, but facade is not for that. Facade's responsibility is that it should hide the complexity of the functionality and should have easy-to-use interaction point. So it is the responsibility of the facade, which is why it is already supporting SIP, like single— sorry, SRP.

---
Yes — **your understanding is very good**. You have correctly identified the core purpose of Facade, and your distinction from Adapter is especially important.

There are just **two refinements** I'd make for your interview answer.

---

# 1. What problem does Facade solve? — ✅

You said:

> "Understanding/interacting with different components of a functional flow can be simplified."

Exactly.

Without Facade:

```text
Caller
 ├── SessionValidator
 ├── FeeCalculator
 ├── DiscountService
 ├── PaymentService
 ├── ReceiptService
 ├── EmailService
 └── SessionService
```

With Facade:

```text
Caller
  ↓
ParkingPaymentFacade
  ↓
Multiple internal services
```

The caller only needs to know:

```ts
parkingPaymentFacade.processPayment(session);
```

---

# 2. Your reuse point — ✅ but not the primary purpose

You said Facade can be reusable and avoid repeating the same flow.

**Correct.**

For example, if:

```text
Mobile App
Admin Portal
Background Job
```

all need the same parking-payment workflow, they can use:

```text
ParkingPaymentFacade
```

instead of each implementing:

```text
validate
→ calculate
→ discount
→ payment
→ receipt
→ email
→ update session
```

But in an interview, I'd say:

> **Reuse is a benefit; hiding subsystem complexity is the primary purpose.**

---

# 3. Facade shouldn't own all business logic — ✅

Your statement was excellent:

> "Facade will not actually have the business logic. It will be the interaction point for the business logic."

Exactly.

For example:

```text
ParkingPaymentFacade
       ↓
 ┌─────┼───────────────┐
 ↓     ↓               ↓
Fee   Payment        Receipt
Service Service      Service
```

The Facade coordinates them.

It shouldn't become:

```text
ParkingPaymentFacade
→ calculate fee
→ calculate discount
→ validate payment
→ generate receipt
→ send email
```

with hundreds of lines of actual business rules.

Otherwise you've just created a **God class**.

---

# 4. Facade vs Adapter — ✅ Excellent

Your explanation is accurate.

### Adapter

> **Changes the interface.**

```text
Our Interface
     ↑
  Adapter
     ↓
External Interface
```

Example:

```text
getReceipt()
     ↓
UberAdapter
     ↓
getTripReceipt()
```

### Facade

> **Simplifies access to multiple components.**

```text
                    Facade
                      ↓
        ┌─────────────┼────────────┐
        ↓             ↓            ↓
     Payment        Receipt      Email
```

### Best one-line distinction

> **Adapter makes interfaces compatible; Facade makes a complex subsystem easier to use.**

Keep that in your notes.

---

# 5. Facade vs Service — ⚠️ One refinement

You said:

> "Service should have one responsibility and should do orchestration about one functionality."

That's broadly fine, but don't make **"Service = orchestration"** a strict definition.

A Service can contain domain/application business logic depending on the architecture.

A better distinction:

### Service

> **Owns a specific business responsibility/capability.**

```text
PaymentService
→ payment-related operations
```

### Facade

> **Provides a simplified entry point that coordinates/accesses multiple components of a subsystem.**

```text
ParkingPaymentFacade
        ↓
FeeService
PaymentService
ReceiptService
NotificationService
```

So a Facade **can use services**, but a service isn't automatically a Facade.

---

# 6. Your SRP explanation — ✅ Very good

You correctly identified the apparent problem:

```text
Facade
 ├── Payment
 ├── Receipt
 ├── Email
 ├── Discount
 └── Session
```

At first glance:

> "Isn't this violating SRP?"

Not necessarily.

Because the Facade has **one responsibility**:

> **Provide a simplified entry point to the parking-payment workflow/subsystem.**

The individual business responsibilities remain inside their respective services.

---

# 🧠 Final notes for you

### Facade Pattern

> **Facade provides a simple, unified interface over a complex subsystem, hiding the internal components and their interactions from the caller.**

### Main purpose

```text
Hide complexity
      ↓
Simple entry point
      ↓
Multiple internal components
```

### Facade should:

* Simplify interaction
* Hide subsystem complexity
* Coordinate existing components
* Reduce coupling between caller and internal components
* Potentially provide a reusable workflow

### Facade should NOT:

* Become a God class
* Duplicate all underlying business logic
* Replace every service

### Key comparisons

```text
Adapter
→ Make incompatible interfaces compatible

Facade
→ Make a complex subsystem easier to use

Decorator
→ Add behavior to an existing object

Strategy
→ Select interchangeable behavior
```

And your mental model is now getting quite strong:

> **Adapter = "Translate this."**
> **Facade = "Simplify this."**
> **Decorator = "Add to this."**
> **Strategy = "Choose how to do this."**
