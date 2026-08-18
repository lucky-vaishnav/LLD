# 🧠 LLD Fundamentals — Application #4

## Design a Food Ordering System

This one is intentionally broader. I won't tell you which SOLID principle or OOP concept to use.

### Requirements

Design a simple food-ordering system.

A customer can:

1. Browse restaurants.
2. Add food items to a cart.
3. Remove items from the cart.
4. Place an order.
5. Pay for the order.
6. Cancel the order.

### Business rules

* A cart can contain items from **only one restaurant**.
* An order cannot be placed if the cart is empty.
* Once an order is placed, the cart should no longer be modified for that order.
* Payment must succeed before the order becomes confirmed.
* An order can be cancelled only when it is in a cancellable state.
* Different payment methods may exist.
* Different restaurants may have different menu items.
* We may add more payment methods and order states in the future.

---

## 🎤 Your interview challenge

This time, **don't try to answer everything at once**.

Start by telling me how you would model the main objects/classes.

### Q1 — Identify the objects

What classes/entities would you create?

For example, would you have:

```text
Customer
Restaurant
Menu
MenuItem
Cart
Order
Payment
```

or something different?

Explain **why each important object exists**.

---

### Q2 — Ownership

Think carefully about:

```text
Customer
   ↓
  Cart
   ↓
CartItem
```

and:

```text
Restaurant
   ↓
  Menu
   ↓
MenuItem
```

Who should own what?

---

### Q3 — Encapsulation & invariants

Where should this rule live?

> "A cart cannot contain items from multiple restaurants."

Would you put that logic in:

```text
Cart
CartService
Restaurant
Controller
```

and why?

---

### Q4 — Order state

An order can move through states such as:

```text
CREATED
CONFIRMED
PREPARING
READY
DELIVERED
CANCELLED
```

Who should control these state transitions?

And what would you do to prevent something like:

```text
DELIVERED → CREATED
```

from accidentally happening?

You can mention any **new pattern/concept that comes to mind**, even if we haven't learned it yet.

---

### Q5 — Payment

Where does payment processing belong?

Would you make:

```text
Order
   ↓
Payment
```

or:

```text
OrderService
   ↓
PaymentService
```

or something else?

And how would you keep different payment implementations loosely coupled?

---

### Q6 — Future change

Suppose tomorrow we add:

```text
UPI
Credit Card
Wallet
Cash on Delivery
```

and later:

```text
Order state rules become more complex
```

What parts of your design should be open for extension without constantly modifying existing business logic?

---

### Q7 — One final senior-level question

Imagine another developer puts this in the controller:

```text
if (order.status === "DELIVERED") {
    // ...
}
```

and another developer puts:

```text
if (order.status === "CANCELLED") {
    // ...
}
```

and soon there are `if/else` checks for every state throughout the codebase.

**Would you consider that a design problem? Why?**

---
### My Thought-
So as there are different functionality, different responsibility, so we can properly utilize the SOLID principle here. So as first we can identify the object like customer, restaurant, menu, menu item, cart, order, and payment as you said. So for those, we should create an abstraction for it. Like for customer, they will be the customer name, address, phone number. For restaurant, it will be restaurant name, restaurant location, restaurant menu item. And that menu item will be a different abstraction where the menu item information will be there. So combining these both objects, we can show the menus of the restaurant. And there will be a cart object as well. The cart will have actions like add menu in the cart, remove menu in the cart. And cart will also have that place the order. And for the order as well, we have the single abstraction and the class for the single responsibility where the lifecycle of the order will be maintained. And same for the payment. For payment, in the future there can be multiple payment methods supported. So we can use here the abstraction and the payment service as well. So that from the order, we can call the required payment methods for the payment. So I think by using that, we can achieve the single responsibility and open and close principle, and dependency injection via the order and the payment service. And we can put the limitation and the validation on the specific services. Let's say for the order, we can put the validation there. And that order will always be connected with the restaurant. And this thing we can do. And for the responsibility wise, the customer will be responsible for the cart. And restaurant will be the owner of menu. And rest order and payment, so this should not be in an owner way. They can work as a dependency injection. And yeah, for the lifecycle, the customer will, first we will show the restaurants and menu within those restaurants. Then customer will put those menu in the cart. And from the cart, the order will be placed. And on placing the order, we will call the proper payment flow using the dependency injection of the payment, using the order object. Let's say that the order object have the payment method information. So we will use that. And this order object and this payments object should not be manipulated from the outside of those classes. And this should be managed within the class so we can use so that we should maintain the object state. And that is how we can use the encapsulation here. So in the top view, so it will be like that. And for services like order service and payment service, so it totally depends upon our thought process because we can use the services as well, and we can use just class as well. But for payment, at least we should use a service so that it can be used and can be extended using the new payment method objects when the new payment method will be added. And that is how we can achieve that.

