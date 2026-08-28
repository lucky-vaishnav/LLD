# Topic 14 — Singleton Pattern

This is a short but **very interview-relevant** pattern because interviewers often care more about **when NOT to use Singleton** than simply knowing how to implement it.

---

## 1. What problem does Singleton solve?

Sometimes we want:

> **Only one instance of a particular class to exist within a given application/process, with a common access point to it.**

For example, conceptually:

```text
Application
     ↓
  Config
     ↓
Only one instance
```

Instead of:

```ts
const config1 = new Config();
const config2 = new Config();
const config3 = new Config();
```

we want:

```text
config1 === config2 === config3
```

---

# 2. Basic implementation

In TypeScript:

```ts
class Config {
  private static instance: Config;

  private constructor() {}

  static getInstance(): Config {
    if (!Config.instance) {
      Config.instance = new Config();
    }

    return Config.instance;
  }
}
```

Usage:

```ts
const config1 = Config.getInstance();
const config2 = Config.getInstance();

console.log(config1 === config2);
```

Result:

```text
true
```

The important parts are:

### Private constructor

```ts
private constructor() {}
```

This prevents:

```ts
new Config();
```

from outside the class.

### Static instance

```ts
private static instance: Config;
```

Stores the single instance.

### Static accessor

```ts
getInstance()
```

returns the existing instance or creates it if it doesn't exist.

---

# 3. Why would we want one instance?

Consider application configuration:

```ts
class AppConfig {
  // configuration
}
```

You generally don't want different parts of the application creating independent configuration objects with potentially different state.

Conceptually:

```text
        AppConfig
           ↑
     ┌─────┼─────┐
     ↓     ↓     ↓
 Service  API   Worker
```

All can access the same instance.

---

# 4. But here's the important part

**Singleton is often overused.**

A common interview mistake is:

> "We should use Singleton whenever we want a shared object."

Not necessarily.

In modern backend applications, especially Node.js/NestJS, **dependency injection is often a better way to manage shared instances**.

For example, NestJS can manage provider lifecycle and give you a singleton-scoped provider by default.

You don't necessarily need to manually implement:

```ts
getInstance()
```

---

# 5. Singleton vs Dependency Injection

This is important given what you've learned already.

### Singleton

The class controls its own instance:

```text
Caller
  ↓
Config.getInstance()
  ↓
Singleton
```

The class decides:

> "There can only be one of me."

### Dependency Injection

The dependency is supplied from outside:

```text
Container
   ↓
Config instance
   ↓
Service
```

The service doesn't care how the dependency was created.

This is generally more flexible and testable.

---

# 6. Testing problem

Suppose:

```ts
class Logger {
  private static instance: Logger;

  static getInstance() {
    // ...
  }
}
```

Then:

```ts
const logger = Logger.getInstance();
```

Your code is directly coupled to the global Singleton.

Testing can become harder because:

```text
Test A
 ↓
changes Singleton state

Test B
 ↓
gets same Singleton state
```

That shared state can create test-order problems.

With DI:

```ts
class PaymentService {
  constructor(
    private logger: Logger
  ) {}
}
```

we can easily inject:

```ts
const mockLogger = new MockLogger();

const service = new PaymentService(mockLogger);
```

Much easier to isolate.

---

# 7. Singleton vs Static class

Another common interview question.

They look similar, but they're conceptually different.

### Static class-style approach

```ts
Logger.log("Payment completed");
```

You don't have an object instance.

### Singleton

```ts
const logger = Logger.getInstance();

logger.log("Payment completed");
```

There is an actual object instance, but only one is intended to exist.

Singleton can therefore:

* implement interfaces
* be passed around
* maintain instance state
* participate in polymorphism

---

# 8. Node.js perspective

You may hear:

> "Node.js modules are already effectively singleton-like."

There is some truth here.

When a module is imported, Node.js caches the loaded module, so repeated imports generally reference the same module instance within that process.

For example:

```ts
// config.ts

export const config = {
  environment: "production"
};
```

