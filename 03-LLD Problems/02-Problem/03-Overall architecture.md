 First the overall structure and flow, then a clean Node.js/TypeScript implementation showing the important design decisions without overengineering.

# Problem 2 — Food Delivery Order System

## 1. Overall Architecture

```text
                         ┌─────────────────┐
                         │    Customer     │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │   OrderService  │
                         └────────┬────────┘
                                  │
             ┌────────────────────┼─────────────────────┐
             │                    │                     │
             ▼                    ▼                     ▼
      ┌─────────────┐      ┌──────────────┐      ┌──────────────┐
      │    Cart     │      │ FareCalculator│      │PaymentService│
      └─────────────┘      └──────────────┘      └───────┬──────┘
                                                          │
                                                          ▼
                                                 ┌─────────────────┐
                                                 │ PaymentMethod   │
                                                 ├─────────────────┤
                                                 │ CardPayment     │
                                                 │ UpiPayment      │
                                                 │ WalletPayment   │
                                                 └─────────────────┘

                         Order Lifecycle
                               │
                               ▼
                    ┌─────────────────────┐
                    │        Order        │
                    │                     │
                    │ currentState        │
                    └──────────┬──────────┘
                               │
                               ▼
                         ┌─────────────┐
                         │ OrderState  │
                         ├─────────────┤
                         │ Placed      │
                         │ Accepted    │
                         │ Preparing   │
                         │ Ready       │
                         │ OutDelivery │
                         │ Delivered   │
                         │ Cancelled   │
                         │ Rejected    │
                         └─────────────┘

                         Order Events
                               │
                               ▼
                      ┌─────────────────┐
                      │ EventPublisher  │
                      └────────┬────────┘
                               │
             ┌─────────────────┼─────────────────┐
             ▼                 ▼                 ▼
        ┌──────────┐      ┌──────────┐      ┌──────────┐
        │  Email   │      │   SMS    │      │   Push   │
        └──────────┘      └──────────┘      └──────────┘
```

---

# 2. Main Entities

### Customer

Responsible for customer information.

```text
Customer
 ├── id
 ├── name
 ├── phone
 └── email
```

### Restaurant

```text
Restaurant
 ├── id
 ├── name
 ├── rating
 └── menu
```

### Menu

Contains menu items.

```text
Menu
 └── MenuItem[]
```

### MenuItem

```text
MenuItem
 ├── id
 ├── name
 ├── price
 └── available
```

### Cart

Temporary collection of items selected by customer.

```text
Cart
 ├── customer
 ├── restaurant
 └── CartItem[]
```

### Order

Represents the actual placed order.

Important distinction:

```text
CartItem != OrderItem
```

When an order is created, we should snapshot the relevant information into `OrderItem`.

For example:

```text
OrderItem
 ├── menuItemId
 ├── name
 ├── price
 └── quantity
```

If the restaurant later changes a pizza from ₹300 → ₹350, the historical order should still show ₹300.

---

# 3. Services

```text
OrderService
PaymentService
FareCalculator
DeliveryService
NotificationService
```

### Responsibilities

| Component             | Responsibility                      |
| --------------------- | ----------------------------------- |
| `OrderService`        | Place/order lifecycle orchestration |
| `PaymentService`      | Execute payment                     |
| `FareCalculator`      | Calculate order total               |
| `DeliveryService`     | Delivery-partner workflow           |
| `NotificationService` | Notification abstraction            |
| `EventPublisher`      | Publish order events                |

---

# 4. Patterns Used

### State

For:

```text
PLACED
ACCEPTED
PREPARING
READY
OUT_FOR_DELIVERY
DELIVERED
CANCELLED
REJECTED
```

Because valid operations depend on current order state.

### Strategy + Composition/DI

For:

```text
PaymentMethod
```

Implementations:

```text
CardPayment
UpiPayment
WalletPayment
```

### Observer

For:

```text
OrderPlaced
OrderAccepted
OrderReady
OrderDelivered
OrderCancelled
```

Multiple independent components can react to an event.

### Factory — optional

Only introduce it if runtime input requires creation/selection:

```text
paymentType = "UPI"
       ↓
PaymentFactory
       ↓
UpiPayment
```

It isn't mandatory just because we have multiple payment implementations.

### Facade — optional

Could be used if we want to expose a simplified interface over a complicated ordering subsystem.

For this design, `OrderService` is already sufficient, so I wouldn't force a Facade.

---

# 5. TypeScript Implementation

## Entities

```typescript
// customer.ts

export class Customer {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly phone: string,
    public readonly email: string,
  ) {}
}
```

