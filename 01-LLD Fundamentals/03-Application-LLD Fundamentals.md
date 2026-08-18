Today we'll do **Application #3**. This one will intentionally test a different area than Notification/Payment: **encapsulation, object state, and business rules**.

# 🧠 LLD Fundamentals — Application #3

## Design a Bank Account

### Problem

Design a `BankAccount` that supports:

* Deposit money
* Withdraw money
* Check balance
* Transfer money to another account

### Business rules

1. Balance cannot become negative.
2. Deposit amount must be positive.
3. Withdrawal amount must be positive.
4. Withdrawal cannot exceed the available balance.
5. A transfer should follow the same rules as withdrawal/deposit.
6. The account's balance should not be directly modifiable from outside.
7. We may add more account types later, such as:

   * `SavingsAccount`
   * `CurrentAccount`

Different account types may have different rules in the future.

---

## 🎤 Your design challenge

Don't write code yet. Explain your design in your own words.

### Q1. Encapsulation

Where would you keep the `balance`?

Would you make it:

```text
public
private
protected
```

and why?

---

### Q2. Business rules

Where should this rule live?

> "Withdrawal cannot exceed the available balance."

Would you put it in:

```text
BankAccount
BankAccountService
Controller
some separate validator
```

Why?

---

### Q3. Deposit / Withdraw

Would you expose something like:

```text
account.balance = 5000
```

or:

```text
account.deposit(5000)
account.withdraw(1000)
```

Why?

Think about **encapsulation and maintaining valid object state**.

---

### Q4. Transfer

Where should the transfer logic live?

Would you have:

```text
account.transfer(toAccount, amount)
```

or:

```text
BankService.transfer(from, to, amount)
```

or something else?

There isn't necessarily one universally correct answer here. **Explain your reasoning.**

---

### Q5. Future account types

Suppose later:

```text
SavingsAccount
CurrentAccount
```

have different withdrawal rules.

How would you design for that?

This is where you can think about **inheritance vs composition, polymorphism, and OCP**.

And if you think some pattern we haven't formally learned might help, **feel free to mention it**. You don't need to know the pattern implementation yet.

---

### Q6. Testing

How would you test that:

```text
withdraw(100)
```

doesn't allow an account with:

```text
balance = 50
```

to become:

```text
balance = -50
```

---

### 🎯 What I'm looking for

This exercise isn't about producing the "perfect" architecture.

I'm mainly checking whether you naturally think:

```text
Object owns its state
        ↓
State cannot be arbitrarily modified
        ↓
Business invariants are protected
        ↓
Operations control state changes
```

That's one of the most important practical applications of **encapsulation**.

---
### My Thought -
For this application, three design bank account. So, according to this description, you have said a bank account. So, generally, this bank account have these sections like deposit money, withdraw, check balance, transfer money to other account. This will be the similar option for different type of accounts. So, instead of making an interface for abstraction, we can create a class as well with the basic functionality like deposit, withdrawal, and check balance, and transfer. And also, we can create an interface for this function for the abstraction, and there will be a parent class which will be implementing this interface. And then we should make a subclasses according to the types of bank account if they require they work differently. And differently means that according to bank account, we will use the composition for it, and we can transfer the dependency injection according to the bank account. And those functions will use it. Or maybe a base class will be enough, and trying to send that dependency injection on the function like withdraw or transfer money to other account. So, we will use composition plus inheritance together. And for the testing purpose as well, we will be using a payment transfer service which will be injected into the class, base class, or maybe we can send it via the function call of this class. And this money-related operation all will be handled within this base class or class only. This object should not be modified from the outside of the class. So, we will be using the encapsulation to manage the state of the object. And yeah, and those operations, these all operations will be allowed within this class only by using the methods. And yeah, check balance we can avoid that. Check balance we can directly call, but that is also should be a function. And yeah, those variables should be private. And that is how we can control the state of the object. For transfer, we can make a transfer service as well, which will be expecting the dependency injection for the related account. And we can use it within the class as well. So, those all update and the payment flow will not directly call from the outside of the class. So, that is why the class will be important here, just not just the interface. Because class will decide the basic functionality or the state of the class. And if in the future we want to extend it, we can make a new class. And if those accounts also support the same functionality, we can just use the same class as well.

---
Good answer. 👍 This one shows that you're thinking more deeply about **encapsulation and object ownership**, which was the main goal of this exercise.

There are a few places where I'd refine your thinking, especially around **inheritance vs composition and where transfer should live**.

# 📝 Review — Bank Account

## 1. Your main idea is correct

You identified that we can have a common abstraction for:

```text
BankAccount
 ├── deposit()
 ├── withdraw()
 ├── getBalance()
 └── transfer()
```

And potentially:

```text
             BankAccount
                  ↑
        ┌─────────┴─────────┐
        │                   │
   SavingsAccount      CurrentAccount
```

That's a reasonable starting point.

But we need to decide **what actually varies** before introducing subclasses.

---

# ✅ 2. Balance should absolutely be private

You said:

> "These variables should be private."

Correct.

This is the most important part of today's exercise.

We don't want:

```ts
account.balance = -5000;
```

Instead:

```ts
account.withdraw(500);
```

The object itself controls whether that operation is valid.

Conceptually:

```text
Outside
   │
   │ withdraw(500)
   ↓
BankAccount
   │
   ├── Is amount positive?
   ├── Is balance sufficient?
   └── Update balance
```

This is **encapsulation**.

### Notes

> Keep important domain state private and expose controlled operations for changing it. The object should protect its own invariants.

---

# ✅ 3. Business rules belong close to the state they protect

You asked where:

> "Withdrawal cannot exceed available balance"

should live.

For the basic design, I'd put that rule inside `BankAccount`.

```text
BankAccount
    ↓
balance
    ↓
withdraw()
    ↓
protect balance invariant
```

Why?

Because `BankAccount` **owns the balance**.

If we instead do:

```text
BankService
    ↓
check account.balance
    ↓
withdraw
```

then the account's validity depends on external code.

That weakens encapsulation.

A strong domain-modeling rule to remember:

> **The object that owns the state should generally protect the rules/invariants around that state.**

---

# ⚠️ 4. `transfer()` is the interesting one

You mentioned both:

```text
account.transfer(...)
```

and:

```text
TransferService
```

Both can be valid depending on the model.

But think carefully about the responsibility.

A transfer involves **two accounts**:

```text
Source Account
     ↓
 withdraw
     ↓
   amount
     ↓
 deposit
     ↓
Destination Account
```

So you could model:

```ts
sourceAccount.transfer(destinationAccount, amount);
```

where the `BankAccount` controls its own withdrawal and the destination controls its own deposit.

That gives us:

```text
Source Account
    ↓
withdraw()
    ↓
Destination Account
    ↓
deposit()
```

Alternatively, if transfers have more complex business orchestration—transaction handling, fees, audit records, external payment rails, etc.—a separate:

```text
TransferService
```

can make sense.

### Important distinction

Don't create `TransferService` **just because transfer exists**.

Ask:

> **Is transfer itself a significant business workflow involving multiple objects or external concerns?**

If yes → separate service can be appropriate.

For this simple problem, I'd keep the model simpler.

---

# ⚠️ 5. Composition + inheritance together

You said:

> "We will use composition plus inheritance together."

This **can** be correct, but we shouldn't decide that upfront.

First ask:

> **What differs between SavingsAccount and CurrentAccount?**

Suppose:

```text
SavingsAccount
→ minimum balance required

CurrentAccount
→ overdraft allowed
```

Now there is genuinely different behavior.

Inheritance could work:

```text
             BankAccount
                  ↑
        ┌─────────┴─────────┐
        │                   │
 SavingsAccount       CurrentAccount
```

and each subclass can customize the relevant behavior.

But suppose the only difference is:

```text
Savings = 4% interest
Current = 0% interest
```

Then composition might be cleaner:

```text
BankAccount
    ↓
InterestPolicy
    ↑
 ┌──┴───────┐
Savings   Current
```

So don't think:

> "Different account type → inheritance."

Instead:

> **"Does the variation represent a genuine subtype, or is it a replaceable behavior?"**

That's a much stronger LLD thought process.

---

# ⚠️ 6. You mentioned DI inside `withdraw()` / `transfer()`

You said something like:

> "We can send dependency injection to the function like withdraw or transfer."

I'd avoid thinking of DI that way for this example.

For example, don't do:

```ts
account.withdraw(amount, someDependency);
```

unless that dependency is genuinely required for the operation.

Dependency injection is generally about providing a dependency to an object/component from outside:

```text
BankAccount
    ↓
HAS-A dependency
```

But `BankAccount` itself may not need any dependency here.

The balance and withdrawal rules are **intrinsic to the account**.

So:

```text
BankAccount
 ├── balance
 ├── deposit()
 ├── withdraw()
 └── getBalance()
```

may be perfectly sufficient.

Don't introduce DI just because we're studying DI.

That's an important senior-level lesson:

> **Use a principle when the design needs it, not merely because the principle exists.**

---

# ⭐ 7. Your "class + interface" idea

You said we could have:

```text
BankAccount interface
       ↑
BankAccount base class
       ↑
SavingsAccount
CurrentAccount
```

This can work, but we should ask whether the interface adds value.

For example:

```ts
interface BankAccount {
    deposit();
    withdraw();
    getBalance();
}
```