Other modules importing it will generally receive the same exported object within that Node.js process.

So manually writing:

```ts
class Config {
  private static instance...
}
```

is often unnecessary.

Again:

> **Don't use Singleton simply because you need a shared object in Node.js.**

---

# 9. Important limitation

Singleton means **one instance within the relevant runtime/process**, not necessarily one instance across your entire distributed application.

For example:

```text
Load Balancer
    ↓
 ┌───────┬───────┐
 ↓       ↓       ↓
Node 1  Node 2  Node 3
```

Each process can have its **own Singleton instance**.

So:

```text
Singleton ≠ distributed global singleton
```

If you need shared state across servers, you need something like an external/shared system rather than an in-process Singleton.

---

# 🧠 Your turn

Let's use a **Logger** example.

### Q1.

What is the main purpose of the Singleton Pattern?

### Q2.

Why do we make the constructor private?

### Q3.

Why is Singleton often considered problematic in large applications?

### Q4.

What is the difference between Singleton and Dependency Injection?

### Q5.

If you're building a NestJS application and want one shared `ConfigService`, would you necessarily implement Singleton manually? Why?

### Q6.

If you have 3 Node.js server instances behind a load balancer, will a Singleton give you one instance across all 3 servers?

### Q7.

Would you use Singleton just because multiple parts of the application need the same service? Explain your thought process.

Answer in your own words as usual. This one is important because **the best interview answer isn't "here's how to implement Singleton" — it's knowing when Singleton is actually justified and when DI is the better choice.**

---
### My Thoughts -
So the main purpose of Singleton pattern is to have a single instance of a class, and we can use it according to the requirement if we really want that only one instance is required, and we need to make the constructor private for this so that no object can be created outside of the class. And Singleton often considered problematic in large application. If we are talking about large means distributed system, then yeah Singleton will not be properly used for what it should be used to have a single instance. So it can have a single instance. And large application means that maybe for some application part we need a new instance, then yeah Singleton can be problematic. And difference between Singleton and Dependency Injection: Singleton, we only create the object within the class and then the same instance will be used. And Dependency Injection follow the composition technique where we inject the required dependency in the constructor according to the requirement. So it gives us the flexibility. And as you said that Nest.js automatically provides the Singleton type of implementation, so we do not need to implement Singleton in Nest.js. And if there are multiple Node.js server instances behind the load balancer, then Singleton will not give us one instance, as multiple instance is running on different server. So would I use Singleton just because multiple parts of the application need the same service? So it totally depends upon the thought process and how we want to implement it. If using Singleton we can achieve that, then yeah. But nowadays the application, if the application is big, then the Singleton approach will not achieve the actual behavior as there will be multiple instance running. But yeah, for the small application, if it is required, then we can use it, I think.

---
Yes — **your core understanding is correct**, but there are two important refinements I'd make, especially for a senior-level interview.

## 1. Main purpose — ✅

Correct:

> Singleton ensures that only one instance of a class exists within a particular application runtime/process and provides a common access point to it.

The **process/runtime scope** is important.

---

## 2. Private constructor — ✅

Exactly.

```ts
private constructor() {}
```

This prevents callers from doing:

```ts
new Logger();
```

and forces access through something like:

```ts
Logger.getInstance();
```

---

## 3. Why Singleton can be problematic — ⚠️ Important refinement

You focused mainly on distributed systems.

That's **one problem**, but not the main reason Singleton is often criticized.

Even in a **single Node.js process**, Singleton can cause problems because it introduces **global/shared state and tight coupling**.

For example:

```ts
const logger = Logger.getInstance();
```

Now the consumer is directly coupled to `Logger`.

Testing becomes harder:

```text
Service
   ↓
Logger.getInstance()
```

instead of:

```text
Service
   ↓
Logger injected
```

With DI, we can easily replace it:

```ts
new Service(mockLogger);
```

So the stronger interview answer is:

