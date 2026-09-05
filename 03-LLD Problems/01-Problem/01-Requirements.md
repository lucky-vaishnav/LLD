# Integrated LLD Problem 1 — Ride Booking System

Design a simplified **Ride Booking System** similar to an Uber/BART ride-booking flow.

### Requirements

A customer should be able to:

1. Request a ride by providing:

   * Pickup location
   * Drop location
   * Ride type
2. System should find an appropriate driver.
3. Driver can accept or reject the ride.
4. Once accepted, the trip becomes active.
5. Customer can cancel the trip.
6. Driver can complete the trip.
7. Once completed:

   * Fare should be calculated.
   * Payment should be processed.
   * Receipt should be generated.
   * Customer should be notified.

### Additional requirements

The system should support different:

* **Ride types** — e.g. Economy, Premium
* **Payment methods** — e.g. Credit Card, UPI
* **Notification channels** — e.g. Email, SMS, Push

And assume that more ride types, payment methods, and notification channels can be added in the future.

---

# Your task

Don't worry about making the design perfect.

Walk me through your thinking **as if you're answering an interviewer**.

I'd like you to cover:

### 1. Identify the main objects/classes

For example:

```text
Customer
Driver
Trip
...
```

But **you decide the complete list**.

### 2. Responsibilities

Tell me what each important class should be responsible for.

For example:

> "Trip should manage the lifecycle of a trip..."

Explain your reasoning.

### 3. Interface vs Class vs Service

For the important components, tell me:

> "I would use an interface here because..."

or

> "I would use a class here because..."

or

> "This should be a service because..."

### 4. Composition vs Inheritance

Tell me where you would use either and **why**.

### 5. SOLID

You don't need to force all five principles.

Tell me where you naturally see:

* SRP
* OCP
* LSP
* ISP
* DIP

### 6. Design patterns

**This is the important part.**

I won't tell you which patterns to use.

If you think a pattern is useful, say:

> "I would use Strategy here because..."

or:

> "I would use Observer here because..."

And also tell me **why you wouldn't use another approach** if relevant.

### 7. High-level flow

Explain something like:

```text
Customer
   ↓
Request Ride
   ↓
Find Driver
   ↓
Driver Accepts
   ↓
Trip Active
   ↓
Trip Completed
   ↓
Fare Calculation
   ↓
Payment
   ↓
Receipt + Notification
```

But again, **you decide the actual flow**.

---

# Do you need to write code?

### ❌ Not initially.

For this phase, **your design explanation is more important than code**.

Your first answer should be something like an interview whiteboard discussion:

> "First, I would identify these entities..."

> "I think Trip should be a class because..."

> "Payment should use an interface because different implementations are possible..."

> "For notifications, I would use..."

Then explain the flow.

You **can use pseudocode or small code snippets** if they help explain your design, but don't spend time writing complete TypeScript classes yet.

---

## After you give me your design

I'll review it in this format:

**1. What you got right** ✅
**2. What I'd change** ⚠️
**3. Missing considerations**
**4. SOLID evaluation**
**5. Pattern evaluation**
**6. Better architecture/design**
**7. Interview-ready answer**
**8. Whether code is worth writing**

Then, if the design is good, **I'll ask you to implement the important parts in TypeScript**.

This gives us two stages:

```text
Stage 1
Design thinking
    ↓
Stage 2
Code implementation
```

That's much closer to how a real senior LLD interview works.

### One important rule for this phase

**I will not immediately correct you if you choose a questionable pattern.**

I'll first let you complete your design, then we'll discuss whether the pattern was actually justified.

That way we're testing whether you can **discover the design**, rather than simply recognize a pattern from its name.

Go ahead and give me your design for **Ride Booking System** as if you're sitting in the interview.

---
### My Thoughts That Time -
So for this ride booking system problem, the low-level design. So, like, the main objects will be rider, driver, ride, or you can say trip, ride or trip. And the major interface which will be used is payment and notification, because these both have multiple support. Like, we can do payment by multiple providers, we can send multiple types of notifications. So this interface face will be implemented by different providers for the messaging and for the payments. And those objects can be used as a composition to send notification or do the payments. So for that, we will have two services as well, one will be the payment service, second will be the notification service. So by doing this interface and these services, we will achieve polymorphism and single responsibility technique, and we will also implement like that. And for services, so I said that there will be two, payment service and notification service. And we will also have for the ride class, we will follow the state pattern where we manage the status of the ride. So the status of the ride can be requested, accepted, driver in the route, and driver arrived, trip in progress, and completed. And this class will have different functions to support the cancellation flow of the ride and setting different status of the ride as the ride cycle goes. So yeah, we will follow that state pattern for ride so that it will be easy and it will achieve the single responsibility thing. And via payment service, we are already achieving the composition, and we can, for paying out, we can create the required object while calling the payment service, by which we can achieve the... we can do it via dependency injection. Or if the project requires that the request come from the front end and have this payment-related information, and then we can create this object within a new factory class, which will be making the payment method-related object to use in the payment service. But I will choose the dependency injection because it will give us the flexibility. And so I will not use factory pattern for now. And yeah, this will be those major parts. And while booking a ride, we can also use a proxy pattern as well, so that we can authenticate the rider. And then only rider will be able to book. So we can achieve that via proxy pattern, and we can achieve that lazy loading so that we do not need to create the object. And for the rider class, the properties will be name, phone number, user ID. This is for now only, because we can have multiple properties, but this will be the required at least. And for driver, we have the name, license plate, phone number, and photo URL if it is there. So these will be the required to show this information in the UI so that we can show the license plate of the driver and phone number and photo URL. So yeah, we can achieve like that. And we will have some type, ride types will be economy and premium. And we will have this payment method, like credit card and UPI. So for that, we are already using the interface to decide which payment method to use via the dependency injection. And yeah, mostly I think this ride booking system with the flexibility, we can use state pattern, proxy pattern, factory pattern I am not choosing for right now because I am using the dependency injection. And if making a ride is a big object, like the ride will have the pickup information and dropoff information, the ride will have name, ride will have the URL of the... so there will be lots of properties of a ride. So if making a ride via constructor is very, very complicated, then we can use the builder pattern as well to build a proper ride object. So yeah, builder pattern, we should use it. And payment services and the notification services are extendable, so in the future if something else comes, we can add it. So we are achieving also open and closed principle. So major pattern will be this one only: builder pattern, proxy pattern, state pattern, and technique for the payment or the notification services, we will use the composition thing. So yeah, that will be my short first thought of this problem to solve it.

---
