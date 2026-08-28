# Topic 13 — Composite Pattern

This is a useful pattern to understand because it solves a very specific problem:

> **How do we treat individual objects and groups of objects uniformly?**

---

## 1. The problem

Imagine a parking/admin system with a hierarchy:

```text
Parking Facility
├── Floor 1
│   ├── Slot 1
│   ├── Slot 2
│   └── Slot 3
│
├── Floor 2
│   ├── Slot 1
│   └── Slot 2
```

Or think about a file system:

```text
Folder
├── File
├── File
└── Folder
    ├── File
    └── File
```

A **File** is an individual object.

A **Folder** is a group containing other objects.

But we may want to perform the same operation on both:

```ts
item.getSize();
```

For a file:

```text
File → its own size
```

For a folder:

```text
Folder → total size of everything inside it
```

The caller shouldn't need to care whether it's dealing with a File or Folder.

---

# 2. Composite Pattern

We create a common interface:

```ts
interface FileSystemItem {
  getSize(): number;
}
```

Leaf:

```ts
class File implements FileSystemItem {
  constructor(
    private size: number
  ) {}

  getSize(): number {
    return this.size;
  }
}
```

Composite:

```ts
class Folder implements FileSystemItem {
  private items: FileSystemItem[] = [];

  add(item: FileSystemItem) {
    this.items.push(item);
  }

  getSize(): number {
    return this.items.reduce(
      (total, item) => total + item.getSize(),
      0
    );
  }
}
```

Now:

```ts
const file1 = new File(100);
const file2 = new File(200);

const folder = new Folder();

folder.add(file1);
folder.add(file2);
```

We can simply do:

```ts
folder.getSize();
```

and get:

```text
300
```

---

# 3. The important part

Notice this:

```ts
interface FileSystemItem {
  getSize(): number;
}
```

Both:

```text
File
Folder
```

implement the same interface.

So:

```ts
const items: FileSystemItem[] = [
  file1,
  folder
];
```

The caller doesn't need to know:

```text
Is this a File?
Is this a Folder?
```

It simply calls:

```ts
item.getSize();
```

That's the key idea.

---

# 4. Leaf vs Composite

This terminology is important for interviews.

### Leaf

An individual object that doesn't contain other objects of the same abstraction.

```text
File
Slot
MenuItem
```

### Composite

An object that can contain other objects implementing the same abstraction.

```text
Folder
ParkingFloor
Menu
```

So:

```text
Component
   │
   ├── Leaf
   │
   └── Composite
```

---

# 5. Another example — Restaurant Menu

This one is closer to the LLD problems we've already done.

Suppose:

```text
Restaurant Menu
├── Burger
├── Pizza
└── Drinks Menu
    ├── Coke
    ├── Coffee
    └── Juice
```

We could have:

```ts
interface MenuComponent {
  getPrice(): number;
}
```

Menu item:

```ts
class MenuItem implements MenuComponent {
  constructor(
    private price: number
  ) {}

  getPrice(): number {
    return this.price;
  }
}
```

Menu:

```ts
class Menu implements MenuComponent {
  private items: MenuComponent[] = [];

  add(item: MenuComponent) {
    this.items.push(item);
  }

  getPrice(): number {
    return this.items.reduce(
      (total, item) => total + item.getPrice(),
      0
    );
  }
}
```

Now:

```text
MenuItem → its own price

Menu → sum of its children
```

Yet both expose:

```ts
getPrice()
```

That's Composite.

---

# 6. Why is this useful?

Without Composite, the caller might need:

```ts
if (item instanceof MenuItem) {
   // calculate item price
}

if (item instanceof Menu) {
   // calculate menu price
}
```

With Composite:

```ts
item.getPrice();
```

That's the major benefit:

> **The caller can treat individual objects and compositions of objects uniformly.**

---

# 7. Composite vs Tree Data Structure

Don't confuse these.

A tree is a **data structure**.

Composite is a **design pattern**.

Composite often naturally produces a tree-like structure:

```text
        Menu
       /    \
    Item    Menu
           /   \
        Item   Item
```

But the pattern is about the **common abstraction and uniform treatment**, not merely having a tree.

---

# 8. Composite vs Strategy

Another distinction:

### Strategy

```text
PaymentService
      ↓
PaymentStrategy
   ├── Card
   ├── UPI
   └── Wallet
```

Choose one implementation.

### Composite

```text
Menu
 ├── MenuItem
 ├── MenuItem
 └── SubMenu
      ├── MenuItem
      └── MenuItem
```

Treat individual objects and groups uniformly.

---

# 9. When NOT to use Composite