---
You are now thinking in terms of **responsibilities, ownership, state, abstraction, and extensibility**, rather than just naming SOLID principles.

There are a few important corrections, though.

# 📝 Review — Food Ordering System

## 1. Main objects — Good

You identified:

```text
Customer
Restaurant
Menu
MenuItem
Cart
Order
Payment
```

That's a good domain model.

One refinement: I wouldn't automatically call every one of these an **abstraction/interface**.

For example:

```text
Customer
Restaurant
MenuItem
Cart
Order
```

can simply be **domain classes**.

Interfaces are useful when we have:

* multiple implementations
* replaceable behavior
* an important abstraction boundary

For example, payment is an obvious candidate:

```text
PaymentProcessor
      ↑
 ┌────┼─────┐
UPI  Card  Wallet
```

So remember:

> **Not every object needs an interface.**

---

# ✅ 2. Restaurant → Menu → MenuItem

Your ownership thinking is good.

A reasonable model is:

```text
Restaurant
    ↓ owns
  Menu
    ↓ contains
MenuItem
```

And:

```text
Restaurant
    ↓
   Menu
    ↓
 MenuItems
```

The important question is:

> Who owns the lifecycle of the object?

That's what we mean by ownership/composition at the domain level.

---

# ⚠️ 3. Cart → Order

You said:

> "Cart will also have ... place the order."

This is the one I'd change.

A `Cart` should primarily be responsible for:

```text
addItem()
removeItem()
getItems()
calculateTotal()
```

But **placing an order is a larger business operation**.

Conceptually:

```text
Cart
 ↓
contains items

Order
 ↓
created from cart
```

A service/application layer could orchestrate:

```text
OrderService
    ↓
validate Cart
    ↓
create Order
    ↓
process Payment
    ↓
confirm Order
```

So I'd avoid making `Cart` responsible for the entire ordering workflow.

This is **SRP applied at the domain level**.

---

# ⭐ 4. The restaurant constraint should live close to Cart

The requirement was:

> A cart can contain items from only one restaurant.

You said validation could go into a service.

That **can work**, but I'd prefer the `Cart` itself to protect this invariant.

For example conceptually:

```text
Cart.addItem(menuItem)

    ↓

Does cart already have restaurant?

    ↓ YES
Does menuItem belong to same restaurant?

    ↓ NO
Reject
```

Why?

Because `Cart` owns the collection of items.

So:

> **An object should protect the invariants of the state it owns.**

This is the same lesson we saw with `BankAccount.balance`.

---

# ✅ 5. Encapsulation — Good

You correctly said:

> "These objects should not be manipulated from outside."

Exactly.

For example:

```text
Cart.items
```

shouldn't be freely replaceable:

```ts
cart.items = [...]
```

Instead:

```text
cart.addItem(item)
cart.removeItem(item)
```

The cart controls its own state.

Likewise:

```text
Order.status
```

shouldn't be:

```ts
order.status = "DELIVERED";
```

from anywhere in the application.

The order should control valid state transitions.

