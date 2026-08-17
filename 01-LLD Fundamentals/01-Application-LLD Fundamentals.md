Today we'll start the **application phase** rather than introducing another isolated principle.

# 🧠 LLD Fundamentals — Application #1

## Design a Notification System

This is deliberately based on examples we've already discussed, but now **you have to make the design decisions**.

The goal is to see whether you can naturally apply:

* Encapsulation
* Abstraction
* Composition
* Polymorphism
* Dependency Injection
* Low coupling / high cohesion
* SOLID

---

## 🎯 Problem

Design a notification system that supports:

```text
Email
SMS
Push Notification
```

The system should allow us to send a notification:

```ts
notificationService.send(
    recipient,
    message,
    notificationType
);
```

### Requirements

1. Initially support **Email, SMS, and Push**.
2. We should be able to add a new channel later, e.g. WhatsApp, **without modifying the existing core notification logic**.
3. Each channel has a different implementation for sending.
4. The notification system should be easy to unit test.
5. Don't worry about actual email/SMS provider APIs yet.
6. Keep the design reasonably simple—we're designing an interview-level LLD, not a production framework.

---

# 🎤 Your turn — interview style

Don't write code yet.

First answer these questions **in your own words**:

### Q1.

What classes/interfaces would you create?

### Q2.

Would you use inheritance, composition, or both? Why?

### Q3.

What should `NotificationService` depend on?

### Q4.

How would you make sure adding `WhatsAppNotification` doesn't require modifying `NotificationService`?

### Q5.

Where would you use Dependency Injection?

---

### One important instruction

**Don't try to give a perfect answer.**

I want to see how you would naturally approach the problem after everything we've learned.

---
I'll review your design **as an interviewer would**, point out what is good, where there's a design smell, and then we'll refine it together.

This exercise is also what will help me decide when we can officially say:

> 🎉 **LLD Fundamentals Completed**
---
### My Thought-
So, we will take it as a notification functionality. For that, I will create an interface called Notification, which will have that contract that which function will be supported. And this Notification interface will be implemented by, for now, let's say, SMS, email, and push notification. And that how we will achieve the abstraction and the polymorphism. And then we will create a Notification Service, and Notification Service should have only the single responsibility, which will be the sending the notification. And it can be extended earlier by using Open and Close Principle. And also, that interface we are making should have the required functionality only. And this should be related to the functionality only, no other functionality should be there. And Notification Service will have this Notification object via dependency injection. And while, whenever user want to send a notification, we will provide that notification object to the Notification Service via dependency injection. And in the future, if there is one more notification to add, so we will not require a Notification Service change because we are providing the notification object via dependency injection. And for this Notification Service and all, actually we can use composition like I just said. And we are using abstraction, polymorphism. And for there, we can use inheritance, but for now, there is only one functionality, so and we are not sure what in the future, so just composition. And if there is more functionality, we can use inheritance as well with the composition. But we are already using the inheritance via implementing the interface. Where would you use dependency injection? So I will use the dependency injection in the Notification Service. So that is all, and I will not be able to write the code for now, but this is how we will proceed with this question. And when you share your review, your input, share in the way that I can put it in my notes as well.

---
Yes — **your overall design is correct**, and more importantly, you're now naturally connecting the principles instead of treating them as separate definitions. That's exactly what I wanted to test with this exercise.

There are just **2–3 important refinements** I'd make.

# 📝 Review of Your Design

### Your proposed design

```text
                    Notification
                         ↑
              ┌──────────┼──────────┐
              │          │          │
            Email        SMS       Push
              │          │          │
              └──────────┼──────────┘
                         │
                         ↓
                NotificationService
                         ↑
                         │
                Dependency Injection
```

This is fundamentally a **good design**.

---

## ✅ 1. Notification interface — Correct

You said:

> "I'll create a Notification interface that defines the contract."

Exactly.

For example:

```ts
interface Notification {
    send(recipient: string, message: string): void;
}
```

Then:

```text
EmailNotification
SmsNotification
PushNotification
```

implement it.

This gives us:

* **Abstraction** → `Notification`
* **Polymorphism** → different implementations
* **OCP** → new notification types can be added

Good.

---

# ✅ 2. NotificationService depending on Notification — Correct

You said:

> "NotificationService will have this Notification object via dependency injection."

Exactly.

Conceptually:

```ts
class NotificationService {
    constructor(
        private notification: Notification
    ) {}

    send(recipient: string, message: string) {
        this.notification.send(recipient, message);
    }
}
```

So:

```text
NotificationService
        │
        │ depends on
        ↓
   Notification
        ↑
   ┌────┼────┐
 Email  SMS Push
```

This gives us:

**DIP + DI + Composition**

---

# ⚠️ 3. One important correction: "Notification object per send"

You said:

> "Whenever user wants to send a notification, we will provide that notification object to NotificationService via dependency injection."

This **can work**, but I'd challenge it in an interview.

If the service is created like:

```ts
new NotificationService(emailNotification)
```

then that particular service is configured for Email.

If we want:

```text
Send Email
Send SMS
Send Push
```

through the same service, we need to decide **where the selection happens**.

For example:

```text
                NotificationService
                       │
                       ↓
                Notification
                       ↑
              selected implementation
```

There are several valid ways to do this.

