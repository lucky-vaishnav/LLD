Let's move to the **next topic: Template Method Pattern**.

This is a good next step because it connects directly with concepts you've already learned: **inheritance, polymorphism, abstraction, and the Open/Closed Principle**. It will also give you a useful comparison with **Strategy**, which you've already understood well.

# Topic 9 — Template Method Pattern

## 1. The problem

Suppose we have different payment flows:

```text
Credit Card Payment
UPI Payment
Wallet Payment
```

The overall payment process is mostly the same:

```text
1. Validate payment
2. Authenticate
3. Process payment
4. Record transaction
5. Send confirmation
```

But some individual steps differ.

For example:

```text
Credit Card:
    authenticate → card authentication

UPI:
    authenticate → UPI PIN/UPI authentication
```

We don't want to duplicate the entire workflow in every class.

---

## 2. Template Method idea

We put the **overall algorithm/workflow in a base class**:

```text
PaymentProcessor
       |
       |── processPayment()
       |
       ├── validate()
       ├── authenticate()
       ├── process()
       ├── record()
       └── notify()
```

The parent controls the overall sequence.

The subclasses provide the steps that vary.

---

## 3. Example

```ts
abstract class PaymentProcessor {

  processPayment(): void {
    this.validate();
    this.authenticate();
    this.process();
    this.record();
    this.notify();
  }

  validate(): void {
    console.log("Common validation");
  }

  abstract authenticate(): void;

  abstract process(): void;

  record(): void {
    console.log("Recording transaction");
  }

  notify(): void {
    console.log("Sending confirmation");
  }
}
```

Credit card:

```ts
class CardPaymentProcessor extends PaymentProcessor {

  authenticate(): void {
    console.log("Authenticate card");
  }

  process(): void {
    console.log("Process card payment");
  }
}
```

UPI:

```ts
class UpiPaymentProcessor extends PaymentProcessor {

  authenticate(): void {
    console.log("Authenticate UPI");
  }

  process(): void {
    console.log("Process UPI payment");
  }
}
```

Now:

```ts
const payment = new CardPaymentProcessor();

payment.processPayment();
```

The sequence is always:

```text
validate
   ↓
authenticate
   ↓
process
   ↓
record
   ↓
notify
```

The subclass cannot accidentally change the overall workflow unless the base class allows it.

---

# 4. Why is it called Template Method?

Because:

```text
processPayment()
```

is the **template** for the algorithm.

It defines:

> **"These steps must happen in this order."**

Some steps are common:

```text
validate()
record()
notify()
```

Some steps are customizable:

```text
authenticate()
process()
```

---

# 5. The important concept: Hollywood Principle

You may hear this term in interviews:

> **"Don't call us, we'll call you."**

The parent/base class controls the flow.

The subclass doesn't decide:

```text
"First I'll process, then I'll validate."
```

Instead:

```text
Base class
    ↓
processPayment()
    ↓
calls subclass methods
```

So control remains with the base class.

---

# 6. Template Method vs Strategy

This is **very important for you** because you've already learned Strategy.

### Strategy

You have an algorithm/behavior that can be completely replaced.

```text
PaymentService
      ↓
PaymentStrategy
 ┌────┼────┐
 ↓    ↓    ↓
Card UPI Wallet
```

The service delegates the behavior to the selected strategy.

### Template Method

The **overall algorithm remains fixed**, but individual steps can vary.

```text
PaymentProcessor
      ↓
processPayment()
      ↓
validate
      ↓
authenticate ← varies
      ↓
process      ← varies
      ↓
record
      ↓
notify
```

### Easy memory:

> **Strategy = change the whole algorithm/behavior.**

> **Template Method = keep the algorithm structure, customize specific steps.**

---

# 7. Template Method vs Factory

Don't confuse this with Factory either.

### Factory

Concerned with:

> **Object creation**

```text
createPaymentProcessor()
        ↓
CardPaymentProcessor
```

### Template Method

Concerned with:

> **Algorithm/workflow structure**

```text
processPayment()
    ↓
step 1
step 2
step 3
```

---

# 8. Where inheritance comes in

Template Method commonly uses:

```text
Base class
    ↑
 ┌──┴───────┐
 ↓          ↓
Card       UPI
```

This is different from the composition-heavy patterns we've recently discussed.

That's why Template Method is a good pattern to learn now.

But remember:

> **Don't use inheritance just because Template Method can use inheritance.**

If the workflow doesn't have a stable common structure, Strategy/composition may be better.

---

# 9. Real backend example

Imagine your application processes different types of refunds:

```text
Uber Refund
Parking Refund
Payment Gateway Refund
```

Common flow:

```text
validate refund
     ↓
check eligibility
     ↓
calculate refund
     ↓
execute refund
     ↓
record refund
     ↓
send notification
```

But:

```text
calculate refund
execute refund
```

may differ by refund type.

Template Method can model this nicely.

---

# 🧠 Your turn

Let's use this refund example.

### Questions

**Q1.** What would be the Template Method in the refund flow?

**Q2.** Which steps should probably be common in the base class?

**Q3.** Which steps might be abstract/customizable?

**Q4.** Why is Template Method different from Strategy?

**Q5.** What could go wrong if we use Template Method when the workflows don't actually have a stable common sequence?

Answer in your own words, and I'll review it like we have been doing.

---
### My Thoughts-
Template method in the refund flow will be calculate refund and execute refund. And the common in the base class will be like validation of the refund, check eligibility, calculate refund, and this type of functions. And which step might be abstract will be the same: calculate refund or the execute refund. And template method is different from the strategy because strategy is mostly specific to one behavior, which can be implemented in different way, but where template comes with a defined functional flow, and where some are common functionalities, like functions flow, and where some can vary, where we can use abstract class because abstract class do not allow the initialization, so it will work as a template, and by using polymorphism, we can define some abstract function, and subclass can implement them according to that. So yeah, if there is no stable common sequence, then using template method is not good because then the subclass will not follow the template. So it will be just something like a class, abstract class, which is just having some functions, but most of the subclass will update the definition of those functions of the abstract class or abstract method, or the methods which are not abstract, they will also be updated by the subclass. Then there is no point to use the template because there is no fixed template to follow. So the function can be rewritten in the subclass, which is breaking the template pattern requirement.

---
Yes — **your understanding is good**, especially your answer to Q5. You correctly identified that Template Method only makes sense when there is a **stable overall workflow**.

There are just a couple of important corrections.

## Q1. Template Method — ⚠️ Small correction

You said:

> "Template method will be calculate refund and execute refund."

Not quite.

The **Template Method is the entire workflow method**, for example:

```ts
processRefund()
```

Inside it:

```text
processRefund()
   ↓
validate()
   ↓
checkEligibility()
   ↓
calculateRefund()
   ↓
executeRefund()
   ↓
recordRefund()
   ↓
notifyCustomer()
```

So:

> **`processRefund()` is the Template Method.**

`calculateRefund()` and `executeRefund()` are individual steps/hooks within that template.

---

## Q2. Common steps — ✅

You correctly identified things like:

```text
validate()
checkEligibility()
recordRefund()
notifyCustomer()
```

These can be implemented in the base class when their behavior is common.

For example:

```ts id="4lpf8v"
abstract class RefundProcessor {

  processRefund(): void {
    this.validate();
    this.checkEligibility();
    this.calculateRefund();
    this.executeRefund();
    this.recordRefund();
  }

  validate() {
    // common
  }

  checkEligibility() {
    // common
  }

  abstract calculateRefund(): void;

  abstract executeRefund(): void;

  recordRefund() {
    // common
  }
}
```

---

## Q3. Variable steps — ✅

Exactly:

```text id="qf92de"
calculateRefund()
executeRefund()
```

could vary between:

```text
UberRefundProcessor
ParkingRefundProcessor
GatewayRefundProcessor
```

---

# Q4. Template Method vs Strategy — ✅

Your answer is correct.

I'd make your interview version slightly sharper:

### Strategy

> **Encapsulates interchangeable ways of performing a behavior.**

```text id="q2b5qk"
PaymentService
      ↓
PaymentStrategy
 ┌────┼─────┐
 ↓    ↓     ↓
Card UPI Wallet
```

### Template Method

> **Defines the skeleton of an algorithm and allows specific steps to vary.**

```text id="u7y3w9"
processRefund()
    ↓
validate       ← common
    ↓
eligibility   ← common
    ↓
calculate     ← varies
    ↓
execute       ← varies
    ↓
record         ← common
```

### 🧠 Easy memory

> **Strategy → replace the behavior.**

> **Template Method → preserve the workflow, customize the steps.**

---

# Q5. Your answer was particularly good — ✅

You said:

> "If there is no stable common sequence, then using Template Method is not good."

**Exactly.**

This is one of the most important practical considerations.

Suppose:

```text id="x0b7ak"
Uber Refund:
Validate → Calculate → Execute → Record

Parking Refund:
Validate → Eligibility → Calculate → Execute → Record

SomeOther Refund:
Validate → Execute → Calculate → Record
```

Now the workflow itself differs.

Trying to force them into:

```text id="n6q7ds"
validate
→ eligibility
→ calculate
→ execute
→ record
```

creates awkward overrides and conditions.