---

# ⭐ 6. Order lifecycle — this is where you recognized a new pattern

You said:

> "For the lifecycle, the order will be maintained."

Good.

The requirement:

```text
CREATED
   ↓
CONFIRMED
   ↓
PREPARING
   ↓
READY
   ↓
DELIVERED
```

with cancellation rules is a classic example where a **State Pattern** may eventually become useful.

You don't need to know it yet.

But this is exactly the kind of pattern recognition I want you to develop.

You can remember:

> **Complex object behavior that changes based on its current state → State Pattern may be appropriate.**

We'll formally learn it later.

---

# ⚠️ 7. Payment + Dependency Injection

Your thinking here is good:

```text
OrderService
      ↓
PaymentService / PaymentProcessor
      ↓
abstraction
      ↑
UPI / Card / Wallet
```

But one small correction:

> **DI itself doesn't select the payment method.**

DI provides a dependency.

Something else has to determine:

```text
UPI → UpiProcessor
CARD → CardProcessor
```

We previously identified that this is where a Factory/Registry-type approach can help.

Again, you don't need to implement it yet.

---

# ⚠️ 8. "Payment method information inside Order"

You said:

> "The order object has the payment method information."

That's possible, but I'd distinguish:

```text
Payment method
```

from:

```text
Payment processing
```

An order may know:

```text
paymentMethod = UPI
```

but it shouldn't necessarily know:

```text
how UPI payment is processed
```

So:

```text
Order
 └── paymentMethod = UPI

PaymentService
 └── knows how to process UPI
```

This keeps domain objects independent of infrastructure details.

---

# ✅ 9. Services vs classes

You said:

> "It depends on our thought process whether we use services or classes."

That's a **good instinct**.

Don't create:

```text
CustomerService
RestaurantService
MenuService
CartService
OrderService
PaymentService
...
```

just because that's what backend projects often do.

Ask:

> Is this object representing domain state/behavior, or is this a workflow/orchestration operation?

For example:

```text
Cart
→ domain object

Order
→ domain object

PaymentProcessor
→ abstraction

OrderService
→ potentially application/workflow orchestration
```

That's a much better way to decide.

---

# 🧠 Your design, refined

A reasonable conceptual design would be:

```text
                    Customer
                       │
                       ↓
                      Cart
                       │
                       │ contains
                       ↓
                   MenuItem
                       ↑
                       │
                    Menu
                       ↑
                       │
                  Restaurant


Cart
 │
 │ checkout
 ↓
OrderService
 │
 ├── validate Cart
 │
 ├── create Order
 │
 └── PaymentService
          │
          ↓
    PaymentProcessor
          ↑
     ┌────┼─────┐
     │    │     │
    UPI  Card  Wallet
```

And:

```text
Order
 │
 └── controls its own state
       ↓
 CREATED
   ↓
 CONFIRMED
   ↓
 PREPARING
   ↓
 READY
   ↓
 DELIVERED
```

The important part is that external code shouldn't arbitrarily mutate these states.

---

# 📝 Notes — Food Ordering System

> **Domain objects should own their state and protect their invariants.**

### Cart

Responsible for:

```text
addItem()
removeItem()
calculateTotal()
```

and protecting:

```text
Only one restaurant per cart
```

### Order

Responsible for:

```text
Order state
Valid state transitions
Cancellation rules
```

### OrderService

Can orchestrate:

```text
Cart validation
→ Order creation
→ Payment
→ Order confirmation
```

### Payment

Use an abstraction when multiple implementations exist:

```text
PaymentProcessor
      ↑
UPI / Card / Wallet
```

### Important design rules

* Not every class needs an interface.
* Not every operation needs a service.
* Domain objects should protect their own state.
* Business rules should generally live close to the state they protect.
* DI provides dependencies; it doesn't necessarily decide which implementation to use.
* Complex state-dependent behavior may indicate the **State Pattern**.
* Dynamic implementation selection may indicate a **Factory/Registry** approach.