### Option A — Inject one channel into the service

```ts
new NotificationService(new EmailNotification())
```

Good if the service is specifically configured for one channel.

### Option B — Inject multiple implementations

```text
NotificationService
       │
       ├── Email
       ├── SMS
       └── Push
```

Then select one based on the request.

### Option C — Have a factory/strategy/registry select the implementation

This becomes useful when the number of notification types grows.

**We don't need to solve that complexity yet.**

But this is the main question your design needs to address.

---

# ⚠️ 4. Composition vs inheritance

You said:

> "We can use composition, and if there is more functionality, we can use inheritance as well."

I'd slightly change this thinking.

Don't decide:

> "More functionality → inheritance."

Instead ask:

> **"Is there a genuine IS-A relationship?"**

Here:

```text
EmailNotification IS-A Notification
SMSNotification IS-A Notification
PushNotification IS-A Notification
```

So polymorphism through the `Notification` interface is already appropriate.

Meanwhile:

```text
NotificationService HAS-A Notification
```

So `NotificationService` uses composition.

Therefore we already have both:

```text
IS-A → polymorphism / interface implementation

HAS-A → composition
```

We don't need to introduce class inheritance unless the domain actually benefits from shared base behavior/state.

That's a very important LLD habit.

---

# ✅ 5. SRP — Correct

You said:

> "NotificationService should have only the single responsibility of sending notification."

Mostly correct.

I'd phrase it slightly better:

> **NotificationService is responsible for orchestrating the notification-sending operation, while individual notification implementations are responsible for the channel-specific sending behavior.**

Because otherwise you might accidentally put:

```text
NotificationService
 ├── Email implementation
 ├── SMS implementation
 ├── Push implementation
```

inside it, which would defeat the purpose.

---

# ✅ 6. OCP — Correct

You correctly identified:

```text
Existing NotificationService
          ↓
doesn't need modification
          ↓
WhatsAppNotification added
```

That's exactly what we want.

```text
Notification
     ↑
 ┌───┼────┬────────┐
Email SMS Push WhatsApp
```

Add a new implementation rather than adding:

```ts
if (type === "whatsapp") {
   ...
}
```

to `NotificationService`.

---

# ✅ 7. ISP — Good understanding

You said:

> "The interface should have the required functionality only."

Correct.

Our interface should be focused:

```ts
interface Notification {
    send(...);
}
```

We shouldn't make:

```ts
interface Notification {
    send();
    schedule();
    cancel();
    attachFile();
    trackDelivery();
    sendBulk();
    ...
}
```

unless every implementation genuinely needs those capabilities.

This shows you've understood ISP rather than just memorized it.

---

# 🧠 Your design mapped to the principles

This is the part I'd recommend putting in your notes:

| Principle         | How we apply it                                                            |
| ----------------- | -------------------------------------------------------------------------- |
| **Abstraction**   | `Notification` interface                                                   |
| **Polymorphism**  | Email/SMS/Push implement `Notification`                                    |
| **Composition**   | `NotificationService` HAS-A `Notification`                                 |
| **DI**            | `Notification` implementation is provided from outside                     |
| **SRP**           | Service orchestrates notification; implementations handle channel behavior |
| **OCP**           | Add WhatsApp without modifying core notification logic                     |
| **ISP**           | Keep `Notification` focused on notification behavior                       |
| **DIP**           | `NotificationService` depends on `Notification`, not Email/SMS/Push        |
| **Low coupling**  | Service doesn't depend on concrete channels                                |
| **High cohesion** | Each class has a focused responsibility                                    |

That's a **very strong fundamentals-level design**.

---

# ⭐ The biggest thing I want you to improve

Your knowledge is now good enough that the next step isn't memorizing more principles.

It's learning to ask:

> **"Where does the decision belong?"**

In this problem, the remaining design question is:

> **Who decides whether this particular request should use Email, SMS, or Push?**

That's a genuine LLD question.

We could put that decision in `NotificationService`, a factory, a strategy/registry, or another component depending on the requirements.

And **that is exactly where real LLD starts getting interesting.**

---

## 📝 Notes version

You can save this:

> **Notification System Design**
>
> `Notification` is the abstraction defining the common contract. Email, SMS, and Push provide different implementations, giving us polymorphism.
>
> `NotificationService` depends on the `Notification` abstraction through composition and receives the implementation using dependency injection. This keeps the service loosely coupled to concrete notification channels.
>
> Each notification implementation handles channel-specific behavior, while the service handles the notification workflow/orchestration.
>
> Adding a new channel such as WhatsApp should require adding a new `Notification` implementation rather than modifying the existing notification logic, supporting OCP.
>
> The interface should remain small and focused so implementations are not forced to support unrelated functionality, following ISP.

### One correction to remember

> **Composition and inheritance are not alternatives we choose based on "more functionality." We choose them based on the relationship and design need.**

---

### 🎯 Fundamentals status

**We're not declaring LLD Fundamentals completed quite yet.**

Your understanding of the OOP + SOLID concepts is looking good. I want to do **a couple more application exercises** where you design the classes yourself. That will verify that you can apply the principles under interview pressure.

After that, I'll explicitly give you the:

> 🎉 **LLD Fundamentals Completed**

milestone and we'll move into the next phase.