That's a sign that Template Method may be the wrong abstraction.

---

## One correction to your reasoning

You said:

> "subclass will not follow the template"

Actually, **the whole point of Template Method is that the subclass generally cannot/shouldn't change the template method itself.**

For example:

```ts id="6i5u6h"
abstract class RefundProcessor {

  processRefund() {
    this.validate();
    this.calculateRefund();
    this.executeRefund();
    this.recordRefund();
  }

  abstract calculateRefund(): void;
  abstract executeRefund(): void;
}
```

The subclass changes:

```text id="v1h6b5"
calculateRefund()
executeRefund()
```

but shouldn't redefine:

```text id="2g6m8r"
processRefund()
```

So the sequence remains controlled by the base class.

That's the **core value** of Template Method.

---

# ⚠️ One more important correction

You said:

> "abstract class do not allow initialization"

Be careful with that statement.

An abstract class **can have a constructor and can initialize common state**.

For example:

```ts id="h0k3qu"
abstract class RefundProcessor {

  constructor(
    protected paymentId: string
  ) {}

  abstract calculateRefund(): number;
}
```

What an abstract class generally prevents is **direct instantiation of the abstract class itself**:

```ts id="xk1gk6"
// ❌ Cannot directly instantiate
new RefundProcessor("123");
```

But subclasses can call its constructor.

So your notes should say:

> **"An abstract class cannot be directly instantiated, but it can have constructors, state, concrete methods, and abstract methods."**

---

# 🧠 Final notes for your LLD notebook

### Template Method Pattern

> **Defines the skeleton of an algorithm in a base class while allowing subclasses to customize specific steps without changing the overall workflow.**

```text id="e3gq3f"
Template Method
      ↓
 ┌────┼──────────────┐
 ↓    ↓              ↓
Common Common      Variable
step   step          step
                    ↑
                 subclass
```

### Use when:

* Overall workflow is stable
* Steps occur in a defined sequence
* Some steps are common
* Some steps vary between implementations

### Avoid when:

* Different implementations have substantially different workflows
* The sequence itself varies
* You're forcing unrelated classes into the same inheritance hierarchy

### Most important distinction

```text id="t3w7r0"
Strategy
→ "Which way should I perform this behavior?"

Template Method
→ "The workflow is fixed; which steps should vary?"
```

And one senior-level point worth remembering:

> **Template Method is usually inheritance-based, while Strategy is composition-based.**

That's a useful distinction when deciding between the two.


### Q1

> **If the overall workflow is fixed, but a subclass overrides/replaces most or even all of the methods used by the Template Method, does Template Method still make sense? Or does that indicate that Template Method is the wrong design?**

## Short answer: Usually, no. ✅

If the subclass is replacing **most or all of the steps**, then you're losing the main benefit of Template Method.

The value of Template Method is:

```text
Base class owns the workflow
        ↓
Common steps are reused
        ↓
Only selected steps vary
```

For example:

```ts
processRefund() {
  validate();          // common
  authenticate();      // common
  calculateRefund();   // varies
  executeRefund();     // varies
  recordRefund();      // common
}
```

This is a good Template Method because the base class provides significant reusable behavior **and controls the sequence**.

---

### But if subclass does this:

```ts
class UberRefund extends RefundProcessor {

  validate() {
    // completely different
  }

  authenticate() {
    // completely different
  }

  calculateRefund() {
    // different
  }

  executeRefund() {
    // different
  }

  recordRefund() {
    // different
  }
}
```

Then ask yourself:

> **What am I actually reusing from the base class?**

If the answer is **almost nothing**, Template Method is probably unnecessary.

You are essentially using inheritance just to share a method structure.

---

## The key distinction

It's **not**:

> "Can subclasses override the methods?"

They can.

The important question is:

> **"How much of the algorithm is genuinely shared and controlled by the base class?"**

### Good Template Method

```text
Base
 ├── validate       ← shared
 ├── authenticate   ← shared
 ├── calculate      ← variable
 ├── execute        ← variable
 └── record         ← shared
```

### Bad candidate

```text
Base
 ├── validate       ← overridden
 ├── authenticate   ← overridden
 ├── calculate      ← overridden
 ├── execute        ← overridden
 └── record         ← overridden
```

At that point, I'd reconsider the abstraction—**Strategy/composition or separate services** may be a better fit.

### ⭐ Interview-ready answer

> **"Template Method is useful when the base class can genuinely own and enforce a stable workflow while subclasses customize only some steps. If subclasses override most of the steps, the base class provides little reusable behavior, so Template Method is probably over-engineering and another design such as Strategy or composition may be more appropriate."**

That's the exact thought process I'd want you to use in an LLD interview.
