**Topic 12 — Chain of Responsibility Pattern**.

This one is especially relevant for backend development because it maps naturally to **middleware, validation pipelines, authorization, logging, request processing, and approval flows**.

# Chain of Responsibility

## 1. The problem

Imagine a report API request:

```text
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Rate Limit
 ↓
Validation
 ↓
Report Service
```

A naive implementation could become:

```ts
if (!authenticated) {
  // reject
}

if (!authorized) {
  // reject
}

if (rateLimitExceeded) {
  // reject
}

if (!validRequest) {
  // reject
}

generateReport();
```

As more processing rules are added, this can become difficult to maintain.

---

# 2. Core idea

Instead of one large component handling everything, we create **handlers**, where each handler performs one responsibility and can pass the request to the next handler.

```text
Request
   ↓
AuthHandler
   ↓
AuthorizationHandler
   ↓
RateLimitHandler
   ↓
ValidationHandler
   ↓
ReportService
```

Each handler decides:

> **Should I handle/reject this request, or should I pass it to the next handler?**

That's the core idea.

---

# 3. Handler interface

```ts
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: Request): void;
}
```

Base handler:

```ts
abstract class BaseHandler implements Handler {
  protected next?: Handler;

  setNext(handler: Handler): Handler {
    this.next = handler;
    return handler;
  }

  handle(request: Request): void {
    if (this.next) {
      this.next.handle(request);
    }
  }
}
```

Now concrete handlers can focus on their own responsibility.

---

# 4. Authentication Handler

```ts
class AuthenticationHandler extends BaseHandler {

  handle(request: Request): void {
    if (!request.user) {
      throw new Error("Unauthenticated");
    }

    super.handle(request);
  }
}
```

Authorization:

```ts
class AuthorizationHandler extends BaseHandler {

  handle(request: Request): void {
    if (!request.user.canViewReports) {
      throw new Error("Unauthorized");
    }

    super.handle(request);
  }
}
```

Validation:

```ts
class ValidationHandler extends BaseHandler {

  handle(request: Request): void {
    if (!request.reportId) {
      throw new Error("Invalid request");
    }

    super.handle(request);
  }
}
```

---

# 5. Building the chain

```ts
const auth = new AuthenticationHandler();
const authorization = new AuthorizationHandler();
const validation = new ValidationHandler();

auth
  .setNext(authorization)
  .setNext(validation);
```

Now:

```ts
auth.handle(request);
```

Flow:

```text
Authentication
      ↓
Authorization
      ↓
Validation
      ↓
End
```

If authentication fails:

```text
Authentication
      ↓
❌ STOP
```

The remaining handlers aren't executed.

---

# 6. Why is this useful?

Each handler has a focused responsibility:

```text
AuthenticationHandler
→ authentication

AuthorizationHandler
→ authorization

ValidationHandler
→ validation
```

We can add another handler:

```text
RateLimitHandler
```

without changing:

```text
AuthenticationHandler
AuthorizationHandler
ValidationHandler
```

This gives us a nice **Open/Closed Principle** benefit.

---

# 7. Very important: Middleware connection

Since you're a Node.js developer, this should immediately look familiar.

Express middleware:

```ts
app.use(authMiddleware);
app.use(authorizationMiddleware);
app.use(validationMiddleware);
```

Conceptually:

```text
Request
  ↓
Middleware 1
  ↓ next()
Middleware 2
  ↓ next()
Middleware 3
  ↓ next()
Controller
```

This is very similar to **Chain of Responsibility**.

So if an interviewer asks:

> "Can middleware be considered an implementation of Chain of Responsibility?"

A good answer is:

> **"Yes, conceptually. Middleware forms a chain where each middleware can process the request and either terminate it or pass it to the next middleware. The exact implementation details can differ, but the underlying flow is very similar to Chain of Responsibility."**

---

# 8. Chain of Responsibility vs Decorator

This is important because you've already learned Decorator.

They can look similar:

```text
A → B → C
```

But their intent differs.

### Decorator

Each wrapper generally **adds behavior** around the underlying object.

```text
Logging
  ↓
Retry
  ↓
PaymentService
```

The goal:

> **Enhance behavior.**

---

### Chain of Responsibility

Each handler gets a chance to **handle or pass along the request**.

```text
Auth
 ↓
Authorization
 ↓
Validation
 ↓
Controller
```

The goal:

> **Give multiple handlers a chance to process a request.**

### Easy memory

> **Decorator → add behavior.**

> **Chain → pass responsibility along.**

---

# 9. Chain of Responsibility vs State

Another useful distinction:

### State

```text
Trip
 ↓
Current State
```

The state determines the object's behavior.

### Chain

```text
Request
 ↓
Handler A
 ↓
Handler B
 ↓
Handler C
```

The request moves through a sequence of handlers.

---

# 10. Important practical consideration

A chain doesn't necessarily mean **every handler must pass the request onward**.

A handler can:

```text
Handle → STOP
```

or:

```text
Don't handle → NEXT
```

For example:

```text
Authentication
     ↓
invalid
     ↓
STOP

valid
 ↓
Authorization
```

This is the key behavior.

---

# 🧠 Your turn

Let's use the **Report API** example:

```text
Authentication
Authorization
Rate Limiting
Validation
Report Service
```

### Q1.

What would be the Handler interface?

### Q2.

What would be the concrete handlers?

### Q3.

What happens if the Authentication Handler rejects the request?

### Q4.

Why is Chain of Responsibility better than putting all validation/authentication logic in one large service?

### Q5.

