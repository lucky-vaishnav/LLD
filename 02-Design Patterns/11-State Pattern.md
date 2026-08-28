This is particularly useful for backend systems because objects often behave differently depending on their **current state**.

# Topic 11 — State Pattern

## 1. The problem

Imagine a parking booking.

A booking can have states:

```text
PENDING
CONFIRMED
CANCELLED
COMPLETED
```

And the allowed behavior changes based on the current state.

For example:

```text
PENDING
→ confirm()
→ cancel()

CONFIRMED
→ cancel()
→ complete()

COMPLETED
→ no further modification

CANCELLED
→ no further modification
```

A simple implementation might become:

```ts id="6i5q9a"
if (status === "PENDING") {
   // ...
} else if (status === "CONFIRMED") {
   // ...
} else if (status === "CANCELLED") {
   // ...
}
```

Then every operation starts accumulating conditions:

```ts id="c2l5z3"
cancel() {
  if (status === "PENDING") {
    // ...
  } else if (status === "CONFIRMED") {
    // ...
  } else if (status === "COMPLETED") {
    // ...
  }
}
```

And:

```ts id="1grgaj"
complete() {
  if (status === "PENDING") {
    // ...
  } else if (status === "CONFIRMED") {
    // ...
  } else if (status === "CANCELLED") {
    // ...
  }
}
```

As states and operations increase, this becomes difficult to maintain.

---

# 2. State Pattern

The idea is:

> **Move state-specific behavior into separate state objects.**

Instead of:

```text
Booking
  ↓
if PENDING
if CONFIRMED
if CANCELLED
```

we have:

```text id="h3o4y4"
Booking
   ↓
Current State
   ├── PendingState
   ├── ConfirmedState
   ├── CancelledState
   └── CompletedState
```

The Booking delegates behavior to its current state.

---

# 3. Basic structure

We define a State interface:

```ts id="e4k6r9"
interface BookingState {
  confirm(booking: Booking): void;
  cancel(booking: Booking): void;
  complete(booking: Booking): void;
}
```

Then:

```ts id="yrq1lj"
class PendingState implements BookingState {

  confirm(booking: Booking) {
    console.log("Booking confirmed");
    booking.setState(new ConfirmedState());
  }

  cancel(booking: Booking) {
    console.log("Booking cancelled");
    booking.setState(new CancelledState());
  }

  complete(booking: Booking) {
    throw new Error("Pending booking cannot be completed");
  }
}
```

Confirmed:

```ts id="1v6dsy"
class ConfirmedState implements BookingState {

  confirm(booking: Booking) {
    throw new Error("Already confirmed");
  }

  cancel(booking: Booking) {
    console.log("Booking cancelled");
    booking.setState(new CancelledState());
  }

  complete(booking: Booking) {
    console.log("Booking completed");
    booking.setState(new CompletedState());
  }
}
```

---

# 4. Context

The `Booking` is called the **Context**.

```ts id="1kq3ip"
class Booking {

  private state: BookingState;

  constructor() {
    this.state = new PendingState();
  }

  setState(state: BookingState) {
    this.state = state;
  }

  confirm() {
    this.state.confirm(this);
  }

  cancel() {
    this.state.cancel(this);
  }

  complete() {
    this.state.complete(this);
  }
}
```

Now the caller simply does:

```ts id="bq2rqn"
booking.confirm();
booking.cancel();
booking.complete();
```

The caller doesn't need:

```ts
if (booking.status === ...)
```

The current state handles the behavior.

---

# 5. The important difference from Strategy

This is where State can initially look very similar to Strategy.

Both can use:

* Interface
* Polymorphism
* Composition
* Dependency injection

But their **intent is different**.

### Strategy

The caller/system chooses an algorithm.

```text id="j5l3i6"
PaymentService
      ↓
PaymentStrategy
      ├── Card
      ├── UPI
      └── Wallet
```

Question:

> **"Which strategy should I use?"**

---

### State

The object's **current state determines its behavior**, and the state can change during the object's lifecycle.

```text id="0lqz1a"
Booking
   ↓
Pending
   ↓ confirm()
Confirmed
   ↓ complete()
Completed
```

Question:

> **"What should this object do in its current state?"**

### 🧠 Easy memory

> **Strategy → behavior is selected.**

> **State → behavior changes because state changes.**

---

# 6. Real-world example from your project domain

Think about a trip:

```text
REQUESTED
   ↓
CONFIRMED
   ↓
IN_PROGRESS
   ↓
COMPLETED
```

Different operations are valid in different states.

For example:

```text
REQUESTED
→ cancel()

CONFIRMED
→ cancel()
→ start()

IN_PROGRESS
→ complete()

COMPLETED
→ generateReceipt()
```

If you put everything inside:

```ts
if (trip.status === ...)
```

the Trip class can become huge.

State Pattern lets each state own its behavior.

---

# 7. When should you NOT use State?

This is important.

Don't use State just because you have:

```ts
status = "ACTIVE"
```

If behavior doesn't really change based on state, a simple enum/status field is enough.

For example:

```ts
user.status = "ACTIVE";
```

If all operations work exactly the same regardless of status, State Pattern gives you unnecessary complexity.

Use State when:

> **Different states meaningfully change what operations are allowed to do or how they behave.**

---

# 🧠 Your turn

Let's use the **Trip lifecycle**:

```text
REQUESTED
CONFIRMED
IN_PROGRESS
COMPLETED
CANCELLED
```

### Q1.