```typescript
// menu-item.ts

export class MenuItem {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly price: number,
    private available: boolean = true,
  ) {}

  isAvailable(): boolean {
    return this.available;
  }

  markUnavailable(): void {
    this.available = false;
  }

  markAvailable(): void {
    this.available = true;
  }
}
```

```typescript
// menu.ts

import { MenuItem } from "./menu-item";

export class Menu {
  private readonly items = new Map<string, MenuItem>();

  addItem(item: MenuItem): void {
    this.items.set(item.id, item);
  }

  removeItem(itemId: string): void {
    this.items.delete(itemId);
  }

  getItem(itemId: string): MenuItem | undefined {
    return this.items.get(itemId);
  }

  getItems(): MenuItem[] {
    return [...this.items.values()];
  }
}
```

```typescript
// restaurant.ts

import { Menu } from "./menu";

export class Restaurant {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly rating: number,
    public readonly phone: string,
    public readonly menu: Menu,
  ) {}
}
```

---

# 6. Cart

```typescript
// cart.ts

import { MenuItem } from "./menu-item";
import { Restaurant } from "./restaurant";

export interface CartItem {
  menuItem: MenuItem;
  quantity: number;
}

export class Cart {
  private readonly items = new Map<string, CartItem>();

  constructor(
    public readonly customerId: string,
    public readonly restaurant: Restaurant,
  ) {}

  addItem(menuItem: MenuItem, quantity: number): void {
    if (!menuItem.isAvailable()) {
      throw new Error("Menu item is unavailable");
    }

    const existing = this.items.get(menuItem.id);

    if (existing) {
      existing.quantity += quantity;
    } else {
      this.items.set(menuItem.id, {
        menuItem,
        quantity,
      });
    }
  }

  removeItem(menuItemId: string): void {
    this.items.delete(menuItemId);
  }

  getItems(): CartItem[] {
    return [...this.items.values()];
  }

  clear(): void {
    this.items.clear();
  }
}
```

---

# 7. Order + OrderItem

This is an important design decision.

**Don't keep the Cart itself as the Order.**

Once the customer places the order:

```text
Cart
 ↓
Order
 ↓
OrderItem snapshot
```

```typescript
// order-item.ts

export class OrderItem {
  constructor(
    public readonly menuItemId: string,
    public readonly name: string,
    public readonly price: number,
    public readonly quantity: number,
  ) {}

  getTotal(): number {
    return this.price * this.quantity;
  }
}
```

---

# 8. Order State

```typescript
// order-state.ts

import { Order } from "./order";

export interface OrderState {
  accept(order: Order): void;
  reject(order: Order): void;
  startPreparing(order: Order): void;
  markReady(order: Order): void;
  pickUp(order: Order): void;
  deliver(order: Order): void;
  cancel(order: Order): void;
}
```

---

## Base State

To avoid duplicating invalid-operation errors:

```typescript
// base-order-state.ts

import { Order } from "./order";
import { OrderState } from "./order-state";

export abstract class BaseOrderState implements OrderState {
  accept(order: Order): void {
    throw new Error(`Cannot accept order in ${order.getStatus()} state`);
  }

  reject(order: Order): void {
    throw new Error(`Cannot reject order in ${order.getStatus()} state`);
  }

  startPreparing(order: Order): void {
    throw new Error(`Cannot start preparing in ${order.getStatus()} state`);
  }

  markReady(order: Order): void {
    throw new Error(`Cannot mark ready in ${order.getStatus()} state`);
  }

  pickUp(order: Order): void {
    throw new Error(`Cannot pick up in ${order.getStatus()} state`);
  }

  deliver(order: Order): void {
    throw new Error(`Cannot deliver in ${order.getStatus()} state`);
  }

  cancel(order: Order): void {
    throw new Error(`Cannot cancel in ${order.getStatus()} state`);
  }
}
```

---

# 9. Concrete States

```typescript
// placed-state.ts

import { BaseOrderState } from "./base-order-state";
import { Order } from "./order";
import { AcceptedState } from "./accepted-state";
import { RejectedState } from "./rejected-state";
import { CancelledState } from "./cancelled-state";

export class PlacedState extends BaseOrderState {
  accept(order: Order): void {
    order.setState(new AcceptedState());
  }

  reject(order: Order): void {
    order.setState(new RejectedState());
  }

  cancel(order: Order): void {
    order.setState(new CancelledState());
  }
}
```

```typescript
// accepted-state.ts

import { BaseOrderState } from "./base-order-state";
import { Order } from "./order";
import { PreparingState } from "./preparing-state";

export class AcceptedState extends BaseOrderState {
  startPreparing(order: Order): void {
    order.setState(new PreparingState());
  }
}
```