> **Singleton can introduce global state, hidden dependencies, tight coupling, and testing difficulties. In distributed systems, it also doesn't provide a single instance across processes/servers.**

That's a much stronger answer.

---

# 4. Singleton vs DI — ✅ Very good

You said:

> Singleton creates/manages the object itself, while DI provides the dependency externally.

Exactly.

I'd put it in your notes as:

```text
Singleton
→ Object controls its own single instance.

DI
→ Dependency is supplied from outside.
```

And therefore:

```text
Singleton
→ tighter coupling

DI
→ more flexible/testable
```

One small correction: **DI isn't specifically "composition technique."**

Composition is often the underlying OOP relationship:

```ts
class PaymentService {
  constructor(private gateway: PaymentGateway) {}
}
```

DI is the **mechanism of supplying that dependency**.

---

# 5. NestJS — ✅

Correct.

NestJS's default provider scope is singleton, so you generally **don't manually implement the Singleton pattern** just because you want a shared service.

For example, conceptually:

```text
NestJS DI Container
       ↓
   ConfigService
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
A      B     C
```

The container manages the shared instance.

That's preferable to:

```ts
ConfigService.getInstance();
```

---

# 6. Multiple Node.js servers — ✅

Exactly.

```text
Load Balancer
   ├── Node 1 → Singleton A
   ├── Node 2 → Singleton B
   └── Node 3 → Singleton C
```

So:

> **Singleton means one instance per process/runtime, not one instance across a distributed system.**

Very important interview point.

---

# 7. "Would I use Singleton just because multiple parts need the same service?" — ⚠️

This is the biggest thing I'd change in your answer.

You said:

> "If using Singleton we can achieve that, then yeah."

I would **not** recommend saying this in an interview.

Multiple components needing the same service is **not sufficient justification** for Singleton.

Instead:

> **First consider DI/container-managed lifecycle. Use Singleton only when there is a genuine invariant that the application requires exactly one instance within that process.**

For example, if the requirement is:

> "There must be exactly one instance of X within this process."

Then Singleton can make sense.

But:

> "Multiple services need Logger."

doesn't mean:

> "Therefore Logger should be Singleton."

DI can provide a shared instance without making the class responsible for enforcing Singleton.

---

# ⭐ Your interview answer

If an interviewer asks:

**"Would you use Singleton in a Node.js application?"**

A strong senior-level answer would be:

> "I wouldn't use Singleton just to share an object between services. In a Node.js/NestJS application, I'd generally prefer dependency injection and let the DI container manage the provider lifecycle. Singleton can be appropriate when having exactly one instance per process is a real business or technical requirement, but it introduces global/shared state and can make testing and dependency management harder. Also, in a distributed deployment, each process would have its own Singleton."

That's a **very good interview answer**.

---

# 🧠 Final notes

### Singleton

> **Ensures only one instance of a class exists within a process/runtime and provides a common access point to it.**

### Why private constructor?

> Prevents external instantiation.

### Main concerns

* Global/shared state
* Tight coupling
* Hidden dependencies
* Harder testing
* One instance only per process, not distributed system-wide

### Singleton vs DI

```text
Singleton
→ Class controls instance creation/access.

DI
→ External container/caller provides dependency.
```

### Node.js/NestJS

> Prefer **DI/container-managed lifecycle** over manually implementing Singleton in most application code.

### Mental rule

> **"Shared dependency" ≠ automatically "Singleton."**

> **"Exactly one instance is required" → consider Singleton.**

You're doing well with these patterns. At this point, the more valuable skill is becoming **pattern selection** rather than memorizing implementations—which is exactly what we're practicing.


# Q1. Rephrased question

> **If shared dependency alone is not a reason to use Singleton, what is a practical example where Singleton is genuinely useful?**

### Answer

A good example is a **process-level resource that should have exactly one instance**.

For example, suppose you have an in-memory metrics collector:

```ts
class Metrics {
  private static instance: Metrics;

  private counters = new Map<string, number>();

  private constructor() {}

  static getInstance(): Metrics {
    if (!Metrics.instance) {
      Metrics.instance = new Metrics();
    }

    return Metrics.instance;
  }

  increment(name: string) {
    this.counters.set(
      name,
      (this.counters.get(name) || 0) + 1
    );
  }

  get(name: string) {
    return this.counters.get(name) || 0;
  }
}
```

Different parts of the application can use:

```ts
Metrics.getInstance().increment("payment.success");
```

and:

```ts
Metrics.getInstance().increment("payment.success");
```

Both are operating on the **same Metrics object**.

Why might exactly one instance make sense?

Because you want one in-memory metrics registry for that Node.js process rather than accidentally creating independent registries:

```text
❌ Metrics A
   → payment.success = 5

❌ Metrics B
   → payment.success = 3
```

Instead:

```text
✅ One Metrics instance
   ├── Service A → +1
   ├── Service B → +1
   └── Service C → +1
```

### Important caveat

This is **one instance per Node.js process**.

If you have:

```text
Node 1 → Metrics A
Node 2 → Metrics B
Node 3 → Metrics C
```

they are still separate.

So Singleton is useful when the requirement is specifically:

> **"Exactly one instance within this process."**

---

# Q2. Rephrased question

> **When we say Node.js shares the same instance when a module is imported/exported, what does "same instance" actually mean? If one part of the application modifies the object, will another part see that modification?**

### Answer: Yes — exactly.

This is the important part.

Suppose you have:

```ts
// config.ts

export const config = {
  environment: "development"
};
```

Then:

```ts
// serviceA.ts

import { config } from "./config";

config.environment = "production";
```

And somewhere else:

```ts
// serviceB.ts

import { config } from "./config";

console.log(config.environment);
```

It will see:

```text
production
```

because both are referencing the **same exported object within that Node.js process**.

Conceptually:

```text
             config object
                   ↑
          ┌────────┴────────┐
          │                 │
      serviceA          serviceB
          │                 │
      modifies it       sees change
```

So when we say **shared instance**, we mean:

> **The consumers hold a reference to the same object in memory.**

It is **not** a template.

---

## Template vs Instance

This distinction is important.

A class:

```ts
class Config {
  environment = "development";
}
```

is a **template/blueprint**.

An object:

```ts
const config = new Config();
```

is an **instance**.

If both services receive:

```ts
config
```

they receive the same object reference.

---

# Q3. Rephrased question

> **Since Node.js module caching and NestJS dependency injection can already provide shared instances, do we ever need to manually implement the Singleton pattern in Node.js/NestJS? Or should Singleton always be avoided there?**

### Answer

Don't make it an absolute:

> ❌ "Singleton is never required in Node.js/NestJS."

That's too strong.

Instead:

> **Manual Singleton implementation is usually unnecessary in Node.js/NestJS application code because the runtime/framework already provides mechanisms for sharing instances and managing their lifecycle.**

### In plain Node.js

Node.js caches modules after they are loaded.

So you can have:

```ts
// logger.ts

export const logger = new Logger();
```

Then:

```ts
// serviceA.ts
import { logger } from "./logger";
```

and:

```ts
// serviceB.ts
import { logger } from "./logger";
```

Both generally receive the same exported object **within that Node.js process**.

You don't necessarily need:

```ts
Logger.getInstance();
```

---

# NestJS makes this even more explicit

NestJS has a dependency-injection container.

A provider is **singleton-scoped by default**.

So you can have:

```ts
@Injectable()
export class ConfigService {
  // ...
}
```

and:

```ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly configService: ConfigService
  ) {}
}
```

Another service:

```ts
@Injectable()
export class ReportService {
  constructor(
    private readonly configService: ConfigService
  ) {}
}
```

The NestJS container manages the provider.

Conceptually:

```text
             NestJS DI Container
                    │
             ConfigService
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
   PaymentService       ReportService
```

They can receive the **same ConfigService instance** according to the provider's scope.

So you generally don't need:

```ts
ConfigService.getInstance();
```

---

