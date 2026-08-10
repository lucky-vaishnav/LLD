Great. Let's start with the foundation.

## 🆕LLD Topic: OOP Fundamentals

Before we get into **SOLID, design patterns, or LLD problems**, we need to make sure OOP concepts are solid because interviewers often evaluate LLD through how well you apply OOP—not just whether you know the definitions.

### What we'll cover

1. **Class & Object**
2. **Encapsulation**
3. **Abstraction**
4. **Inheritance**
5. **Polymorphism**
6. **Composition vs Inheritance**
7. **Association / Aggregation / Composition**
8. How these concepts translate into **real LLD designs**

### Goal

You shouldn't just be able to say:

> "Encapsulation means wrapping data and methods together."

You should be able to explain:

> **Why** we need encapsulation, **where** you'd use it in a real system, and **what problem it solves**.

For example, instead of:

```text
order.status = "SHIPPED"
```

a better design might be:

```text
order.ship()
```

because the `Order` object controls **how** its state can transition.

That's the level of understanding we'll target.

---

### Our learning flow

For each concept:

**Concept → Why → Real-world example → TypeScript implementation → LLD application → Interview questions**


Next, we'll start with **Class vs Object — from an LLD/interview perspective**.