```typescript
// preparing-state.ts

import { BaseOrderState } from "./base-order-state";
import { Order } from "./order";
import { ReadyState } from "./ready-state";

export class PreparingState extends BaseOrderState {
  markReady(order: Order): void {
    order.setState(new ReadyState());
  }
}
```

```typescript
// ready-state.ts

import { BaseOrderState } from "./base-order-state";
import { Order } from "./order";
import { OutForDeliveryState } from "./out-for-delivery-state";

export class ReadyState extends BaseOrderState {
  pickUp(order: Order): void {
    order.setState(new OutForDeliveryState());
  }
}
```

```typescript
// out-for-delivery-state.ts

import { BaseOrderState } from "./base-order-state";
import { Order } from "./order";
import { DeliveredState } from "./delivered-state";

export class OutForDeliveryState extends BaseOrderState {
  deliver(order: Order): void {
    order.setState(new DeliveredState());
  }
}
```

Terminal states:

```typescript
// delivered-state.ts

import { BaseOrderState } from "./base-order-state";

export class DeliveredState extends BaseOrderState {}
```

```typescript
// cancelled-state.ts

import { BaseOrderState } from "./base-order-state";

export class CancelledState extends BaseOrderState {}
```

```typescript
// rejected-state.ts

import { BaseOrderState } from "./base-order-state";

export class RejectedState extends BaseOrderState {}
```

---

# 10. Order

The `Order` owns its current state.

```typescript
// order.ts

import { Customer } from "./customer";
import { Restaurant } from "./restaurant";
import { OrderItem } from "./order-item";
import { OrderState } from "./order-state";
import { PlacedState } from "./placed-state";

export class Order {
  private state: OrderState;

  constructor(
    public readonly id: string,
    public readonly customer: Customer,
    public readonly restaurant: Restaurant,
    public readonly items: OrderItem[],
    public readonly totalAmount: number,
  ) {
    this.state = new PlacedState();
  }

  setState(state: OrderState): void {
    this.state = state;
  }

  getStatus(): string {
    return this.state.constructor.name;
  }

  accept(): void {
    this.state.accept(this);
  }

  reject(): void {
    this.state.reject(this);
  }

  startPreparing(): void {
    this.state.startPreparing(this);
  }

  markReady(): void {
    this.state.markReady(this);
  }

  pickUp(): void {
    this.state.pickUp(this);
  }

  deliver(): void {
    this.state.deliver(this);
  }

  cancel(): void {
    this.state.cancel(this);
  }
}
```

Notice the important point:

**Nobody outside the Order/State hierarchy directly changes `order.status`.**

That's the actual value of State.

---

# 11. Fare Calculator

```typescript
// fare-calculator.ts

import { OrderItem } from "./order-item";

export class FareCalculator {
  calculate(items: OrderItem[]): number {
    return items.reduce(
      (total, item) => total + item.getTotal(),
      0,
    );
  }
}
```

If later we introduce:

```text
Delivery fee
Platform fee
Tax
Discount
Surge
Coupon
```

we can evolve this separately rather than making `Order` responsible for pricing.

---

# 12. Payment — Strategy + DI

```typescript
// payment-method.ts

export interface PaymentMethod {
  pay(amount: number): Promise<string>;
}
```

```typescript
// card-payment.ts

import { PaymentMethod } from "./payment-method";

export class CardPayment implements PaymentMethod {
  async pay(amount: number): Promise<string> {
    console.log(`Charging card: ${amount}`);

    return `card-txn-${Date.now()}`;
  }
}
```

```typescript
// upi-payment.ts

import { PaymentMethod } from "./payment-method";

export class UpiPayment implements PaymentMethod {
  async pay(amount: number): Promise<string> {
    console.log(`Processing UPI payment: ${amount}`);

    return `upi-txn-${Date.now()}`;
  }
}
```

```typescript
// wallet-payment.ts

import { PaymentMethod } from "./payment-method";

export class WalletPayment implements PaymentMethod {
  async pay(amount: number): Promise<string> {
    console.log(`Deducting wallet balance: ${amount}`);

    return `wallet-txn-${Date.now()}`;
  }
}
```

Payment service:

```typescript
// payment-service.ts

import { PaymentMethod } from "./payment-method";

export class PaymentService {
  constructor(
    private readonly paymentMethod: PaymentMethod,
  ) {}

  async processPayment(amount: number): Promise<string> {
    return this.paymentMethod.pay(amount);
  }
}
```

This is **composition + dependency injection**.

We can change:

```typescript
new PaymentService(new CardPayment());
```