# ⭐ The important distinction

There are actually **three different ideas** here:

### 1. Manual Singleton Pattern

The class itself controls its single instance:

```ts
Config.getInstance();
```

### 2. Node.js module sharing

The module system/cache means an exported object can be shared within a process:

```ts
import { config } from "./config";
```

### 3. NestJS DI singleton scope

The **DI container** creates and manages one provider instance by default.

```ts
constructor(private config: ConfigService) {}
```

These can produce a similar **"one shared instance"** result, but they are **not the same mechanism**.

---

# 🧠 The interview takeaway

If an interviewer asks:

### "Do you use Singleton in NestJS?"

Don't say:

> "No, Singleton is bad."

Instead say:

> **"I generally wouldn't implement Singleton manually in NestJS. NestJS's DI container manages provider lifecycles and providers are singleton-scoped by default. I'd use DI unless I have a specific reason to enforce a process-level singleton myself."**

And if they ask:

### "Does NestJS Singleton mean one instance across all servers?"

No.

```text
Load Balancer
   │
   ├── Node Process 1 → ConfigService A
   ├── Node Process 2 → ConfigService B
   └── Node Process 3 → ConfigService C
```

Each process has its own instance.

---

## Final notes for you

> **Shared instance = same object/reference in memory, not a template.**

> **Node.js module exports can share the same object within a process because of module caching.**

> **NestJS providers are singleton-scoped by default and managed by the DI container.**

> **Manual Singleton is usually unnecessary in NestJS, but "never use Singleton" is too absolute.**

> **Singleton means one instance per process—not one instance across a distributed system.**

And one subtle but important point for your senior-level understanding: **"singleton-like behavior" provided by Node.js module caching or NestJS DI is not automatically the GoF Singleton Pattern.** The *result* can be similar (one shared instance), but the *design mechanism and ownership of instance creation* are different.

Yes — and your question is really about **how NestJS controls provider lifetime/scope**, not about the Singleton pattern itself.

## Q4

> **In NestJS, providers are singleton-scoped by default. Do we need to explicitly implement anything in the provider class to make it singleton, or does NestJS handle it automatically? If we don't want the default singleton behavior, how do we change the provider's lifecycle? Also, when would we actually want different instances of the same provider in different consumers?**

### 1. Do we need to manually make `ConfigService` singleton?

**No.** NestJS handles it.

You typically write:

```ts
@Injectable()
export class ConfigService {
  // ...
}
```

Then:

```ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly configService: ConfigService
  ) {}
}
```

and:

```ts
@Injectable()
export class ReportService {
  constructor(
    private readonly configService: ConfigService
  ) {}
}
```

NestJS's DI container manages the instance.

Conceptually:

```text
              NestJS DI Container
                      │
               ConfigService
                      │
             one instance
                /       \
               /         \
              ↓           ↓
       PaymentService   ReportService
```

You **do not** need:

```ts
private static instance;
private constructor();
getInstance();
```

That's manual Singleton implementation and isn't necessary here.

---

# 2. What if we DON'T want Singleton behavior?

This is the important part.

**Don't remove `@Injectable()`.**

Instead, NestJS allows you to configure the provider's **scope**.

For example:

```ts
@Injectable({ scope: Scope.REQUEST })
export class ConfigService {
}
```

Now NestJS creates a separate instance **for each request**.

Conceptually:

```text
Request A
   ↓
ConfigService A

Request B
   ↓
ConfigService B
```

So the lifecycle becomes:

```text
DEFAULT
→ Singleton

REQUEST
→ One instance per request

TRANSIENT
→ New instance when injected
```

The exact behavior of transient providers has some DI-resolution nuances, but the key interview concept is:

> **Transient means NestJS does not maintain one shared singleton instance for the provider.**

---

# 3. So should we remove `@Injectable()`?

**No.**

This:

```ts
@Injectable()
export class ConfigService {}
```

doesn't mean:

> "Make this Singleton manually."

It tells NestJS:

> "This class can participate in dependency injection."

