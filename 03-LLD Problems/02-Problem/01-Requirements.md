## Problem 2 — Food Delivery Order System

Design a **Food Delivery Order System** similar to Swiggy/Zomato.

### Requirements

A customer should be able to:

1. Browse restaurants.
2. View a restaurant's menu.
3. Add/remove items from a cart.
4. Place an order.
5. Select a payment method.
6. After placing the order:

   * Restaurant should receive the order.
   * Customer should receive an order confirmation.
7. Restaurant can:

   * Accept the order.
   * Reject the order.
   * Mark it as preparing.
   * Mark it as ready.
8. A delivery partner can:

   * Accept the delivery.
   * Pick up the order.
   * Mark it delivered.
9. Customer can cancel the order **only when cancellation is allowed**.
10. Payment can be through multiple methods, such as Card, UPI, or Wallet.
11. Notification can support multiple channels such as Email, SMS, and Push.
12. The system should be designed so that adding a new payment method, notification channel, or future order status doesn't require major changes to existing code.

### Your task

Don't write code yet.

Give me your **first-pass LLD design**, similar to Problem 1:

* What are the main entities/classes?
* What are their responsibilities?
* Which things should be interfaces?
* Which things should be classes?
* Which things should be services?
* How will the main order lifecycle work?
* Where would you use composition/DI?
* Which patterns do you think are appropriate?
* Most importantly, **why** would you use each pattern?

### One important rule for Problem 2

Don't try to use all 14 patterns.

I specifically want to see whether you can now think:

> **Requirement → responsibility → design → pattern (only if needed)**

rather than:

> **Requirement → which pattern can I apply?**

Take your time and give me your first-pass design. I'll review it like a senior-level LLD interview and then start the follow-up questions.

### My Thoughts-