Don't use it simply because you have multiple objects.

You should have a meaningful **part-whole hierarchy**.

For example:

```text
Order
├── Item A
├── Item B
```

doesn't automatically mean Composite.

Ask:

> **Should both the individual item and the group expose the same operation through the same abstraction?**

If yes, Composite might be appropriate.

---

# 🧠 Your turn

Let's use the **Restaurant Menu** example.

### Q1.

What would be the common interface/component?

### Q2.

What would be the Leaf?

### Q3.

What would be the Composite?

### Q4.

Why can both `MenuItem` and `Menu` implement the same interface?

### Q5.

What is the main benefit of Composite for the caller?

### Q6.

What is the difference between Composite and Strategy?

### Q7.

Would you use Composite just because you have a parent-child relationship? Why or why not?

Answer these in your own words, and I'll review them and turn the important parts into your interview notes.

---
### My Thoughts-
So for restaurant menu, the common interface should be menu itself, and the abstraction will be get price, and the leaf will be like the item in the menu, item in the menu, and composite will be the menu, and within that menu there will be menu items. And benefit of using composite is that if we know that the objects share the same, or need to share, or need to expose the same functionality, then we can use the composition. Not just because there is a parent-child relationship, we should not use composition. We should use composition when we know that these objects are sharing the same responsibility, or need to expose the same operation, and we can use the composition. And composition and the strategy are completely different patterns. Strategy is about which behavior to choose. And for composition, as we discussed, it is about mainly how the objects are exposed, and if we think that object and group of objects need to be exposed, or can be exposed in the same way, then we can use composition to avoid the checking that if this is a parent component or a child component instance.

---
Yes — **the core understanding is correct**, especially your point that a parent-child relationship alone does **not** justify Composite.

There are just a couple of terminology corrections.

### Q1. Common interface — ⚠️ Small correction

You said:

> "The common interface should be menu itself."

Not exactly. The common abstraction should be something like:

```ts
interface MenuComponent {
  getPrice(): number;
}
```

Then both implement it:

```text
MenuComponent
   ├── MenuItem   ← Leaf
   └── Menu       ← Composite
```

So **`Menu` is the Composite**, not the common interface.

---

### Q2. Leaf — ✅

Correct:

> `MenuItem` is the Leaf.

It represents an individual object and doesn't contain other `MenuComponent` objects.

---

### Q3. Composite — ✅

Correct:

> `Menu` is the Composite because it contains `MenuItem`s and potentially other `Menu`s.

For example:

```text
RestaurantMenu
├── Burger       ← MenuItem
├── Pizza        ← MenuItem
└── Drinks       ← Menu
    ├── Coke     ← MenuItem
    └── Coffee   ← MenuItem
```

---

### Q4/Q5. Why same interface? — ✅

Your reasoning is good.

The important benefit is:

> **The caller can treat an individual object and a group of objects uniformly.**

So the caller doesn't need:

```ts
if (item instanceof MenuItem) ...
else if (item instanceof Menu) ...
```

It can simply do:

```ts
component.getPrice();
```

Polymorphism takes care of the actual implementation.

---

### Q7. Parent-child relationship alone? — ✅ Exactly

This was your strongest point.

Don't think:

> "There is a parent-child relationship → use Composite."

Instead think:

> **"Do the individual object and the collection/group of objects need to expose the same abstraction and be treated uniformly?"**

If yes → Composite is a good candidate.

---

## One terminology correction

You repeatedly said **"composition"** when you meant **"Composite."**

These are different:

```text
Composition
→ OOP technique where one object contains another object.

Composite Pattern
→ Design pattern where Leaf and Composite objects share a common abstraction.
```

For example, our `Menu` contains `MenuItem`s — that's **composition** at the implementation level, while the overall design is the **Composite Pattern**.

---

# Interview-ready notes

### Composite Pattern

> **Allows individual objects and groups of objects to be treated uniformly through a common abstraction.**

```text
             MenuComponent
              /          \
             /            \
        MenuItem          Menu
          Leaf          Composite
                         /     \
                    MenuItem   MenuItem
```

### Use when:

* There is a meaningful part-whole hierarchy.
* Individual and group objects expose the same operations.
* Caller should not need to distinguish between a leaf and a composite.

### Don't use merely because:

> ❌ There is a parent-child relationship.

### Composite vs Strategy

```text
Strategy
→ Which behavior/algorithm should I choose?

Composite
→ Can individual objects and groups be treated uniformly?
```

And one very useful mental shortcut:

> **Composite = "Treat one and many the same way."**

That's the essence of the pattern.
