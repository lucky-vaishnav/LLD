# LLD Design Patterns — Interview Cheat Sheet

Idea is not to create another set of detailed notes. We already have those. This sheet should answer, in **10–15 minutes**, the questions most likely to ask:

1. **What is it?**
2. **When would you use it?**
3. **What problem does it solve?**
4. **What is it commonly confused with?**
5. **What is the one important interview caveat?**


| Pattern                         | Interview Definition                                                                                       | Use When / Mental Trigger                                                | Commonly Confused With             | Key Interview Point                                                                                                        |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **1. Factory Method**           | Encapsulates object creation so the caller doesn't directly depend on concrete creation logic.             | **"Which object should be created?"**                                    | Abstract Factory, Strategy         | It doesn't magically remove all knowledge of the type/input; its main value is **centralizing/decoupling creation logic**. |
| **2. Strategy**                 | Encapsulates interchangeable algorithms/behaviors behind a common interface.                               | **"Same job, different ways to do it."**                                 | State                              | Usually uses **polymorphism + composition/DI**. Choose one strategy at a time.                                             |
| **3. Observer**                 | Establishes a one-to-many relationship where subscribers are notified when a subject changes/events occur. | **"One event → multiple independent reactions."**                        | EventEmitter, Pub/Sub              | Great for **decoupling event producer from consumers**.                                                                    |
| **4. Decorator**                | Dynamically adds responsibilities/behavior to an object without modifying its original class.              | **"Add optional behavior around existing behavior."**                    | Proxy                              | Decorators **enhance**; they don't primarily control access. Can be stacked.                                               |
| **5. Adapter**                  | Converts one interface into another interface expected by the client.                                      | **"These two interfaces don't match."**                                  | Facade                             | Particularly useful with **legacy code/external SDKs/vendors**.                                                            |
| **6. Facade**                   | Provides a simple interface over a complex subsystem.                                                      | **"Hide a complicated workflow behind one simple API."**                 | Adapter                            | Facade **simplifies access**; it doesn't make incompatible interfaces compatible.                                          |
| **7. Proxy**                    | Provides a substitute/control layer in front of another object to control access or execution.             | **"Control access before reaching the real object."**                    | Decorator                          | Common for authorization, caching, lazy loading, access control, rate limiting.                                            |
| **8. Command**                  | Encapsulates a request/action as an object.                                                                | **"I need to represent, store, queue, retry, delay or undo an action."** | Strategy                           | The key value is **making an operation a first-class object**, not simply adding an extra method/class.                    |
| **9. Template Method**          | Defines the skeleton of an algorithm while allowing subclasses to customize selected steps.                | **"Workflow is fixed, some steps vary."**                                | Strategy                           | Requires a **stable sequence**. Usually uses an abstract/base class.                                                       |
| **10. Builder**                 | Separates complex object construction from its representation.                                             | **"Creating this object has many optional/complex construction steps."** | Factory                            | Factory answers **which object?** Builder answers **how to construct it?**                                                 |
| **11. State**                   | Allows an object to change behavior when its internal state changes.                                       | **"Behavior depends on lifecycle/state."**                               | Strategy                           | Especially useful when state-specific rules would otherwise become large `if/else`/`switch` logic.                         |
| **12. Chain of Responsibility** | Passes a request through a sequence of handlers where each handler can process or pass it onward.          | **"Multiple handlers/validations execute in sequence."**                 | Decorator, Middleware              | A handler can **stop the chain** or pass to the next handler.                                                              |
| **13. Composite**               | Lets individual objects and groups of objects be treated uniformly through the same interface.             | **"Treat one and many the same way."**                                   | Tree structures generally          | Leaf + Composite share a common abstraction. **Parent-child alone isn't enough.**                                          |
| **14. Singleton**               | Ensures only one instance of a class exists within a given process/runtime.                                | **"Exactly one instance is required."**                                  | Static class, DI-managed singleton | Don't use it merely because something is shared; **DI is often preferable**.                                               |

---

# 🔥 The 14 Mental Triggers

If you have **30 seconds before an interview**, remember this:

```text
Factory
→ WHICH object?

Strategy
→ WHICH behavior?

Observer
→ ONE event → MANY reactions

Decorator
→ ADD behavior

Adapter
→ MAKE interfaces compatible

Facade
→ SIMPLIFY complex subsystem

Proxy
→ CONTROL access

Command
→ REPRESENT an action/request as an object

Template Method
→ FIXED workflow + variable steps

Builder
→ COMPLEX object construction

State
→ Behavior changes with STATE

Chain of Responsibility
→ REQUEST passes through HANDLERS

Composite
→ ONE and MANY treated uniformly

Singleton
→ EXACTLY ONE instance per process
```

That alone is an excellent last-minute revision tool.

---

# ⭐ The Most Important Comparisons

These are the comparisons I'd **definitely memorize**, because interviewers love them.

### Strategy vs State

```text
Strategy
→ Choose a behavior.

State
→ Behavior changes because object's state changes.
```

**Shortcut:**
Strategy = **choice**
State = **lifecycle**

---

### Decorator vs Proxy

```text
Decorator
→ Add/enhance behavior.

Proxy
→ Control access to behavior.
```

**Shortcut:**
Decorator = **enhance**
Proxy = **control**

---

### Adapter vs Facade

```text
Adapter
→ Make incompatible interfaces compatible.

Facade
→ Make a complex subsystem easier to use.
```

**Shortcut:**
Adapter = **compatibility**
Facade = **simplicity**

---

### Factory vs Builder

```text
Factory
→ Which object should I create?

Builder
→ How should I construct this complex object?
```

**Shortcut:**
Factory = **which**
Builder = **how**

---

### Strategy vs Template Method

```text
Strategy
→ Entire behavior/algorithm can be swapped.

Template Method
→ Overall algorithm remains fixed,
  selected steps can vary.
```

**Shortcut:**
Strategy = **replace behavior**
Template = **customize steps**

---

### Observer vs Chain of Responsibility

```text
Observer
→ One event notifies multiple subscribers.

Chain
→ Request moves through handlers sequentially.
```

**Shortcut:**
Observer = **broadcast**
Chain = **pipeline**

---

### Composite vs Strategy

```text
Composite
→ Structure / one-and-many relationship.

Strategy
→ Behavior / algorithm selection.
```

---

### Singleton vs DI

```text
Singleton
→ Class controls its single instance.

DI
→ Container/external mechanism provides dependency.
```

And in NestJS:

> **Prefer DI/container-managed lifecycle instead of manually implementing Singleton in most cases.**

---

# 🧠 One More Senior-Level Rule

This is probably the **most valuable line in the entire cheat sheet**:

> **Don't use a design pattern because the pattern exists. Use it when the problem it solves actually exists.**

For example:

```text
Multiple payment algorithms?
→ Strategy

External vendor interface doesn't match ours?
→ Adapter

Complex payment workflow needs simple entry point?
→ Facade

Optional logging/retry behavior?
→ Decorator

Fixed workflow with customizable steps?
→ Template Method

Complex object construction?
→ Builder
```

And sometimes the correct interview answer is:

> **"I wouldn't introduce a pattern here; simple composition/classes are sufficient."**

That's actually a **stronger senior-level answer** than forcing a pattern into the design.

---
That will also tell us whether any of these 14 patterns need to be revisited before your interview.