---
```ts
interface PaymentProcessor {
  pay(amount: number): void;
}

class UpiPaymentProcessor implements PaymentProcessor {
  pay(amount: number): void {
    console.log(`Paid ₹${amount} using UPI`);
  }
}

class CardPaymentProcessor implements PaymentProcessor {
  pay(amount: number): void {
    console.log(`Paid ₹${amount} using Card`);
  }
}

class WalletPaymentProcessor implements PaymentProcessor {
  pay(amount: number): void {
    console.log(`Paid ₹${amount} using Wallet`);
  }
}

class Customer {
  constructor(
    public readonly id: string,
    public readonly name: string
  ) {}
}

class MenuItem {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly price: number,
    public readonly restaurantId: string
  ) {}
}

class Menu {
  private readonly items: MenuItem[] = [];

  constructor(public readonly restaurantId: string) {}

  addItem(item: MenuItem): void {
    if (item.restaurantId !== this.restaurantId) {
      throw new Error("Item belongs to a different restaurant");
    }

    this.items.push(item);
  }

  getItems(): MenuItem[] {
    return [...this.items];
  }
}

class Restaurant {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly menu: Menu
  ) {}
}

class Cart {
  private readonly items: MenuItem[] = [];
  private restaurantId: string | null = null;

  addItem(item: MenuItem): void {
    if (
      this.restaurantId !== null &&
      this.restaurantId !== item.restaurantId
    ) {
      throw new Error("Cart can contain items from only one restaurant");
    }

    this.restaurantId = item.restaurantId;
    this.items.push(item);
  }

  removeItem(itemId: string): void {
    const index = this.items.findIndex(item => item.id === itemId);

    if (index === -1) {
      throw new Error("Item not found in cart");
    }

    this.items.splice(index, 1);

    if (this.items.length === 0) {
      this.restaurantId = null;
    }
  }

  getItems(): MenuItem[] {
    return [...this.items];
  }

  getTotal(): number {
    return this.items.reduce((total, item) => total + item.price, 0);
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

type OrderStatus =
  | "CREATED"
  | "CONFIRMED"
  | "PREPARING"
  | "READY"
  | "DELIVERED"
  | "CANCELLED";

class Order {
  private status: OrderStatus = "CREATED";

  constructor(
    public readonly id: string,
    public readonly customer: Customer,
    public readonly restaurantId: string,
    private readonly items: MenuItem[],
    public readonly totalAmount: number
  ) {}

  getStatus(): OrderStatus {
    return this.status;
  }

  confirm(): void {
    if (this.status !== "CREATED") {
      throw new Error("Order cannot be confirmed in current state");
    }

    this.status = "CONFIRMED";
  }

  startPreparing(): void {
    if (this.status !== "CONFIRMED") {
      throw new Error("Order cannot start preparing");
    }

    this.status = "PREPARING";
  }

  markReady(): void {
    if (this.status !== "PREPARING") {
      throw new Error("Order cannot be marked ready");
    }

    this.status = "READY";
  }

  deliver(): void {
    if (this.status !== "READY") {
      throw new Error("Order cannot be delivered");
    }

    this.status = "DELIVERED";
  }

  cancel(): void {
    if (
      this.status !== "CREATED" &&
      this.status !== "CONFIRMED"
    ) {
      throw new Error("Order cannot be cancelled");
    }

    this.status = "CANCELLED";
  }

  getItems(): MenuItem[] {
    return [...this.items];
  }
}

class PaymentService {
  constructor(
    private readonly paymentProcessor: PaymentProcessor
  ) {}

  pay(amount: number): void {
    if (amount <= 0) {
      throw new Error("Payment amount must be positive");
    }

    this.paymentProcessor.pay(amount);
  }
}

class OrderService {
  constructor(
    private readonly paymentService: PaymentService
  ) {}

  placeOrder(
    customer: Customer,
    cart: Cart
  ): Order {
    if (cart.isEmpty()) {
      throw new Error("Cannot place an empty order");
    }

    const items = cart.getItems();

    const order = new Order(
      crypto.randomUUID(),
      customer,
      items[0].restaurantId,
      items,
      cart.getTotal()
    );

    this.paymentService.pay(order.totalAmount);

    order.confirm();

    return order;
  }
}


// Example usage

const customer = new Customer(
  "C001",
  "Lucky"
);

const menu = new Menu("R001");

const pizza = new MenuItem(
  "M001",
  "Pizza",
  300,
  "R001"
);

const burger = new MenuItem(
  "M002",
  "Burger",
  200,
  "R001"
);

menu.addItem(pizza);
menu.addItem(burger);

const restaurant = new Restaurant(
  "R001",
  "Food Palace",
  menu
);

const cart = new Cart();

cart.addItem(pizza);
cart.addItem(burger);

const paymentProcessor = new UpiPaymentProcessor();

const paymentService = new PaymentService(
  paymentProcessor
);

const orderService = new OrderService(
  paymentService
);

const order = orderService.placeOrder(
  customer,
  cart
);

console.log(order.getStatus());

order.startPreparing();

order.markReady();

order.deliver();

console.log(order.getStatus());
```

