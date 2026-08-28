# Topic 10 — Builder Pattern

This one is quite practical and will also help you understand **when a class constructor becomes too complicated**.

---

## 1. The problem

Suppose we have a `Trip` object.

It may have many properties:

```text
Trip
├── pickup
├── destination
├── passenger
├── vehicle
├── paymentMethod
├── coupon
├── scheduledTime
├── notes
├── priority
└── ...
```

A naive constructor could become:

```ts
new Trip(
  pickup,
  destination,
  passenger,
  vehicle,
  paymentMethod,
  coupon,
  scheduledTime,
  notes,
  priority
);
```

Now imagine some fields are optional.

You can quickly end up with:

```ts
new Trip(
  pickup,
  destination,
  passenger,
  undefined,
  paymentMethod,
  undefined,
  scheduledTime,
  undefined,
  true
);
```

That's difficult to read and easy to misuse.

---

# 2. Builder Pattern

Instead of constructing everything at once:

```ts
new Trip(...)
```

we build the object step by step:

```ts
const trip = new TripBuilder()
  .setPickup("Delhi")
  .setDestination("Jaipur")
  .setPassenger(passenger)
  .setVehicle(vehicle)
  .setPaymentMethod(paymentMethod)
  .setPriority(true)
  .build();
```

This is the main idea:

> **Builder separates the construction of a complex object from the final object itself.**

---

# 3. Basic implementation

```ts
class Trip {
  constructor(
    public pickup: string,
    public destination: string,
    public passenger: string,
    public vehicle?: string,
    public paymentMethod?: string,
    public priority?: boolean
  ) {}
}
```

Builder:

```ts
class TripBuilder {
  private pickup!: string;
  private destination!: string;
  private passenger!: string;
  private vehicle?: string;
  private paymentMethod?: string;
  private priority?: boolean;

  setPickup(pickup: string) {
    this.pickup = pickup;
    return this;
  }

  setDestination(destination: string) {
    this.destination = destination;
    return this;
  }

  setPassenger(passenger: string) {
    this.passenger = passenger;
    return this;
  }

  setVehicle(vehicle: string) {
    this.vehicle = vehicle;
    return this;
  }

  setPaymentMethod(paymentMethod: string) {
    this.paymentMethod = paymentMethod;
    return this;
  }

  setPriority(priority: boolean) {
    this.priority = priority;
    return this;
  }

  build(): Trip {
    return new Trip(
      this.pickup,
      this.destination,
      this.passenger,
      this.vehicle,
      this.paymentMethod,
      this.priority
    );
  }
}
```

Usage:

```ts
const trip = new TripBuilder()
  .setPickup("Delhi")
  .setDestination("Jaipur")
  .setPassenger("Lucky")
  .setVehicle("SUV")
  .setPaymentMethod("UPI")
  .setPriority(true)
  .build();
```

Notice the:

```ts
return this;
```

This enables **method chaining**.

---

# 4. Why not just use an object?

This is an important question.

You could simply do:

```ts
const trip = {
  pickup: "Delhi",
  destination: "Jaipur",
  passenger: "Lucky",
  vehicle: "SUV",
  paymentMethod: "UPI"
};
```

Or:

```ts
new Trip({
  pickup: "Delhi",
  destination: "Jaipur",
  passenger: "Lucky"
});
```

And in many modern TypeScript/Node.js applications, **that's perfectly reasonable**.

So don't say:

> "Builder should always be used for objects with many properties."

That's too broad.

Builder becomes more valuable when **construction itself has complexity**.

For example:

```text
Step 1 → validation
Step 2 → defaults
Step 3 → conditional configuration
Step 4 → construct object
```

---

# 5. Example of actual construction complexity

Suppose a trip has different rules:

```text
If scheduled:
    scheduledTime required

If priority:
    priorityFee required

If Uber:
    Uber-specific configuration required
```

Now the builder can centralize construction rules:

```ts
build(): Trip {
  if (!this.pickup) {
    throw new Error("Pickup is required");
  }

  if (!this.destination) {
    throw new Error("Destination is required");
  }

  if (this.priority && !this.paymentMethod) {
    throw new Error("Payment method required");
  }

  return new Trip(...);
}
```

Now the builder is doing more than just making the syntax pretty.

---

# 6. Builder vs Factory

This is important because you already learned Factory.

### Factory

> **Which object should I create?**

```text
NotificationFactory
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
Email SMS   Push
```

Factory is mainly about **choosing/creating the appropriate type**.

---

### Builder

> **How should I construct this complex object?**

```text
TripBuilder
   ↓
setPickup()
setDestination()
setVehicle()
setPayment()
setPriority()
   ↓
build()
   ↓
Trip
```

### Easy memory:

> **Factory → Which object?**

> **Builder → How to construct the object?**

---

# 7. Builder vs Template Method

Another useful connection.

### Template Method

Controls the **algorithm/workflow**:

```text
processRefund()
 ↓
validate
 ↓
calculate
 ↓
execute
```

### Builder

Controls **object construction**:

```text
TripBuilder
 ↓
set A
 ↓
set B
 ↓
set C
 ↓
build
```

So:

> **Template Method → how a process runs.**

> **Builder → how an object is constructed.**

---

# 8. One important interview point

