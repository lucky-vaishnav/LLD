# Integrated LLD Problem 1 — Ride Booking System

## 1. Overall architecture

I'd structure it roughly like this:

```text
                    Controller
                        │
                        ▼
                  RideService
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
     DriverMatching  Trip/State   FareCalculator
                                      │
                                      ▼
                                PaymentService
                                      │
                                      ▼
                               PaymentStrategy
                              /               \
                           Card               UPI

After successful completion
                        │
                        ▼
                 TripCompletedEvent
                        │
                Event/Observer Manager
                /          |          \
               ▼           ▼           ▼
          Receipt       Notification  Analytics
           Observer       Observer     Observer
```

The important thing is that we're **not trying to use every pattern**.

We're using patterns only where the problem naturally requires them.

---

# 2. Domain classes

## Rider

```ts
class Rider {
  constructor(
    private readonly id: string,
    private readonly name: string,
    private readonly phoneNumber: string,
  ) {}

  getId(): string {
    return this.id;
  }

  getName(): string {
    return this.name;
  }
}
```

Nothing complicated here.

We don't need an interface just because `Rider` exists.

---

## Driver

```ts
class Driver {
  constructor(
    private readonly id: string,
    private readonly name: string,
    private readonly phoneNumber: string,
    private readonly licensePlate: string,
  ) {}

  getId(): string {
    return this.id;
  }

  getName(): string {
    return this.name;
  }
}
```

Again, a simple class is enough.

---

# 3. Ride Type

We can use an enum:

```ts
enum RideType {
  ECONOMY = 'ECONOMY',
  PREMIUM = 'PREMIUM',
}
```

We don't need Strategy merely because there are two ride types.

If later ride types have different **pricing algorithms**, then Strategy becomes more interesting.

---

# 4. Trip State

This is where your State Pattern decision was strong.

```ts
interface TripState {
  accept(trip: Trip, driver: Driver): void;
  cancel(trip: Trip): void;
  start(trip: Trip): void;
  complete(trip: Trip): void;
}
```

### Requested state

```ts
class RequestedState implements TripState {
  accept(trip: Trip, driver: Driver): void {
    trip.assignDriver(driver);
    trip.setState(new AcceptedState());
  }

  cancel(trip: Trip): void {
    trip.setState(new CancelledState());
  }

  start(trip: Trip): void {
    throw new Error('Trip cannot start before driver accepts it');
  }

  complete(trip: Trip): void {
    throw new Error('Trip cannot be completed yet');
  }
}
```

### Accepted state

```ts
class AcceptedState implements TripState {
  accept(trip: Trip, driver: Driver): void {
    throw new Error('Trip already accepted');
  }

  cancel(trip: Trip): void {
    trip.setState(new CancelledState());
  }

  start(trip: Trip): void {
    trip.setState(new InProgressState());
  }

  complete(trip: Trip): void {
    throw new Error('Trip is not in progress');
  }
}
```

And you could continue with:

```ts
class InProgressState implements TripState {
  accept(trip: Trip, driver: Driver): void {
    throw new Error('Trip already started');
  }

  cancel(trip: Trip): void {
    // Business rule could allow/disallow cancellation here.
    throw new Error('Trip cannot be cancelled now');
  }

  start(trip: Trip): void {
    throw new Error('Trip already started');
  }

  complete(trip: Trip): void {
    trip.setState(new CompletedState());
  }
}
```

---

# 5. Trip

Now look at what Trip does.

```ts
class Trip {
  private state: TripState;
  private driver?: Driver;

  constructor(
    private readonly id: string,
    private readonly rider: Rider,
    private readonly pickup: string,
    private readonly destination: string,
    private readonly rideType: RideType,
  ) {
    this.state = new RequestedState();
  }

  accept(driver: Driver): void {
    this.state.accept(this, driver);
  }

  cancel(): void {
    this.state.cancel(this);
  }

  start(): void {
    this.state.start(this);
  }

  complete(): void {
    this.state.complete(this);
  }

  assignDriver(driver: Driver): void {
    this.driver = driver;
  }

  setState(state: TripState): void {
    this.state = state;
  }

  getDriver(): Driver | undefined {
    return this.driver;
  }
}
```