The **scope configuration** determines its lifecycle.

For example:

```ts
@Injectable()
class ConfigService {}
```

→ default singleton scope.

Whereas:

```ts
@Injectable({ scope: Scope.REQUEST })
class ConfigService {}
```

→ request-scoped.

---

# 4. When would we actually want multiple instances?

Your question here is very important.

For a normal `ConfigService`, you're right:

> **Why would PaymentService and ReportService need different ConfigService instances?**

Usually they **shouldn't**.

Configuration is typically application-level information:

```text
DB URL
API configuration
environment
feature flags
etc.
```

Having:

```text
PaymentService → Config A

ReportService → Config B
```

would usually be unnecessary.

---

## But there are legitimate cases for different instances.

### Example: Request-specific context

Imagine:

```ts
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  userId: string;
  requestId: string;
}
```

Every HTTP request should have its **own context**.

```text
Request 1
   ↓
RequestContext #1
   ├── userId = 101
   └── requestId = abc

Request 2
   ↓
RequestContext #2
   ├── userId = 202
   └── requestId = xyz
```

You absolutely **don't** want these to be one shared Singleton:

```text
❌ One RequestContext

Request 1 → userId = 101
Request 2 → userId = 202
               ↓
       overwrites same object
```

That could cause data leaking between requests.

So request-scoped providers are useful when the provider contains **request-specific state**.

---

# 5. Another example — Transient provider

Suppose you have a small object that represents some operation-specific behavior/state.

You may want:

```text
Service A → Instance A
Service B → Instance B
```

rather than:

```text
Service A ─┐
           ├── Same instance
Service B ─┘
```

That's where `Scope.TRANSIENT` can be appropriate.

But don't choose transient simply because:

> "I don't want Singleton."

There should be a lifecycle requirement.

---

# 6. Very important rule for your notes

Think about **state**.

### Stateless shared service

Usually:

```text
Singleton
```

Example:

```text
ConfigService
Logger
Database connection manager
```

### Request-specific state

Consider:

```text
REQUEST scope
```

Example:

```text
RequestContext
CurrentUserContext
Request-specific metadata
```

### Independent short-lived instances

Consider:

```text
TRANSIENT scope
```

when you specifically need a new provider instance for each injection/resolution scenario.

---

# 7. One correction to your thinking

You said:

> "ConfigService also should have been built in the other part of code, right?"

The important distinction is:

**You generally don't create it yourself.**

Don't do:

```ts
const configService = new ConfigService();
```

Instead, let NestJS resolve it:

```ts
constructor(
  private readonly configService: ConfigService
) {}
```

That's the whole benefit of DI.

---

# ⭐ Senior-level interview answer

If asked:

**"What is the default provider scope in NestJS?"**

You can say:

> "NestJS providers are singleton-scoped by default. The DI container creates and manages the provider instance, so consumers receive the same instance within the application process. We don't need to implement the Singleton pattern manually. If the provider needs a different lifecycle, NestJS supports request and transient scopes."

And if asked:

**"Why would you use request scope?"**

> "When the provider contains request-specific state and sharing one instance across requests would be incorrect or unsafe—for example, a request context containing the current user or request ID."

### The mental model I want you to remember:

```text
@Injectable()
      ↓
Can participate in DI

Default scope
      ↓
Singleton

Need request-specific state?
      ↓
REQUEST scope

Need independently created instances?
      ↓
TRANSIENT scope
```

So yes, **for your `ConfigService` example, PaymentService and ReportService would normally receive the same ConfigService instance.** That's exactly the behavior you were asking about.

### Q5

> **When we define a custom provider scope such as `REQUEST` or `TRANSIENT` in NestJS, how do we consume that provider from another service? Does the injection syntax change, and how does NestJS decide which instance to provide?**

The important answer is: **the injection syntax normally does not change.** NestJS's DI container handles the scope for you.

---

## 1. Default Singleton scope

Suppose:

```ts
@Injectable()
export class ConfigService {
  getEnvironment() {
    return 'production';
  }
}
```