and:

```ts
abstract class BaseBankAccount implements BankAccount {
    private balance;
    
    deposit() {}
    withdraw() {}
    getBalance() {}
}
```

This could be useful if multiple completely different implementations need to satisfy the same contract.

But if we only have:

```text
BaseBankAccount
    ↓
SavingsAccount
    ↓
CurrentAccount
```

then the extra interface might not be necessary.

### Don't create:

```text
Interface
   ↓
Abstract class
   ↓
Concrete class
```

automatically.

Each layer should have a reason to exist.

---

# 🧠 The most important insight from today's problem

Your answer demonstrates something important:

You are starting to think:

> **"Who owns this state, and who should be allowed to change it?"**

That's exactly the right LLD question.

For `balance`:

```text
❌ Controller
❌ Service
❌ External caller

✅ BankAccount
```

Because:

```text
BankAccount owns balance
        ↓
BankAccount controls balance changes
        ↓
BankAccount protects invariants
```

That's **encapsulation applied to domain modeling**, not just:

> "Make variables private."

---

# 📝 Notes version

### Bank Account — LLD

> A `BankAccount` should encapsulate its state, especially `balance`. External code should not directly modify the balance; state changes should happen through controlled operations such as `deposit()` and `withdraw()`.

```text
private balance
      ↓
deposit()
withdraw()
getBalance()
```

Business rules that protect the account's state should generally live close to the state they protect.

For example:

```text
withdraw(amount)
    ↓
amount > 0
    ↓
amount <= balance
    ↓
update balance
```

`transfer()` can be modeled as an account operation for a simple domain, while a separate `TransferService` may be appropriate when transfer becomes a larger workflow involving multiple concerns.

For future account types, first identify **what actually varies**. Use inheritance when there is a meaningful subtype relationship; use composition when the varying behavior can be modeled as an independent strategy/policy.

> **Don't introduce inheritance, composition, DI, interfaces, or services merely because they are available. Introduce them when the domain/problem gives you a reason.**

That last point is especially important for your senior-level interviews.

---

# 🎯 Fundamentals status

We've now done **3 application exercises**:

1. Notification System → abstraction, polymorphism, DI, SOLID
2. Payment System → ISP, DIP, OCP, DI + introduced the idea of Factory
3. Bank Account → **encapsulation, state ownership, invariants, composition vs inheritance**

```ts
// Bank Account - LLD Application #3

interface BankAccount {
  deposit(amount: number): void;
  withdraw(amount: number): void;
  getBalance(): number;
  transfer(toAccount: BankAccount, amount: number): void;
}

abstract class BaseBankAccount implements BankAccount {
  private balance: number;

  constructor(initialBalance: number = 0) {
    if (initialBalance < 0) {
      throw new Error("Initial balance cannot be negative");
    }

    this.balance = initialBalance;
  }

  deposit(amount: number): void {
    if (amount <= 0) {
      throw new Error("Deposit amount must be positive");
    }

    this.balance += amount;
  }

  withdraw(amount: number): void {
    if (amount <= 0) {
      throw new Error("Withdrawal amount must be positive");
    }

    if (amount > this.balance) {
      throw new Error("Insufficient balance");
    }

    this.balance -= amount;
  }

  getBalance(): number {
    return this.balance;
  }

  transfer(toAccount: BankAccount, amount: number): void {
    this.withdraw(amount);
    toAccount.deposit(amount);
  }
}

class SavingsAccount extends BaseBankAccount {
  withdraw(amount: number): void {
    if (amount > this.getBalance()) {
      throw new Error("Insufficient balance in savings account");
    }

    super.withdraw(amount);
  }
}

class CurrentAccount extends BaseBankAccount {
  private overdraftLimit: number;

  constructor(
    initialBalance: number = 0,
    overdraftLimit: number = 1000
  ) {
    super(initialBalance);
    this.overdraftLimit = overdraftLimit;
  }

  withdraw(amount: number): void {
    if (amount <= 0) {
      throw new Error("Withdrawal amount must be positive");
    }

    if (amount > this.getBalance() + this.overdraftLimit) {
      throw new Error("Overdraft limit exceeded");
    }

    // Example only:
    // Base implementation prevents negative balance,
    // so a real design would need protected state/change method
    // or a withdrawal policy to support overdraft properly.
  }
}


// Example usage

const savingsAccount = new SavingsAccount(5000);
const currentAccount = new CurrentAccount(1000, 2000);

savingsAccount.deposit(1000);

savingsAccount.withdraw(500);

console.log(savingsAccount.getBalance());

savingsAccount.transfer(currentAccount, 1000);

console.log(savingsAccount.getBalance());
console.log(currentAccount.getBalance());
```
