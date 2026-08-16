## 🆕 LLD Topic #11: Liskov Substitution Principle (LSP)

The **L** in SOLID.

This is one of the principles that is often memorized incorrectly, so we'll focus on the **actual design reasoning** behind it.

---

# 1. What is LSP?

The formal definition is:

> **Objects of a subclass should be replaceable with objects of its superclass without breaking the correctness of the program.**

That's a little abstract.

A simpler way to think about it:

> **If `B` is a subtype of `A`, wherever the system expects `A`, it should be safe to provide `B`.**

In other words:

```text
A is the expected contract
B claims to be an A
        ↓
B must actually behave like an A
```

---

# 2. The classic Bird example

Suppose we design:

```ts id="3z8c7x"
class Bird {
  fly() {
    console.log("Flying");
  }
}
```

Then:

```ts id="f7yqjq"
class Sparrow extends Bird {}

class Penguin extends Bird {}
```

Now:

```ts id="j1k6cq"
function makeBirdFly(bird: Bird) {
  bird.fly();
}
```

We can do:

```ts id="e8s7k2"
makeBirdFly(new Sparrow());
```

Fine.

But:

```ts id="1kz5sp"
makeBirdFly(new Penguin());
```

Our program expects every `Bird` to be able to fly.

But Penguin can't.

So the subclass doesn't properly satisfy the behavior expected from the parent.

**LSP is violated.**

---

# 3. The real problem isn't inheritance itself

This is important.

Someone might say:

> "Penguin shouldn't extend Bird."

But biologically, a penguin **is a bird**.

The actual problem is that we created an incorrect abstraction:

```text id="eqk0c2"
Bird
 └── fly()
```

We put `fly()` into a contract that not every bird can satisfy.

The better abstraction is:

```ts id="h8u5q3"
class Bird {
  eat() {}
}
```

Then:

```ts id="7d6p9n"
interface FlyingBird {
  fly(): void;
}
```

Now:

```ts id="2p0z1y"
class Sparrow extends Bird implements FlyingBird {
  fly() {}
}

class Penguin extends Bird {
  // doesn't need fly()
}
```

Now the model makes more sense.

---

# 4. LSP is about behavior, not just types

This is the most important point.

Suppose:

```ts id="g1v2x8"
class Rectangle {
  setWidth(width: number) {}
  setHeight(height: number) {}
}
```

Then:

```ts id="j4c7qa"
class Square extends Rectangle {
  setWidth(width: number) {
    // changes both width and height
  }

  setHeight(height: number) {
    // changes both width and height
  }
}
```

Mathematically:

```text
Square IS-A Rectangle
```

But from a software behavior perspective, the substitution can cause problems.

Imagine code expects:

```ts id="m1y7cg"
function resize(rectangle: Rectangle) {
  rectangle.setWidth(10);
  rectangle.setHeight(20);

  // expects width = 10
  // expects height = 20
}
```

With a square:

```ts id="x7o8p1"
resize(new Square());
```

you might end up with:

```text
width  = 20
height = 20
```

The caller's assumptions are broken.

That's an LSP violation.

---

# 5. The key idea: Contracts

Think of a parent class/interface as making a **contract** with its consumers.

For example:

```ts id="8q4y2c"
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
}
```

The consumer assumes:

```text id="o7y9ck"
PaymentProcessor
      ↓
pay()
      ↓
returns PaymentResult
```

If we create:

```ts id="r6c4a8"
class RazorpayProcessor implements PaymentProcessor {
  pay(amount: number) {
    throw new Error("I don't support payments");
  }
}
```

Technically TypeScript may allow the implementation.

But conceptually, this is a poor substitute.

The caller expects:

```text
pay()
→ processes payment
→ returns a valid result
```

Instead:

```text
pay()
→ throws "not supported"
```

The implementation doesn't honor the expected contract.

That's the essence of LSP.

---

# 6. A practical backend example

Imagine:

```ts id="8x6k2p"
interface PaymentProcessor {
  pay(amount: number): PaymentResult;
  refund(transactionId: string): RefundResult;
}
```

Now suppose we have:

```text id="7g4tqk"
StripeProcessor
CyberSourceProcessor
RazorpayProcessor
```