Can Express/NestJS middleware be considered an example of Chain of Responsibility? Why?

### Q6.

What is the main difference between **Chain of Responsibility and Decorator**?

Answer in your own words, and I'll review it and create the concise interview/notes version afterward.

---
### My Thoughts-
So handler interface will be set next and handler function, and we can call it base handler. And control handler will be authentication, authorization, rate limiting, validation, and report service will be the final component which will be called. If authentication rejects the request, the request will not go to rest of the handler. So it will just return the response from there only, like the validation. Putting validation, authentication in one large service is not a good idea because it will not follow the OOPs concept as well, or single responsibility functionality, and rest of, and the code will be messy. Yeah, we can consider express and NestJS middleware as a chain of responsibility if we use those like the same way using the next. Chain of responsibility is mostly about that there are multiple behavior of object or functionality we want, some validation we want, and we can divide the responsibility in chainy according to the sequence wise. So it will be chain of responsibility. Decorator function is where the object updates its behavior according to what we require. Let's say we want logging after payment, then we can add that decorator as well. So it is the behavior of the object is changed, or maybe not a behavior, then we want some additional functionality with that flow, we can use that decorator. Chain of responsibility is just assigned using the request and handling the subpart of functionalities.

---
Yes — overall **your understanding is correct**, especially the middleware connection. A few points need tightening for an interview answer.

## Q1. Handler interface — ✅

You said:

> `setNext` and `handle`.

Correct.

A clean contract is:

```ts
interface Handler {
  setNext(handler: Handler): Handler;
  handle(request: Request): void;
}
```

You can have a `BaseHandler` to provide the common `next` handling, but **BaseHandler itself isn't the pattern's required interface**.

---

## Q2. Concrete handlers — ✅

Your chain is correct:

```text
AuthenticationHandler
        ↓
AuthorizationHandler
        ↓
RateLimitHandler
        ↓
ValidationHandler
        ↓
ReportService
```

One nuance: technically, `ReportService` doesn't necessarily need to be a **Handler**. It can simply be the final receiver/business service after the chain.

---

## Q3. Authentication rejects — ✅

Exactly.

```text
Request
  ↓
Authentication
  ↓
❌ Reject
```

The next handlers aren't called.

That's one of the defining characteristics of the pattern:

> **A handler can either process/terminate the request or pass it to the next handler.**

---

## Q4. Why not one large service? — ✅, but improve the reasoning

You said it would violate OOP/SRP and make the code messy.

The stronger answer is:

> **Each handler has one focused responsibility, making the chain easier to maintain, test, reorder, and extend.**

For example:

```text
AuthenticationHandler
→ only authentication

RateLimitHandler
→ only rate limiting

ValidationHandler
→ only validation
```

Adding:

```text
AuditHandler
```

doesn't require modifying the existing handlers.

That's a clearer **SRP + OCP** argument than simply saying "it doesn't follow OOP."

---

# Q5. Express/NestJS middleware — ✅

Yes.

Your understanding of `next()` is exactly the connection.

Conceptually:

```text
Request
 ↓
Middleware A
 ↓ next()
Middleware B
 ↓ next()
Middleware C
 ↓ next()
Controller
```

A middleware can also stop the chain:

```text
Middleware A
 ↓
❌ response
```

So your interview answer can be:

> **"Express or NestJS middleware is conceptually similar to Chain of Responsibility because middleware forms a chain where each component can process the request, terminate it, or pass it to the next component using `next()`."**

I'd say **"conceptually similar"** rather than claiming that every middleware implementation is literally the GoF pattern.

---

# Q6. Chain vs Decorator — ⚠️ Your idea is right, but sharpen it

You said:

> "Decorator is where the object updates its behavior... additional functionality."

Good direction.

But the easiest distinction is:

### Decorator

```text
Object
 ↓
Decorator A
 ↓
Decorator B
 ↓
Object
```

The purpose is:

> **Add/compose additional behavior around an object.**

Example:

```text
PaymentService
 ↓
LoggingDecorator
 ↓
RetryDecorator
 ↓
PaymentService
```

---

### Chain of Responsibility

```text
Request
 ↓
Auth
 ↓
Authorization
 ↓
RateLimit
 ↓
Validation
```

The purpose is:

> **Give multiple handlers an opportunity to process the request, with each handler deciding whether to continue or stop.**

### ⭐ Best memory line

> **Decorator → add behavior.**

> **Chain of Responsibility → pass responsibility.**

That's probably the cleanest distinction for your notes.

---

# One correction to your definition

You said:

> "Chain of responsibility is mostly about multiple behavior of object."

I would avoid that wording.

It's **not primarily about multiple behaviors of an object**.

It's about:

> **A request being passed through a sequence of handlers, where each handler can handle the request or pass it to the next handler.**

That distinction will prevent confusion with **Strategy** and **Decorator**.

---

# 🧠 Final notes

### Chain of Responsibility

> **Passes a request through a chain of handlers, where each handler can process the request or pass it to the next handler.**

### Use when:

* Multiple processing steps are required
* Each step has a focused responsibility
* A handler may stop the request
* Order of processing matters
* You want to add/remove/reorder handlers easily

### Example

```text
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Rate Limit
 ↓
Validation
 ↓
Report Service
```

### Comparison

```text
Strategy
→ Choose one behavior

Decorator
→ Add/compose behavior

Chain
→ Pass request through handlers
```

And this pattern is particularly worth remembering for you as a Node.js developer because **middleware is one of the most natural real-world examples of the Chain of Responsibility concept**.