### Notice something important

Trip does **not** contain:

```ts
if (this.status === ...)
```

everywhere.

Instead:

```text
Trip
 ↓
Current State
 ↓
Allowed operation
 ↓
Transition
```

That's the actual value of State here.

---

# 6. Driver Matching

We identified earlier that this responsibility was missing from your first design.

```ts
class DriverMatchingService {
  findDriver(trip: Trip): Driver {
    // Find available drivers
    // Apply matching rules
    // Select appropriate driver

    return new Driver(
      'driver-1',
      'John',
      '123456789',
      'ABC-123',
    );
  }
}
```

This is a **service**, because it represents a business operation/workflow rather than an entity with its own lifecycle.

---

# 7. Payment

Now your interface + composition + DI idea comes in.

```ts
interface PaymentMethod {
  pay(amount: number): void;
}
```

Implementations:

```ts
class CreditCardPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Paid ${amount} using credit card`);
  }
}
```

```ts
class UpiPayment implements PaymentMethod {
  pay(amount: number): void {
    console.log(`Paid ${amount} using UPI`);
  }
}
```

Then:

```ts
class PaymentService {
  constructor(
    private readonly paymentMethod: PaymentMethod,
  ) {}

  processPayment(amount: number): void {
    this.paymentMethod.pay(amount);
  }
}
```

This is:

```text
PaymentService
      ↓
PaymentMethod
     / \
    /   \
 Card   UPI
```

**DIP + polymorphism + composition.**

---

# 8. Fare calculation

We can keep it simple initially:

```ts
class FareCalculator {
  calculate(trip: Trip): number {
    // Calculate based on distance,
    // ride type, surge, etc.

    return 500;
  }
}
```

If later the requirement becomes:

```text
Economy → algorithm A
Premium → algorithm B
Airport → algorithm C
```

then Strategy could naturally appear:

```text
FareCalculator
      ↓
FareStrategy
   /     |      \
Economy Premium Airport
```

But **we don't need to introduce Strategy prematurely**.

---

# 9. Observer

Now we reach the interesting part.

We don't want:

```ts
class Trip {
  complete() {
    receiptService.generate();
    notificationService.send();
    analyticsService.update();
  }
}
```

That would tightly couple Trip to multiple services.

Instead:

```ts
interface TripEventObserver {
  update(trip: Trip): void;
}
```

Observers:

```ts
class ReceiptObserver implements TripEventObserver {
  update(trip: Trip): void {
    console.log(`Generate receipt for ${trip}`);
  }
}
```

```ts
class NotificationObserver implements TripEventObserver {
  update(trip: Trip): void {
    console.log(`Notify rider`);
  }
}
```

```ts
class AnalyticsObserver implements TripEventObserver {
  update(trip: Trip): void {
    console.log(`Update analytics`);
  }
}
```

---

# 10. Observer manager

We need something to manage subscriptions.

```ts
class TripEventPublisher {
  private observers: TripEventObserver[] = [];

  subscribe(observer: TripEventObserver): void {
    this.observers.push(observer);
  }

  publish(trip: Trip): void {
    for (const observer of this.observers) {
      observer.update(trip);
    }
  }
}
```

Now:

```text
TripEventPublisher
       │
       ├── ReceiptObserver
       ├── NotificationObserver
       └── AnalyticsObserver
```

### This answers our follow-up question

> Should Trip know about all these services?

**No.**

Trip doesn't know:

```text
ReceiptService
NotificationService
AnalyticsService
```

It only participates in the completion/event flow.

---

# 11. Completion workflow

Now we can create the orchestration service.

```ts
class RideCompletionService {
  constructor(
    private readonly fareCalculator: FareCalculator,
    private readonly paymentService: PaymentService,
    private readonly eventPublisher: TripEventPublisher,
  ) {}

  completeTrip(trip: Trip): void {
    const fare = this.fareCalculator.calculate(trip);

    this.paymentService.processPayment(fare);

    trip.complete();

    this.eventPublisher.publish(trip);
  }
}
```

And now our flow becomes:

```text
completeTrip()
      │
      ├── Calculate Fare
      │
      ├── Process Payment
      │
      ├── Complete Trip
      │
      └── Publish TripCompleted
                 │
          ┌──────┼───────┐
          ↓      ↓       ↓
       Receipt Notify  Analytics