Builder doesn't necessarily require a separate `Builder` class in every implementation.

In TypeScript, you might see:

```ts
Trip.builder()
  .setPickup(...)
  .setDestination(...)
  .build();
```

or a fluent factory/builder function.

The **pattern is about separating complex construction from the final object**, not about the exact class naming.

---

# 🧠 Your turn

Let's use our **Trip** example.

### Q1.

When would you choose Builder instead of simply doing:

```ts
new Trip({
  pickup,
  destination,
  passenger,
  vehicle
});
```

### Q2.

What is the main responsibility of `TripBuilder`?

### Q3.

What is the difference between **Builder and Factory**?

### Q4.

Suppose Trip has 15 fields, but all are simple and there are no construction rules or dependencies. Would you automatically use Builder? Why or why not?

### Q5.

Why do Builder methods commonly return `this`?

Answer in your own words, and I'll review it and turn the important parts into your interview notes.

---
### My Thoughts-
So for a question, so builder we can use whenever we think that object construction is becoming complicated, or we know the requirement and when we start the implementation of some functionality, we know that this will be complicated and there will be some different construction-related validation while we are making the object. That time we can choose the builder to use. Main responsibility of the builder is to properly create complex object and to avoid the misuse of the object. And builder is mostly about the object construction, and factory method is about which object to choose, not about how the object will be constructed or created. And if we know that there is no specific rule or dependency, and if still there are lots of fields, yeah we can use simply the constructor only while making the object. But yeah, if there are more fields, like in the future as well, if there can be any changes according to the thinking structure, we can use the builder pattern. But yeah, if we are sure that there is no dependency, we can use simply constructor. And builder method commonly returns this to have the reference of that object, so that the object will be updated, and we need a complete object.

---
Yes — **your understanding is good**, and importantly, you're thinking about Builder from a **practical design perspective**, not just "many fields = Builder." That's the right approach.

A few refinements:

## Q1. When to use Builder — ✅

Your answer is correct:

> Use Builder when object construction is complex or has construction-related rules, validation, optional configurations, dependencies, or multiple steps.

The important part is **construction complexity**, not simply the number of fields.

For example:

```ts
new Trip({
  pickup,
  destination,
  passenger,
  vehicle
});
```

is perfectly fine if construction is straightforward.

But:

```text
Trip
 ├── required fields
 ├── optional fields
 ├── conditional fields
 ├── validation
 ├── defaults
 └── different configurations
```

→ Builder becomes more attractive.

---

## Q2. Builder responsibility — ✅

You said:

> "properly create complex object and avoid misuse."

Good.

I'd phrase it more precisely:

> **Builder encapsulates the construction logic of a complex object and helps ensure the object is created in a valid configuration.**

That is a strong interview answer.

---

## Q3. Builder vs Factory — ✅ Excellent

Your distinction is exactly right:

```text
Factory
→ Which object/type should I create?

Builder
→ How should I construct this object?
```

For example:

```text
NotificationFactory
→ EmailNotification
→ SMSNotification
→ PushNotification
```

versus:

```text
TripBuilder
→ pickup
→ destination
→ vehicle
→ payment
→ build()
```

---

## Q4. Many fields but no complexity — ✅

You correctly said you **wouldn't automatically use Builder**.

This is important.

Suppose:

```ts
new User({
  firstName,
  lastName,
  email,
  phone,
  city,
  country,
  age,
  company
});
```

Even with 8–10 fields, if construction is straightforward:

> **An object/constructor is probably enough.**

Don't introduce Builder just because the class has many properties.

### Your practical rule:

> **Complex construction → consider Builder.**

> **Simple construction with many properties → object parameter/constructor may be enough.**

---

## Q5. Why `return this`? — Almost correct

You said:

> "to have the reference of that object"

That's basically right, but the main reason is **method chaining**.

For example:

```ts
builder
  .setPickup(...)
  .setDestination(...)
  .setVehicle(...)
  .setPaymentMethod(...)
  .build();
```

Each method:

```ts
setPickup(value) {
  this.pickup = value;
  return this;
}
```

returns the same Builder instance, allowing the next method to be called on it.

So the interview answer:

> **"Builder methods return `this` to support fluent method chaining."**

---

# 🧠 Notes for your notebook

### Builder Pattern

> **Separates complex object construction from the final object and allows the object to be built step-by-step.**

### Use when:

* Construction is complex
* Many optional configurations
* Conditional construction rules
* Validation during construction
* Multiple construction steps
* Want readable/fluent construction

### Don't use just because:

> ❌ "The object has many fields."

### Builder vs Factory

```text
Factory
→ Which object should I create?

Builder
→ How should I construct it?
```

### Builder vs Constructor

```text
Simple object
→ Constructor / object parameter

Complex construction
→ Builder
```

### `return this`

> **Enables fluent method chaining.**

```ts
builder
  .setA()
  .setB()
  .setC()
  .build();
```

---

### One senior-level nuance

You mentioned:

> "in the future there can be changes"

That's **not by itself a strong reason** to introduce Builder today.

Avoid designing for hypothetical complexity.

A better principle is:

> **Introduce Builder when the current construction complexity justifies it, or when the known requirements clearly indicate that construction will be complex.**

That's the kind of reasoning that will keep your LLD designs from becoming over-engineered.
