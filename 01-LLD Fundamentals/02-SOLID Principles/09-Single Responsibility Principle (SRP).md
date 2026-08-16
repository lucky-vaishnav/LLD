## 🆕 LLD Topic #9: SOLID — Single Responsibility Principle (SRP)

We've now covered enough OOP fundamentals to start **SOLID**. We'll take **one SOLID principle at a time**, because understanding *why* each principle exists is much more important than memorizing its definition.

---

# 1. What is SRP?

The formal definition is:

> **A class should have only one reason to change.**

This is often simplified as:

> **A class should have one responsibility.**

The first definition is actually more useful for interviews.

---

## 2. What does "one reason to change" mean?

Consider:

```ts
class InvoiceService {
  createInvoice() {
    // create invoice
  }

  calculateTax() {
    // calculate tax
  }

  saveToDatabase() {
    // save invoice
  }

  sendEmail() {
    // send invoice email
  }
}
```

At first glance, you might think:

> "It's all related to invoices, so this is fine."

But there are actually **multiple reasons this class could change**:

```text
Invoice business rules change
        ↓
InvoiceService changes

Tax rules change
        ↓
InvoiceService changes

Database changes
        ↓
InvoiceService changes

Email requirements change
        ↓
InvoiceService changes
```

That's a violation of SRP.

---

# 3. Better design

Separate the responsibilities:

```text
InvoiceService
    ↓
Invoice creation/business rules

TaxService
    ↓
Tax calculation

InvoiceRepository
    ↓
Database persistence

NotificationService
    ↓
Email notification
```

For example:

```ts
class InvoiceService {
  constructor(
    private taxService: TaxService,
    private repository: InvoiceRepository,
    private notificationService: NotificationService
  ) {}

  createInvoice(data: InvoiceData) {
    const tax = this.taxService.calculate(data);

    const invoice = {
      ...data,
      tax
    };

    this.repository.save(invoice);
    this.notificationService.send(invoice);

    return invoice;
  }
}
```

Now `InvoiceService` focuses on the **invoice workflow/business responsibility**.

It delegates:

```text
Tax calculation       → TaxService
Persistence            → InvoiceRepository
Notification           → NotificationService
```

This is much more cohesive.

---

# 4. Important: SRP does NOT mean "one method per class"

This is a common misunderstanding.

Someone might say:

> "If SRP means one responsibility, every class should have only one method."

❌ No.

For example:

```ts
class User {
  create() {}
  update() {}
  delete() {}
  changePassword() {}
  getProfile() {}
}
```

These methods can all belong to the same **user-related responsibility**, depending on the design.

SRP is about **responsibility**, not the number of methods.

---

# 5. Another important point: SRP doesn't mean "make tiny classes everywhere"

This is also important for senior-level LLD.

You shouldn't turn:

```text
OrderService
```

into:

```text
OrderCreateService
OrderValidationService
OrderStatusService
OrderDataService
OrderHelperService
OrderFieldService
...
```

just to say:

> "Now everything follows SRP."

That can create unnecessary complexity.

The goal is **meaningful separation of responsibilities**, not maximum number of classes.

---

# 6. A practical backend example

Imagine:

```ts
class UserService {
  createUser() {}
  validateEmail() {}
  hashPassword() {}
  saveUser() {}
  sendWelcomeEmail() {}
}
```

There are several responsibilities:

```text
UserService
 ├── User business logic
 ├── Email validation
 ├── Password hashing
 ├── Database persistence
 └── Email notification
```

A better design could be:

```text
UserService
    ↓
User-related workflow

PasswordService
    ↓
Password hashing

UserRepository
    ↓
Persistence

EmailService
    ↓
Email sending
```

Then:

```ts
class UserService {
  constructor(
    private passwordService: PasswordService,
    private repository: UserRepository,
    private emailService: EmailService
  ) {}

  createUser(data: CreateUserRequest) {
    const passwordHash =
      this.passwordService.hash(data.password);

    const user = this.repository.create({
      ...data,
      passwordHash
    });

    this.emailService.sendWelcomeEmail(user);

    return user;
  }
}
```

Notice how this connects directly with topics we've already covered:

**SRP + Composition + Abstraction + DI + Low Coupling**

---

# 7. "Reason to change" is the key

Let's say:

```text
Requirement A:
Password hashing algorithm changes.
```

With poor design:

```text
UserService changes
```

With better design:

```text
PasswordService changes
```

Now:

```text
Requirement B:
Database changes from MongoDB → PostgreSQL.
```

Better design:

```text
UserRepository implementation changes
```

`UserService` doesn't need to know.

That's the real benefit of SRP.

---

# 8. SRP and cohesion

Remember yesterday's topic:

> **High cohesion = related responsibilities stay together.**

SRP helps us achieve that.

Bad:

```text
UserService
 ├── user logic
 ├── payment
 ├── email
 ├── PDF generation
 └── image processing
```

Low cohesion.

Better:

```text
UserService
PaymentService
EmailService
PdfService
ImageService
```

Each has a clearer purpose.

Higher cohesion.

---

# 9. SRP and coupling

There's an interesting relationship here.

If one class handles everything:

```text
UserService
    ↓
Database
Email
Payment
PDF
Storage
```

it tends to become highly coupled.

Separating responsibilities and depending on abstractions can reduce that coupling.

So our earlier principle comes back:

```text
SOLID
  ↓
Better responsibility boundaries
  ↓
High cohesion
  +
Low coupling
```

---

# 10. Senior-level interview answer

If asked:

> **"What is the Single Responsibility Principle?"**

A strong answer:

> "SRP states that a class should have one reason to change. In practice, I try to keep a class focused on one cohesive responsibility and delegate unrelated concerns such as persistence, notification, or external integrations to separate components. This makes changes more localized and reduces coupling."

That's better than:

> "A class should do only one thing."

Because "one thing" can be vague.

---

# 11. A subtle point: SRP is contextual

This is important for your experience level.

Consider:

```ts
class OrderService {
  createOrder() {}
  cancelOrder() {}
  updateOrder() {}
  getOrder() {}
}
```

Is this automatically an SRP violation?

**No.**

All of these can reasonably belong to the responsibility:

> **Managing order lifecycle.**

But:

```ts
class OrderService {
  createOrder() {}
  generatePdf() {}
  sendEmail() {}
  uploadImageToS3() {}
}
```

Now the responsibilities are clearly crossing boundaries.

So SRP requires **judgment**, not blindly splitting methods.

---

# 🎯 Today's key takeaway

Write this down:

> **SRP = A class should have one reason to change.**

And remember:

```text
❌ SRP does NOT mean:
one class = one method

❌ SRP does NOT mean:
create as many classes as possible

✅ SRP means:
one cohesive responsibility
with a clear reason for change
```

### Quick mental test

When you see a class, ask:

> **"If the requirements change in two completely different areas, would this same class need to change for both reasons?"**

If yes, that's a strong signal that the class may have **multiple responsibilities**.

---

### 📚 Progress

We're now officially into **SOLID**:

1. Class & Object
2. Encapsulation
3. Abstraction
4. Inheritance vs Composition
5. Polymorphism
6. Association / Aggregation / Composition
7. Dependency Injection
8. Coupling & Cohesion
9. **SRP — Single Responsibility Principle** ← Today