# 🎉 LLD FUNDAMENTALS — COMPLETED

I think you've now demonstrated enough.

We've covered and applied:

```text
OOP
├── Encapsulation
├── Abstraction
├── Inheritance
├── Polymorphism
└── Composition

SOLID
├── SRP
├── OCP
├── LSP
├── ISP
└── DIP

Supporting concepts
├── Dependency Injection
├── Coupling / Cohesion
├── Object ownership
├── State & invariants
├── Domain modeling
└── Interface vs concrete class decisions
```

And you've applied them across:

```text
1. Notification System
2. Payment System
3. Bank Account
4. Food Ordering System
```

### 📝 When to use Class vs Interface vs Service

| Use           | When                                                                                                                                                                                     |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Class**     | When you need to represent an **object/domain concept** that owns state and/or behavior. Use it when the behavior is concrete and doesn't need multiple interchangeable implementations. |
| **Interface** | When you need to define a **contract/abstraction** and expect multiple implementations, interchangeable behavior, or want to decouple high-level code from concrete implementations.     |
| **Service**   | When you have a **business workflow/orchestration** that coordinates multiple objects/components rather than owning the core state itself.                                               |

### 🧠 Simple mental model

```text
Class
→ "What is this object and what does it own/do?"

Interface
→ "What contract/capability do I need, regardless of implementation?"

Service
→ "What workflow needs to coordinate multiple things?"
```

### Examples

```text
BankAccount
→ Class
→ owns balance + deposit/withdraw rules

PaymentProcessor
→ Interface
→ Card / UPI / Wallet can implement it

OrderService
→ Service
→ coordinates Cart → Order → Payment
```

### One important correction

Don't think:

> **"No polymorphism → class, polymorphism → interface."**

That's a useful starting heuristic, but not a strict rule.

Also don't create a service just because something is called an "operation."

Ask:

> **Does this behavior naturally belong to an object that owns the relevant state?**

If yes → put it in the **domain class**.

If it's a larger workflow involving multiple objects → a **service** may be appropriate.

And one more senior-level rule:

> **Don't create an interface, service, or abstraction until there is a design reason for it.**

This will help you avoid over-engineering in LLD interviews.


## 🚀 Next phase: Design Patterns

From the next session, we'll start **Design Patterns**.

And we'll follow the approach you requested: whenever a pattern naturally appears in a problem before we've formally learned it, I'll point it out so you can build that mental connection. Then we'll formally learn the pattern, understand **why it exists, when to use it, when NOT to use it, implementation, trade-offs, and interview questions**.

The first pattern will be selected based on what gives you the strongest foundation for the patterns that follow.