to:

```typescript
new PaymentService(new UpiPayment());
```

without modifying `PaymentService`.

---

# 13. Notification

```typescript
// notification-channel.ts

export interface NotificationChannel {
  send(message: string, recipient: string): Promise<void>;
}
```

```typescript
// email-notification.ts

import { NotificationChannel } from "./notification-channel";

export class EmailNotification implements NotificationChannel {
  async send(message: string, recipient: string): Promise<void> {
    console.log(`Email → ${recipient}: ${message}`);
  }
}
```

```typescript
// sms-notification.ts

import { NotificationChannel } from "./notification-channel";

export class SmsNotification implements NotificationChannel {
  async send(message: string, recipient: string): Promise<void> {
    console.log(`SMS → ${recipient}: ${message}`);
  }
}
```

```typescript
// push-notification.ts

import { NotificationChannel } from "./notification-channel";

export class PushNotification implements NotificationChannel {
  async send(message: string, recipient: string): Promise<void> {
    console.log(`Push → ${recipient}: ${message}`);
  }
}
```

---

# 14. Observer

Rather than making `Order` know about:

```text
EmailService
SmsService
PushService
AnalyticsService
```

we keep it independent.

```typescript
// order-event.ts

import { Order } from "./order";

export enum OrderEventType {
  ORDER_PLACED = "ORDER_PLACED",
  ORDER_ACCEPTED = "ORDER_ACCEPTED",
  ORDER_READY = "ORDER_READY",
  ORDER_DELIVERED = "ORDER_DELIVERED",
  ORDER_CANCELLED = "ORDER_CANCELLED",
}

export interface OrderEvent {
  type: OrderEventType;
  order: Order;
}
```

Observer:

```typescript
// order-observer.ts

import { OrderEvent } from "./order-event";

export interface OrderObserver {
  update(event: OrderEvent): Promise<void>;
}
```

Publisher:

```typescript
// order-event-publisher.ts

import { OrderEvent } from "./order-event";
import { OrderObserver } from "./order-observer";

export class OrderEventPublisher {
  private readonly observers: OrderObserver[] = [];

  subscribe(observer: OrderObserver): void {
    this.observers.push(observer);
  }

  async publish(event: OrderEvent): Promise<void> {
    await Promise.all(
      this.observers.map(observer => observer.update(event)),
    );
  }
}
```

Now a notification observer:

```typescript
// notification-observer.ts

import {
  OrderEvent,
  OrderEventType,
} from "./order-event";

import { OrderObserver } from "./order-observer";
import { NotificationChannel } from "./notification-channel";

export class NotificationObserver implements OrderObserver {
  constructor(
    private readonly channel: NotificationChannel,
  ) {}

  async update(event: OrderEvent): Promise<void> {
    if (event.type === OrderEventType.ORDER_PLACED) {
      await this.channel.send(
        `Your order ${event.order.id} has been placed.`,
        event.order.customer.phone,
      );
    }
  }
}
```

We could now register:

```typescript
publisher.subscribe(
  new NotificationObserver(new SmsNotification()),
);

publisher.subscribe(
  new NotificationObserver(new EmailNotification()),
);
```

and both can react to the same event.

---

# 15. Order Service

This is the main application/use-case orchestration layer.

```typescript
// order-service.ts

import { Cart } from "./cart";
import { Customer } from "./customer";
import { FareCalculator } from "./fare-calculator";
import { Order } from "./order";
import { OrderItem } from "./order-item";
import { PaymentService } from "./payment-service";
import {
  OrderEventType,
} from "./order-event";
import { OrderEventPublisher } from "./order-event-publisher";

export class OrderService {
  constructor(
    private readonly fareCalculator: FareCalculator,
    private readonly paymentService: PaymentService,
    private readonly eventPublisher: OrderEventPublisher,
  ) {}

  async placeOrder(
    cart: Cart,
    customer: Customer,
  ): Promise<Order> {

    const cartItems = cart.getItems();

    if (cartItems.length === 0) {
      throw new Error("Cart is empty");
    }

    const orderItems = cartItems.map(
      item =>
        new OrderItem(
          item.menuItem.id,
          item.menuItem.name,
          item.menuItem.price,
          item.quantity,
        ),
    );

    const total = this.fareCalculator.calculate(orderItems);

    const order = new Order(
      crypto.randomUUID(),
      customer,
      cart.restaurant,
      orderItems,
      total,
    );

    await this.paymentService.processPayment(total);

    cart.clear();

    await this.eventPublisher.publish({
      type: OrderEventType.ORDER_PLACED,
      order,
    });

    return order;
  }

  async acceptOrder(order: Order): Promise<void> {
    order.accept();

    await this.eventPublisher.publish({
      type: OrderEventType.ORDER_ACCEPTED,
      order,
    });
  }

  async rejectOrder(order: Order): Promise<void> {
    order.reject();
  }

  async startPreparing(order: Order): Promise<void> {
    order.startPreparing();
  }

  async markReady(order: Order): Promise<void> {
    order.markReady();

    await this.eventPublisher.publish({
      type: OrderEventType.ORDER_READY,
      order,
    });
  }

  async pickUp(order: Order): Promise<void> {
    order.pickUp();
  }

  async deliver(order: Order): Promise<void> {
    order.deliver();

    await this.eventPublisher.publish({
      type: OrderEventType.ORDER_DELIVERED,
      order,
    });
  }

  async cancel(order: Order): Promise<void> {
    order.cancel();

    await this.eventPublisher.publish({
      type: OrderEventType.ORDER_CANCELLED,
      order,
    });
  }
}
```

