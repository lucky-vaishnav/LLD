Let's continue with the **next LLD pattern**.

# Topic 7 — Proxy Pattern

This is an important pattern because it looks similar to **Decorator**, but the **intent is different**. Understanding that distinction is valuable in interviews.

---

## 1. The problem

Suppose we have a service that retrieves sensitive customer information:

```text
CustomerDataService
```

We don't want every caller to directly access it.

We may need:

* Authorization
* Access control
* Logging
* Caching
* Rate limiting
* Lazy loading

Instead of changing the original service, we can put a **Proxy** in front of it.

```text
Caller
  ↓
Proxy
  ↓
Real Service
```

The caller interacts with the Proxy as if it were the real service.

---

## 2. Basic example

Our contract:

```ts
interface CustomerData {
  getCustomer(id: string): Customer;
}
```

Real implementation:

```ts
class RealCustomerDataService implements CustomerData {
  getCustomer(id: string): Customer {
    console.log("Fetching customer from DB");

    return {
      id,
      name: "Lucky"
    };
  }
}
```

Now create a Proxy:

```ts
class CustomerDataProxy implements CustomerData {
  constructor(
    private realService: CustomerData
  ) {}

  getCustomer(id: string): Customer {
    console.log("Checking authorization...");

    // authorization logic

    return this.realService.getCustomer(id);
  }
}
```

Usage:

```ts
const service = new CustomerDataProxy(
  new RealCustomerDataService()
);

service.getCustomer("123");
```

The caller doesn't directly interact with:

```text
RealCustomerDataService
```

It interacts with:

```text
CustomerDataProxy
```

---

# 3. Why call it a Proxy?

Because the Proxy **stands in front of the real object**.

```text
Caller
  ↓
Proxy
  ↓
Real Object
```

The Proxy controls access to the real object.

This is the key idea.

---

# 4. Common uses

### Authorization

```text
Caller
 ↓
AuthorizationProxy
 ↓
RealService
```

Proxy checks:

```text
"Is this user allowed?"
```

before forwarding the request.

---

### Caching

```text
Caller
 ↓
CacheProxy
 ↓
RealService
```

```text
if cached:
    return cached value

else:
    call real service
```

---

### Lazy loading

```text
Caller
 ↓
Proxy
 ↓
Real Object
```

The real object isn't created/loaded until it's actually needed.

---

### Rate limiting

```text
Caller
 ↓
RateLimitProxy
 ↓
RealService
```

Proxy checks:

```text
"Has this client exceeded the limit?"
```

before allowing access.

---

# 5. Proxy vs Decorator

This is the **most important part of today's topic**.

They can look almost identical structurally:

```text
Caller
  ↓
Wrapper
  ↓
Real Object
```

But their **intent is different**.

### Decorator

> **Adds responsibilities/behavior to an object.**

```text
Payment
 ↓
Logging
 ↓
Retry
 ↓
Payment
```

Focus:

```text
"How can I add behavior?"
```

---

### Proxy

> **Controls access to an object.**

```text
Caller
 ↓
Authorization Proxy
 ↓
Real Service
```

Focus:

```text
"Should this request be allowed to reach the real object?"
```

---

# 6. But what about logging?

This is where things become interesting.

A Proxy **can** perform logging.

A Decorator **can** also perform logging.

So you **cannot identify the pattern just from the code**.

You need to look at the **intent**.

### If the primary purpose is:

```text
Control access
Authorization
Caching
Lazy loading
Rate limiting
```

→ **Proxy**

### If the primary purpose is:

```text
Add logging
Add metrics
Add retry
Add compression
Add additional behavior
```

→ **Decorator**

---

# 7. Proxy vs Adapter

Another important comparison:

### Adapter

```text
Application
    ↓
Adapter
    ↓
External API
```

Purpose:

> **Make incompatible interfaces compatible.**

### Proxy

```text
Application
    ↓
Proxy
    ↓
Same Real Interface
```

Purpose:

> **Control access to the real object.**

Notice:

```text
Adapter:
Application interface ≠ External interface

Proxy:
Application interface = Proxy interface = Real object interface
```

That's a very useful interview distinction.

---

# 8. Proxy vs Facade

### Facade

Hides **multiple components**:

```text
             Facade
                ↓
       ┌────────┼────────┐
       ↓        ↓        ↓
    Service  Service   Service
```

### Proxy

Usually stands in front of **one real object/service**:

```text
Proxy
  ↓
Real Object
```

So:

> **Facade simplifies a subsystem.**

> **Proxy controls access to an object.**

---

# 9. Senior-level observation

You may notice something:

```text
Decorator
Proxy
Adapter
```

can all have a similar structure:

```text
Wrapper
  ↓
Real/Other Object
```

Don't identify a pattern based only on structure.

Instead ask:

> **"What problem is this wrapper solving?"**

That's the important interview skill.

```text
Adapter
→ Interface compatibility

Decorator
→ Add behavior

Proxy
→ Control access

Facade
→ Simplify complexity
```

---

# 🧠 Your turn

Let's use a backend example.

### Scenario

We have:

```text
ReportService
```

It generates an expensive report from the database.

We want to introduce:

```text
1. Authorization check
2. Cache
3. Rate limiting
```

without modifying `ReportService`.

### Questions

**Q1.** Would you use Proxy, Decorator, or both?

**Q2.** What should the common interface look like?