And:

```ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly configService: ConfigService,
  ) {}

  pay() {
    return this.configService.getEnvironment();
  }
}
```

You don't do:

```ts
ConfigService.getInstance()
```

NestJS gives `PaymentService` the appropriate instance.

---

# 2. Request scope

Now let's create a request-specific service:

```ts
@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  constructor() {
    console.log('RequestContext created');
  }

  userId?: string;
  requestId?: string;
}
```

Notice:

```ts
scope: Scope.REQUEST
```

Now consume it exactly the same way:

```ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly requestContext: RequestContextService,
  ) {}

  pay() {
    console.log(this.requestContext.userId);
  }
}
```

**No special syntax is required in `PaymentService`.**

---

# 3. What actually happens?

Imagine two HTTP requests.

### Request 1

```text
Request #1
    ↓
NestJS DI Container
    ↓
RequestContextService #1
    ↓
PaymentService
```

And:

```ts
requestContext.userId = 101;
```

### Request 2

NestJS creates another instance:

```text
Request #2
    ↓
NestJS DI Container
    ↓
RequestContextService #2
    ↓
PaymentService
```

And:

```ts
requestContext.userId = 202;
```

So:

```text
Request 1 → Context #1 → userId 101
Request 2 → Context #2 → userId 202
```

They don't share the same `RequestContextService` instance.

---

# 4. Does `PaymentService` also become request-scoped?

This is an important NestJS detail.

If a **singleton service depends on a request-scoped provider**, NestJS has to resolve that dependency in the appropriate request context. This can cause the dependency chain to become request-scoped as well.

Conceptually:

```text
Request
   ↓
PaymentService
   ↓
RequestContextService
```

Because `RequestContextService` is request-specific, NestJS cannot simply keep one universal `PaymentService` instance that contains one fixed request context.

So when designing request-scoped providers, you should understand their effect on the dependency graph.

For interview purposes, remember:

> **Request scope can propagate through the dependency chain.**

---

# 5. Transient scope

Now:

```ts
@Injectable({ scope: Scope.TRANSIENT })
export class PaymentLogger {
  log(message: string) {
    console.log(message);
  }
}
```

And:

```ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly logger: PaymentLogger,
  ) {}
}
```

Again, the syntax is the same:

```ts
constructor(
  private readonly logger: PaymentLogger,
) {}
```

NestJS sees:

```text
PaymentLogger
     ↓
TRANSIENT
```

and manages its lifecycle accordingly.

You don't manually do:

```ts
new PaymentLogger()
```

or:

```ts
PaymentLogger.getInstance()
```

---

# 6. So where is the scope actually used?

At the **provider definition**:

```ts
@Injectable()
```

means default:

```text
SINGLETON
```

While:

```ts
@Injectable({
  scope: Scope.REQUEST,
})
```

means:

```text
REQUEST
```

And:

```ts
@Injectable({
  scope: Scope.TRANSIENT,
})
```

means:

```text
TRANSIENT
```

The consumer remains:

```ts
constructor(private service: SomeService) {}
```

The **DI container determines which instance/lifecycle to provide.**

---

# 7. Very important mental model

Don't think:

> "How do I get the request-scoped instance?"

Think:

> **"I declare the dependency; NestJS resolves the correct instance according to its scope."**

That's the whole purpose of dependency injection.

```text
Provider definition
       ↓
     Scope
       ↓
NestJS DI Container
       ↓
Correct instance
       ↓
Consumer
```

---

## ⭐ Interview-ready answer

If the interviewer asks:

**"How do you consume a request-scoped provider in NestJS?"**

You can say:

> "The injection syntax doesn't normally change. I define the provider with `@Injectable({ scope: Scope.REQUEST })` and inject it through the constructor like any other provider. NestJS's DI container manages the lifecycle and provides the appropriate instance for the current request. The same applies to transient providers; the consumer doesn't manually create or retrieve the instance."

And the key distinction:

> **Scope is a provider lifecycle concern; the consumer generally doesn't need to know how the instance is created.**

This is actually a very good connection to what we've already learned about **Dependency Inversion + Dependency Injection**.

### Q6 

> **If a provider is `REQUEST` scoped and multiple requests are using the same consumer service, how does NestJS ensure that each request receives the correct request-scoped instance? For example, if Request 1 has `userId = 101` and Request 2 has `userId = 202`, what does `PaymentService` receive? Does it receive one fixed instance, or does NestJS resolve the correct instance based on the current request?**

### Answer: It gets the instance belonging to the current request.

There is **no fixed "first instance" or "second instance" assigned to `PaymentService`**.

Think of NestJS as maintaining a **request context**.

For example:

```ts
@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  userId: number;
}
```

And:

```ts
@Injectable()
export class PaymentService {
  constructor(
    private readonly requestContext: RequestContextService,
  ) {}

  pay() {
    console.log(this.requestContext.userId);
  }
}
```

Now suppose two requests arrive.

### Request 1

```text
Request #1
userId = 101
     ↓
NestJS request context #1
     ↓
RequestContextService #1
userId = 101
     ↓
PaymentService for request #1
     ↓
requestContext.userId
     ↓
101
```

### Request 2

```text
Request #2
userId = 202
     ↓
NestJS request context #2
     ↓
RequestContextService #2
userId = 202
     ↓
PaymentService for request #2
     ↓
requestContext.userId
     ↓
202
```

So:

```text
Request 1 → Context #1 → 101
Request 2 → Context #2 → 202
Request 3 → Context #3 → 303
```

There is **no sharing of the request context between requests**.

---

## The important question: "What does PaymentService get?"

It depends on the **current request context**.

Conceptually:

```text
                 NestJS DI Container
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
          Request #1            Request #2
              │                     │
              ↓                     ↓
         Context #1             Context #2
          userId=101             userId=202
              │                     │
              ↓                     ↓
        PaymentService         PaymentService
          (request 1)             (request 2)
```

So when `PaymentService` executes as part of Request 1:

```ts
this.requestContext.userId
```

→ `101`

When it executes as part of Request 2:

```ts
this.requestContext.userId
```

→ `202`

---

# One important correction to our previous discussion

I want to make one thing **more precise** than I explained earlier.

If `PaymentService` is truly a **singleton** while directly depending on a `REQUEST`-scoped provider, you cannot think of it as simply:

```text
one permanent PaymentService
        ↓
one RequestContext
```

That wouldn't work because the context changes per request.

NestJS handles this through its **DI context/request-scoped resolution**, and request-scoped dependencies can cause the relevant dependency chain to be resolved per request.

So for your mental model, think:

> **The request context determines which instance of the request-scoped dependency is resolved for that execution.**

You don't manually select:

```ts
RequestContextService[1]
RequestContextService[2]
```

and you don't pass a request number yourself.

NestJS does that through its DI/request context.

---

## Think of it like a "request-specific dependency graph"

For Request 1:

```text
Request 1
 ↓
PaymentService
 ↓
RequestContext #1
```

For Request 2:

```text
Request 2
 ↓
PaymentService
 ↓
RequestContext #2
```

The **same application code** is being executed, but the DI container resolves the dependency according to the current request.

---

### ⭐ Interview-ready answer

> **"With a request-scoped provider, NestJS creates or resolves a separate instance for each request context. If PaymentService depends on that provider, the dependency is resolved against the current request, so Request 1 gets its RequestContext instance and Request 2 gets a different one. The consumer doesn't choose the instance manually; NestJS's DI container manages that resolution."**

And the simplest mental model:

> **Singleton → same instance across requests.**
> **Request scope → one instance per request context.**
> **Transient → independently resolved instances according to transient DI rules.**

So if you have 100 concurrent requests, you should **not** think "PaymentService gets instance 1, then 2, then 3 sequentially." Each request has its own DI context, and the appropriate request-scoped instance is resolved for that request.