---

# 16. Putting It Together

A simplified composition root could look like:

```typescript
const paymentMethod = new UpiPayment();

const paymentService = new PaymentService(
  paymentMethod,
);

const fareCalculator = new FareCalculator();

const eventPublisher = new OrderEventPublisher();

eventPublisher.subscribe(
  new NotificationObserver(
    new SmsNotification(),
  ),
);

eventPublisher.subscribe(
  new NotificationObserver(
    new EmailNotification(),
  ),
);

const orderService = new OrderService(
  fareCalculator,
  paymentService,
  eventPublisher,
);
```

Then:

```text id="2j4w0x"
Customer
   ↓
Cart
   ↓
OrderService.placeOrder()
   ↓
Create OrderItem snapshots
   ↓
FareCalculator
   ↓
PaymentService
   ↓
PaymentMethod
   ↓
Order created
   ↓
ORDER_PLACED event
   ↓
Observers
   ├── Email
   ├── SMS
   └── Push
```

---

# 17. Complete Lifecycle

The restaurant side:

```text id="9w8k4v"
PLACED
   │
   ├──────────────→ REJECTED
   │
   ▼
ACCEPTED
   │
   ▼
PREPARING
   │
   ▼
READY
   │
   ▼
OUT_FOR_DELIVERY
   │
   ▼
DELIVERED
```

Customer cancellation:

```text id="0d8j8h"
PLACED
   │
   └──────────────→ CANCELLED
```

The important thing is that **the State object determines whether each transition is valid**.

---

# 18. What We Deliberately Did NOT Add

This is equally important for your interview notes.

We didn't automatically add:

```text
Factory
Facade
Builder
Proxy
Adapter
Command
Singleton
```

Why?

Because there isn't currently a strong requirement for them.

If the interviewer asks:

> "Why didn't you use Factory for payment?"

Your answer:

> "PaymentMethod uses polymorphism and dependency injection. A Factory would only become useful if the concrete payment implementation needs to be selected or created dynamically from runtime input."

If asked:

> "Why no Facade?"

> "OrderService already represents the application-level ordering workflow. I would introduce a Facade only if I had a complex subsystem that needed to be exposed through a simpler interface."

That is **much stronger senior-level reasoning** than trying to fit every pattern into the solution.

---

## Final mental model for Problem 2

```text id="k2j8xq"
                  FOOD DELIVERY SYSTEM
                          │
          ┌───────────────┼────────────────┐
          │               │                │
       DOMAIN          WORKFLOW         EVENTS
          │               │                │
          ▼               ▼                ▼
 Customer             OrderService    EventPublisher
 Restaurant           PaymentService       │
 Menu                 FareCalculator        │
 MenuItem             DeliveryService       │
 Cart                                      ▼
 Order                                 Observers
 OrderItem                                │
 DeliveryPartner                 ┌────────┼────────┐
                                  ▼        ▼        ▼
                                Email     SMS      Push

                     Order
                       │
                       ▼
                  OrderState
                       │
        ┌──────────────┼───────────────┐
        ▼              ▼               ▼
      Placed        Preparing        Ready
        │                              │
        ▼                              ▼
    Cancelled                    OutForDelivery
                                       │
                                       ▼
                                   Delivered
```

### The key interview takeaway

For this problem, your **core design reasoning** should be:

> **Entities model the domain. Services orchestrate use cases. State controls the order lifecycle. Strategy/polymorphism handles payment variations. Observer handles independent reactions to order events. DI/composition keeps implementations replaceable. Factory is optional and only justified by runtime object selection/creation.**

That's the level of abstraction I'd want you to be able to explain verbally **before writing the code**.