```

---

# 12. Important follow-up: What if payment fails?

This is exactly the kind of question an interviewer may ask.

Suppose:

```text
Fare = ₹500
Payment = FAILED
```

Should we execute:

```text
trip.complete()
```

?

Probably **no**, depending on the business requirement.

So:

```ts
class RideCompletionService {
  completeTrip(trip: Trip): void {
    const fare = this.fareCalculator.calculate(trip);

    this.paymentService.processPayment(fare);

    // Only after successful payment:
    trip.complete();

    this.eventPublisher.publish(trip);
  }
}
```

In a real system, `processPayment()` would return something meaningful:

```ts
const result = this.paymentService.processPayment(fare);

if (!result.success) {
  throw new Error('Payment failed');
}
```

Then completion doesn't happen.

This demonstrates an important principle:

> **Don't turn every step of a business workflow into an asynchronous event just because Observer is available.**

Core business invariants should remain explicit.

---

# 13. Where is DI?

At the composition/root level.

Conceptually:

```ts
const paymentMethod = new CreditCardPayment();

const paymentService = new PaymentService(
  paymentMethod,
);

const fareCalculator = new FareCalculator();

const publisher = new TripEventPublisher();

publisher.subscribe(
  new ReceiptObserver(),
);

publisher.subscribe(
  new NotificationObserver(),
);

publisher.subscribe(
  new AnalyticsObserver(),
);

const completionService = new RideCompletionService(
  fareCalculator,
  paymentService,
  publisher,
);
```

In NestJS, the framework's DI container would manage this wiring instead of us manually creating everything.

---

# 14. Where would Factory potentially appear?

Remember our earlier discussion.

Suppose the API says:

```text
paymentMethod = "UPI"
```

Something needs to resolve:

```text
UPI → UpiPayment
CARD → CreditCardPayment
```

At that point a Factory **could** be useful:

```ts
class PaymentMethodFactory {
  create(type: string): PaymentMethod {
    if (type === 'UPI') {
      return new UpiPayment();
    }

    if (type === 'CARD') {
      return new CreditCardPayment();
    }

    throw new Error('Unsupported payment method');
  }
}
```

Then:

```text
Request
   ↓
PaymentMethodFactory
   ↓
UPI
   ↓
PaymentService
```

But if NestJS configuration/DI already provides the appropriate implementation through some composition mechanism, we don't necessarily need this Factory.

**That's why I liked your original instinct not to force Factory into the design.**

---

# 15. Where would Proxy go?

You suggested authentication.

I'd still say:

```text
Controller
   ↓
Auth Guard / Middleware
   ↓
RideService
```

rather than introducing our own Proxy.

The pattern is technically applicable, but **the framework already provides a better mechanism**.

This is an important senior-level design decision:

> **A pattern being applicable doesn't mean it is the best implementation.**

---

# 16. Where would Builder go?

Same conclusion.

Our current Trip constructor is manageable:

```ts
new Trip(
  id,
  rider,
  pickup,
  destination,
  rideType,
);
```

So:

**Don't use Builder yet.**

If later Trip becomes:

```text
20 optional fields
+ validation
+ conditional fields
+ multiple construction configurations
```

then Builder becomes justified.

---

# Final pattern selection

For this particular problem, my final design would primarily use:

```text
State
→ Trip lifecycle

Observer
→ TripCompleted independent reactions

Strategy
→ Potentially fare calculation/payment behavior
   if algorithms genuinely differ

DI + Composition
→ Payment / Notification implementations
```

And **not necessarily**:

```text
Factory
Proxy
Builder
Facade
Decorator
...
```

unless additional requirements justify them.

---

# ⭐ What you should learn from Problem 1

The biggest lesson isn't the code.

It's this decision-making process:

```text
Requirement
    ↓
Responsibility
    ↓
Class / Interface / Service
    ↓
Composition
    ↓
SOLID
    ↓
Does a design problem exist?
    ↓
Choose pattern only if it solves that problem
```

You did reasonably well at identifying patterns. Your next improvement should be **assigning responsibilities before thinking about patterns**.

