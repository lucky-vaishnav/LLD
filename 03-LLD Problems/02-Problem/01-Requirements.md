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
Okay, so for this problem two for our low-level design. So the entities and responsibilities, so I have thought about that, and according to me, the entities will be, or the classes will be, restaurant, second menu, user or client, cart, and these will be the main entities. And one more will be the delivery partner, and these classes will have the required variables. For example, delivery partner will have name, photo, phone number, restaurant will have name, rating, phone number, photos of restaurant, menu item will be a class, and menu items will be used as an object in this class menu. So these will be the major classes. And responsibility-wise, the responsibility will be like main responsibility will be like process a order, placing a order, and second responsibility will be major one will be the payment of the order. Third will be fare calculator, or maybe fare calculation of the order. And fourth will be sending the required notification. So these are the major responsibilities. And there will be one more class, which will be cart as well. Cart will also be a class. And the process or the lifecycle of the order will be like this: that first user will be see the restaurant and the menu, and he will put the menu item in the cart, and after he placed the order, so the fare will be calculated, calculating, and then will be order lifecycle will be start, where that order first first will be placed and then accepted, and then can be rejected by the restaurant or mark is prepared, and as mark is ready. And then there will be start a life cycle of the delivery partner. So the life cycle of delivery partner will be accepted and picked up the order and delivered the order. So this will be the flow, major flow. And for services, so we can create three services at least, I think. First will be the payment service, which will be responsible for the payment, and payment type will be used as a composition in this payment service. Second will be the notification service, where the notification type, which type of notification flow will be triggered. So this will be accepted as a dependency injection, and we will use here is composition. And third one will be the order service, which will be responsible for the life cycle of the order, like I shared, same like this: accepted, reject, accept order, reject order, prepared order, mark is prepared, and ready, and picked up, delivered. So this type of thing will be there. And so classes will be the same which I explained. There will be cart class, cart, restaurant, menu, and there will be order class as well: order, menu, cart. And there will be a fare calculator class, and cart class as well. Fare calculator will be calculating the fare for the menu item which are added, and cart will be doing the menu item to add into the cart or delete from the cart. So this type of operation will be supported. So the interface will be for the payment type and the notification type. We will use the interface, so that this interface will be implemented by the different type of notification or different type of payment providers. And for the payment service and the notification service to share the object as a dependency injection in these services, we can use the factory method as well to create a proper object of notification type or the payment type. So we will use factories for that, factory pattern. And we will also use the observer pattern, so that whenever we need to mostly we can use the observer pattern for triggering the notification as well, and if we are supporting the payment, auto payment as well, for that as well we can use the observer pattern. And so as the lifecycle of the order will be changed, so we can use the observer pattern to call the respected code part where it should be executed. Some code should be executed. So we can use that observer pattern as well. And we can also use the state pattern as well to validate and to control the change of the status of the order. And that's why we can use the state pattern, where will be the order status, state will be closed. And yeah. So these are the patterns I can think of right now, because for REST patterns, I do not think we should put those patterns here, because we know those patterns. And I am not sure why we will put those patterns. There is one more pattern in my mind which we can use, this is the facade pattern. And if we want to put a facade between these operations, we can use that as well. But these are the patterns which I thought will be mostly we should use. And yeah, you can give you my thought process. And there is one more point that whenever from the last problem as well, and this second problem as well, I somehow, my mind is work like that only this pattern, I feel like, which really we use whenever we solve a problem, like this factory method, this observer method, state, and these are the patterns we mostly, I think most of the problems will have according to me. So it will make writing, writing, writing this problem LLD will be very easy. So you can guide me on that as well. So each time my mind works like that also, that these are the patterns first comes in my mind whenever I read the requirement and think about the responsibility. So these are the patterns like this composition, dependency injection, factory pattern, and observer pattern, and state pattern are the common patterns which are always fit into the problems, or maybe it is okay, because so generally it is always helping for us to write the LLD solution for problems. Or maybe you can tell me that how else I can think, so that I can think of some other pattern as well, just in summarized way, not a deep explanation, because we have already learned the pattern and I have notes, but just the thought process.