But Razorpay doesn't support refunds through this particular integration.

If we still force:

```ts id="a8z1cu"
class RazorpayProcessor implements PaymentProcessor {
  refund(transactionId: string) {
    throw new Error("Not supported");
  }
}
```

we may have a design problem.

The abstraction may be too broad.

This leads us directly toward another SOLID principle:

**Interface Segregation Principle (ISP).**

Instead of:

```ts id="q6v7o8"
interface PaymentProcessor {
  pay();
  refund();
}
```

we might have:

```ts id="k0n2x4"
interface PaymentProcessor {
  pay();
}

interface RefundProcessor {
  refund();
}
```

Then:

```text id="0z5v8h"
StripeProcessor
   ├── PaymentProcessor
   └── RefundProcessor

RazorpayProcessor
   └── PaymentProcessor
```

Now every implementation only promises what it can actually support.

This is a very important LLD connection.

---

# 7. LSP and inheritance

LSP doesn't only apply to traditional class inheritance.

It applies whenever one implementation is presented as a substitute for an abstraction.

For example:

```ts id="t8w0x5"
interface Notification {
  send(message: string): void;
}
```

If:

```ts id="z1n4k7"
class EmailNotification implements Notification
```

then Email must behave in a way that makes sense wherever `Notification` is expected.

Same for:

```text id="k2c8n9"
SMS
Push
WhatsApp
```

They can have different implementations, but they must honor the **common contract**.

---

# 8. LSP vs OCP

These two are closely related.

### OCP

> Can I add a new implementation without modifying existing code?

### LSP

> Can the new implementation actually substitute for the existing abstraction without breaking assumptions?

For example:

```text id="9xj6e3"
PaymentProcessor
       ↑
       │
RazorpayProcessor
```

OCP says:

> Great, we can add Razorpay without changing PaymentService.

LSP asks:

> Does Razorpay actually behave according to the PaymentProcessor contract?

You need **both** for a good polymorphic design.

---

# 9. A useful LSP test

When creating a subclass/implementation, ask:

> **"If I replace the original implementation with this new one, will the existing client code still work correctly without special handling?"**

If you need:

```ts id="y6c1x0"
if (processor instanceof RazorpayProcessor) {
  // special handling
}
```

that's often a warning sign.

The abstraction may not be designed correctly.

Similarly, if you find:

```ts id="8z3h5q"
if (bird instanceof Penguin) {
   // don't call fly
}
```

your `Bird` abstraction is probably wrong.

---

# 10. LSP is NOT "subclasses must behave identically"

Another important nuance.

Different implementations **can absolutely behave differently**.

For example:

```text id="2z8m4q"
StripeProcessor.pay()
    → Stripe API

CyberSourceProcessor.pay()
    → CyberSource API
```

Their internal behavior is different.

That's expected.

What must remain consistent is the **contract/expectations of the abstraction**.

So:

```text id="q8t7v3"
Same contract
      +
Different valid implementations
      =
Polymorphism
```

---

# 11. Senior-level interview answer

If asked:

> **"Explain Liskov Substitution Principle."**

A strong answer:

> "LSP says that a subtype should be safely substitutable wherever its parent abstraction is expected, without breaking the client's assumptions or correctness. The important part is behavioral compatibility, not just having the same method signatures. If a subclass needs special handling or cannot honor the contract of the parent, that usually indicates the abstraction or hierarchy needs to be redesigned."

That's a strong senior-level answer.

---

# 🎯 Today's key takeaway

Remember this:

```text id="r0d4p8"
LSP
 ↓
Subtype must honor the parent's contract
 ↓
Client should not need to know
which concrete implementation it received
```

The most useful question:

> **"Can I replace A with B without breaking the code that expects A?"**

If yes → likely good substitution.

If no → **LSP problem / bad abstraction**.

---

### 📚 Progress

We've now covered:

1. Class & Object
2. Encapsulation
3. Abstraction
4. Inheritance vs Composition
5. Polymorphism
6. Association / Aggregation / Composition
7. Dependency Injection
8. Coupling & Cohesion
9. SRP
10. OCP
11. **LSP ← Today**

Next SOLID principle:

**ISP — Interface Segregation Principle**