**Q3.** What would the Proxy be responsible for?

**Q4.** If we add logging around `ReportService`, would that automatically make it a Proxy? Why/why not?

**Q5.** What's the simplest difference between Proxy and Decorator?

Answer these in your own words. Then I'll review them and give you the **interview + notes version**.

---
### My Thoughts-
So as we know that decorator, we should think about decorator when we want to add behavior, and that is how we can use decorator. And so it totally depends upon how we are thinking about implementation or the structure. So let's say that authorization, caching, and rate limiting, all these operations are required when the reports service is called. So we can use proxy here because these are not optional according to the business rule. All are required at once, and we can use a proxy. And for proxy as well, we can generalize the proxy, which can have these three operations together, and then calling the reports service. And proxy can be generalized as a report API proxy, something like that. Or if it required, but it should not, I think, it will just add more and more implementation. But if you think that making different implementation for authorization, caching, and rate limiting, then we can use multiple proxies here. But how these multiple proxies will be chained together, that is the thinking point. And yeah, proxy will be responsible for authorization, caching, rate limiting, and then calling the report service. And for logging, so it is totally if it is optional or required, or it can be changed in future, logging may not require, something like that, then we can use a decorator as well, which will be used in the report service instead if we just wanted to log about the reports. And simplest difference between proxy and decorator is that decorator is mostly about adding the behavior, but proxy is all about controlling the access of a functionality or a behavior. So that can be interrelated, but mostly it is about controlling the access and how we are thinking about the object or the code structure.

---
Yes — **your core answer is strong**, and you've picked up the most important part of Proxy: **intent matters more than the wrapper's structure**.

A few refinements will make it interview-ready.

## Q1. Proxy, Decorator, or both? — ✅

You said Proxy because:

> authorization, caching, and rate limiting are required when the ReportService is called.

That's reasonable, but **"required" vs "optional" isn't the deciding factor**.

The stronger reasoning is:

> **If the primary purpose is to control/protect access to `ReportService`, Proxy is appropriate.**

For example:

```text
Caller
  ↓
ReportProxy
  ├── Authorization
  ├── Rate limiting
  ├── Cache check
  ↓
ReportService
```

That's a Proxy.

And yes, you could technically have multiple proxies:

```text
Caller
  ↓
AuthorizationProxy
  ↓
RateLimitProxy
  ↓
CacheProxy
  ↓
ReportService
```

But whether you actually want that depends on complexity and architecture. You don't need to force every concern into a separate Proxy.

---

## Q2. Common interface — You didn't explicitly answer, but the idea is:

```ts
interface ReportService {
  generateReport(request: ReportRequest): Report;
}
```

Then:

```text
ReportService
      ↑
 ┌────┴───────────┐
 ↓                ↓
RealReportService ReportProxy
```

Both expose the same interface.

---

## Q3. Proxy responsibility — ✅

You correctly identified:

```text
Authorization
Caching
Rate Limiting
      ↓
Real ReportService
```

One refinement:

> **The Proxy should control access/interaction with the real service, but the actual report-generation business logic should remain in `ReportService`.**

So:

```text
Proxy
→ "Can/should this request reach the service?"

ReportService
→ "How do I generate the report?"
```

---

## Q4. Logging doesn't automatically mean Proxy — ✅

Exactly.

This is a very important point.

If we have:

```text
ReportService
    ↓
Logging
```

we cannot say:

> "It's a Proxy because it logs."

Both Proxy and Decorator can log.

We ask:

> **What is the intent?**

If:

```text
Logging
Metrics
Retry
Extra behavior
```

are being added to the object:

→ **Decorator**

If:

```text
Authorization
Rate limiting
Access control
Lazy loading
Cache/access control
```

are being used primarily to control whether/how the real object is accessed:

→ **Proxy**

---

# Q5. Simplest difference — ✅

Your answer was good:

> **Decorator = add behavior. Proxy = control access.**

I'd make it even sharper:

> **Decorator changes or extends what the object does.**

> **Proxy controls how/when the object can be accessed.**

That's a very good interview distinction.

---

# One important correction about your "optional vs required" thought

You said:

> "If these are not optional according to business rule, we can use Proxy."

Don't use **required/optional** as your main rule.

For example, suppose logging is mandatory:

```text
Every ReportService call MUST be logged.
```

It could still be implemented with a **Decorator**.

Likewise, authorization might be implemented using middleware, a Proxy, an API gateway, etc.

The pattern depends primarily on the **intent and responsibility**.

---

# 🧠 Your notes

### Proxy Pattern

> **Proxy provides a substitute object with the same interface as the real object and controls access to, or interaction with, that real object.**

```text
Caller
  ↓
Proxy
  ↓
Real Object
```

### Common uses

* Authorization
* Access control
* Rate limiting
* Caching
* Lazy initialization
* Remote access

### Key distinction

```text
Adapter
→ Make incompatible interfaces compatible

Facade
→ Hide subsystem complexity

Decorator
→ Add/compose behavior

Proxy
→ Control access to an object
```

### ⭐ Most important interview point

> **Proxy and Decorator can have almost the same structure. The difference is primarily their intent.**

```text
Decorator:
"How can I add behavior?"

Proxy:
"How should access to the real object be controlled?"
```

And your observation that **multiple proxies can be chained** is correct. That's worth remembering because it connects nicely with the composition concepts you've already learned.