What would be the **Context** in the State Pattern?

### Q2.

What would the `State` interface contain?

### Q3.

Give me 2–3 concrete state classes.

### Q4.

Why would State Pattern be better than having `if/else` checks throughout `Trip`?

### Q5.

What is the **main difference between State and Strategy**?

### Q6.

If a Trip has a `status` field but its behavior doesn't change based on that status, should we use State Pattern?

Answer in your own words as usual. Then I'll review it and convert the important parts into your interview notes.

---
### My Thoughts-
Context for this problem will be trip, and state interface will contain like functions like cancel, complete, and confirm, this type of function. And this will implement the trip status, like display trip status, and concrete state classes will be implementing this trip state interface. Yeah, it is using state pattern is better than using if-else, because if-else will be complicated to handle, and as the code will grow, we need to put maybe this type of state-level validation or state-level checking in multiple parts of code. So I'm not saying that it is about shared code. Shared code we can achieve by something else as well, but it is the state management which is important there. And if the code will be at one responsible component, then the validation will be there, and so it will be single responsibility type, and there will be no mistake if we do it in the code hardcoding each time. So yeah, and it can be centralized as well. Different between state and strategy: strategy is about which behavior to choose or select, and state is that what is the state of the object, or what is the behavior of the object which changes according to the state and validation. So if trip has just a status, like one or two status only, like active, inactive, then I think there is no sense to use state pattern, because it is mostly about handling the lifecycle. And if that status is related to the state change of the object, then we should think about the state pattern, I think.

---
Yes — **your understanding is correct**, and your answer shows you're getting the important distinction between **State as lifecycle/behavior management** versus simply having a `status` field.

A few refinements will make it stronger.

## Q1. Context — ✅

You said:

> Context will be Trip.

Correct.

```text
Trip
 ↓
Current TripState
```

The `Trip` owns the current state and delegates state-dependent operations to it.

---

## Q2. State interface — ✅

Correct idea.

For example:

```ts
interface TripState {
  confirm(trip: Trip): void;
  cancel(trip: Trip): void;
  start(trip: Trip): void;
  complete(trip: Trip): void;
}
```

One correction: the State interface isn't mainly for **displaying trip status**.

Its main responsibility is:

> **Define the operations whose behavior can vary depending on the current state.**

---

## Q3. Concrete states

For example:

```text
RequestedState
ConfirmedState
InProgressState
CompletedState
CancelledState
```

Each state determines what operations are valid.

For example:

```text
REQUESTED
→ confirm()
→ cancel()

CONFIRMED
→ start()
→ cancel()

IN_PROGRESS
→ complete()

COMPLETED
→ no further lifecycle changes
```

---

## Q4. Why State instead of `if/else` — ✅

Your reasoning is good.

The stronger interview explanation is:

> **As states and state-dependent operations increase, conditional logic becomes scattered across the object. State Pattern encapsulates state-specific behavior inside separate state classes, keeping the Context simpler and making state transitions easier to manage and extend.**

For example, instead of:

```ts
if (trip.status === "CONFIRMED") ...
else if (trip.status === "IN_PROGRESS") ...
```

in many methods, we have:

```text
Trip
 ↓
ConfirmedState
```

or:

```text
Trip
 ↓
InProgressState
```

---

## Q5. State vs Strategy — ✅

Your answer is correct.

I'd make it interview-ready:

> **Strategy allows us to choose between interchangeable algorithms/behaviors, whereas State allows an object's behavior to change as its internal state changes.**

### Strategy

```text
PaymentService
      ↓
PaymentStrategy
 ┌────┼─────┐
 ↓    ↓     ↓
Card UPI Wallet
```

We select the strategy.

### State

```text
Trip
 ↓
Requested
 ↓ confirm()
Confirmed
 ↓ start()
InProgress
 ↓ complete()
Completed
```

The object's state changes, and therefore its behavior changes.

---

# Q6. Simple status — ✅

This was also correct.

If we only have:

```ts
trip.status = "ACTIVE";
```

and perhaps:

```ts
trip.status = "INACTIVE";
```

but behavior doesn't meaningfully change, **don't use State Pattern**.

Just use the status.

Your statement:

> "If that status is related to the state change of the object, then we should think about State Pattern."

is good.

I'd make it even more precise:

> **Use State when different states meaningfully change the object's available operations or behavior, not merely because the object has a status field.**

---

# ⭐ One important point for your notes

You mentioned **centralization**, which is good, but don't make "centralization" the primary reason.

The primary reason is:

> **Encapsulating state-specific behavior and transitions.**

Centralization is a resulting benefit.

---

# 🧠 Final interview notes

### State Pattern

> **Allows an object to change its behavior when its internal state changes by encapsulating state-specific behavior in separate state objects.**

### Use when:

* Object has multiple meaningful states
* Operations behave differently in different states
* State transitions are part of the object's lifecycle
* `if/else` or `switch` logic based on state is becoming complex

### Don't use when:

* There is only a simple status field
* Behavior doesn't really change between states
* State-specific logic is trivial

### State vs Strategy

```text
Strategy
→ Which behavior/algorithm should I choose?

State
→ What should the object do in its current state?
```

### Key mental model

> **Strategy = behavior is selected externally.**

> **State = behavior changes as the object's state changes.**

And one subtle point: State and Strategy can look **almost identical structurally**—both commonly use an interface + composition + polymorphism. **The intent is what distinguishes them.** That's exactly the kind of distinction interviewers often look for.
