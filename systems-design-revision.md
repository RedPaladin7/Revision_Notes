# Systems Design Revision

Weekly cheat sheet. Not a textbook. Skim this, then drill the parts that still feel fuzzy.

---

## Object Oriented Programming

Java-centric notes. OOP models a system as objects: each object has state (data) and behavior (methods). A **class** is the blueprint. An **object** is one instance of that blueprint, with its own data and a unique identity in memory.

Write the class once, mint as many objects as you need.

### Access modifiers

They decide who can touch a class, field, or method. That is the mechanism behind encapsulation.

| | Same class | Same package | Subclass, other package | Everywhere |
|---|---|---|---|---|
| `public` | yes | yes | yes | yes |
| `protected` | yes | yes | yes | no |
| default (no keyword) | yes | yes | no | no |
| `private` | yes | no | no | no |

`protected` is not "package only". Unrelated classes in another package cannot use it; a subclass can.

### Constructors

Same name as the class, no return type, runs automatically on `new`. Job: leave the object in a valid state.

- **Default / no-arg** — Java gives you one only if you wrote none. Fields start at `0` / `false` / `null`. You can write your own to set better defaults.
- **Parameterized** — caller passes the starting values.
- **Copy** — `Movie(Movie other)` clones another object. Copying a reference is a shallow copy; nested objects may still be shared.
- **Private** — outside code cannot `new` it. Used for Singleton and utility classes.

If you write any constructor, Java stops generating the no-arg one. `new Foo()` then fails unless you also wrote `Foo()`.

`this(...)` calls another constructor in the same class (must be the first line). `super(...)` calls the parent. Parent constructor always runs before the child. Constructors are not inherited, and cannot be `final`, `static`, or `abstract`. A bare `return;` is allowed to exit early; returning a value is not.

### `this`

Reference to the current object. Use it to unshadow fields (`this.name = name`), chain constructors, return the same object for fluent APIs (`return this`), or pass the current object into another method. Illegal in `static` methods — there is no instance.

### Encapsulation

Keep the data private and expose only methods that are allowed to read or change it. Example: a bank account hides `balance` and only changes it through `deposit` / `withdraw`, which can reject invalid amounts. Callers depend on the method names, not the fields, so you can change internals later without breaking them.

### Inheritance

A subclass **is-a** parent: it reuses and extends the parent's fields and methods (`extends` in Java). Java allows one parent class only.

- **Single** — one parent.
- **Multilevel** — a chain: `Dog extends Mammal extends Animal`.
- **Hierarchical** — many children of one parent: `Dog` and `Cat` both extend `Animal`.
- **Multiple** — one class, many parents. Forbidden for classes because of the **diamond problem**: two parents define the same method, the child does not know which body to use. Do this with **interfaces** instead.
- **Hybrid** — mix of the above, usually one class plus several interfaces.

Reuse and polymorphism are the wins. The cost is coupling: a change in the parent can break every child, and deep trees get hard to follow. If the relationship is really "has-a", prefer composition.

### Polymorphism

Same call, different behavior depending on the object. A person is a father, a husband, and an employee — same person, different behavior in each role.

**Compile-time (static)** — method **overloading**. Same name, different parameter list (count or types). The compiler picks the method.

**Runtime (dynamic)** — method **overriding**. A subclass replaces a parent method with the same name, params, and return type. The **reference** can be the parent type; the **actual object** decides which body runs.

```java
Vehicle v = new Car();
v.start();   // Car.start(), decided at runtime
```

That is dynamic dispatch. It lets you write `for (Vehicle v : list) v.start()` and later add `Bus` without touching the loop.

Overloading vs overriding: overloading is same class, different params, compile time. Overriding is subclass, same signature, runtime. Put `@Override` on overrides so a typo fails at compile time.

### Abstraction

Show **what** an object does, hide **how**. You drive with the steering wheel; you do not operate the engine. In Java this is done with abstract classes and interfaces.

#### Abstract class vs interface

This is the comparison that actually needs space.

An **abstract class** is a partial class. You cannot `new` it. It can mix abstract methods (children must implement) and concrete methods (shared code). It can hold fields, constructors, and any access modifier (`private`, `protected`, …). A class `extends` only one abstract class.

An **interface** is a contract: "you must be able to do these things." A class `implements` as many as it wants. Classic interface methods are abstract. From Java 8 they may also have `default` and `static` methods; from Java 9, `private` methods too. Fields are `public static final` constants only. No constructors, no instance state.

| | Abstract class | Interface |
|---|---|---|
| Purpose | Shared code + state for **related** types | Behavior contract, often for **unrelated** types |
| Instantiation | No | No |
| Methods | Abstract + concrete | Abstract; also default / static / private |
| Fields | Any (instance, static, final or not) | `public static final` only |
| Constructors | Yes | No |
| Inheritance | One parent class | Many interfaces |

**Pick an abstract class** when subclasses are a family and share implementation or mutable state. All animals `eat()` the same way; only `makeSound()` differs.

**Pick an interface** when you only need a capability, or when a class must take on several roles (`Dog implements Animal, Pet`). `Payment` implemented by `CreditCardPayment` and `CashPayment` is an interface: those classes are not a family, they just both know how to `pay()`.

A class can do both: `class Dog extends Animal implements Pet`. An abstract class can implement an interface and fill in some of the methods, leaving the rest to children.

Bad abstraction: putting `fly()` on `Animal` forces `Dog` to stub it out. Split extra abilities into small interfaces (`Flyable`, `Swimmable`) and implement only what applies.

**Default methods** (Java 8) have a body on the interface so you can add a method later without breaking every implementer. Rules that show up in interviews:

- Two interfaces with the same default method → the class **must** override it (can delegate with `InterfaceName.super.method()`).
- A superclass method and an interface default collide → the **class** method wins.
- Default methods have no instance fields of the implementing class; interfaces do not hold object state.

Do not reach for abstraction on a two-class toy. The extra layer is worth it when you expect more types, a common API, or polymorphism.

### Class relationships (UML / LLD)

These names show up in every design question. Get the strength of the link right.

- **Inheritance (`is-a`)** — `Dog` is an `Animal`. Solid line, closed arrow.
- **Realization** — a class implements an interface. The class signs a contract (`CreditCardPayment implements Payment`).
- **Association** — long-term "knows about". `Person` stores a `Car`. Both can exist on their own.
- **Aggregation** — whole **has** parts, but parts survive if the whole dies. A `Team` has `Player`s; disband the team, the players still exist. UML: empty diamond.
- **Composition** — whole **owns** parts. Destroy the whole, parts go with it. A `House` creates its `Room`s; no house, no rooms. UML: filled diamond. Strongest "has-a".
- **Dependency** — temporary use. `Document.print(Printer p)` borrows a printer for one call and does not keep it. UML: dashed arrow.

Stored as a field → association or stronger. Passed only as a method argument → dependency. Whole controls the part's lifetime → composition. Whole groups independent parts → aggregation.

In a library example: `EBook` **inherits** `Book`; `Book` **associates** with `Author`; `Library` **composes** `Book`s; `ReadingClub` **aggregates** `Reader`s; `Reader` **depends** on `Book` while reading; `Book` **realizes** `Readable`.

### Generics and wildcards

Generics parameterize a type so one class or method works for many types, with mistakes caught at compile time. `List<String>` will not accept an `Integer`. Type arguments must be reference types (`Integer`, not `int`). Primitive arrays are allowed because an array is an object.

- Generic method: `static <T> void show(T x)`
- Generic class: `Box<T>`, or several params: `Pair<T, U>`

Without generics you store `Object`, cast on the way out, and blow up at runtime (`ClassCastException`). With generics the compiler refuses the bad `add`.

**Wildcards** (`?`) mean "some unknown type", usually on a method parameter.

- `List<?>` — any list. Read as `Object`. Cannot add (except `null`).
- `List<? extends Number>` — Number or a subclass. **Read** as `Number`. Do not add. Producer Extends.
- `List<? super Integer>` — Integer or a superclass. Safe to **add** `Integer`. Consumer Super.

PECS: producer-extends, consumer-super.

Use `<T>` when the same type must line up in more than one place (two arguments, or argument + return, or you need to add to the collection). Use `?` when you only read and do not care what the exact type is. `List<?>` returning a value gives you `Object` and forces a cast; a generic `<T> T getFirst(List<T>)` preserves the type.

---

## Development Principles

Four rules that show up in LLD interviews. DRY, KISS, and YAGNI are short. SOLID is five rules; that is the one worth slowing down for.

### DRY (Don't Repeat Yourself)

A piece of knowledge or logic should live in **one** place. If you copy-paste a swap, a URL, or a button renderer, a later fix will miss one of the copies.

How: extract a method, a constant (`Config.BASE_URL`), or a small reusable class. Share behavior with inheritance or interfaces when the structure is actually the same.

Trap: do not glue together things that only *look* similar. Forced reuse couples unrelated code and is worse than a little duplication.

### KISS (Keep It Simple, Stupid)

The simplest design that still works. Nested loops to compute a factorial, mystery names like `x` and `y`, or one giant `processOrder` that also computes tax — all of that is extra complexity you do not need.

How: break the problem up, name things for what they are (`basePrice`), keep functions small and single-purpose. Simple is readable, not "fewest lines."

Trap: oversimplifying until edge cases disappear, or refusing a real pattern because it "looks fancy."

### YAGNI (You Aren't Gonna Need It)

Build what the **current** requirement asks. Not PayPal and crypto because "we might need them." Closely related to KISS: KISS is about *how* you write it, YAGNI is about *whether you write it at all*.

Interview version: they ask for debit and credit cards only. Extra payment types waste time, add branches, and signal overengineering.

Trap: YAGNI is not an excuse to skip a design you actually need right now. It is a ban on speculative features.

### SOLID

Five OOP rules. Goal: change one thing without breaking everything else. They overlap with the OOP notes (especially abstraction and "don't put `fly()` on `Animal`").

**S — Single Responsibility.** A class should have one reason to change. A `BreadBaker` that also manages inventory, orders supplies, serves customers, and cleans the shop has five jobs. Split them: `BreadBaker`, `InventoryManager`, `SupplyOrder`, `CustomerService`, `BakeryCleaner`. One change (how you order flour) should not touch the baking class.

**O — Open/Closed.** Open for extension, closed for modification. You should add behavior by writing new code, not by editing a growing `if (type.equals("circle"))` chain. Make `Shape` abstract (or an interface) with `calculateArea()`. `Circle`, `Rectangle`, then later `Triangle` are new classes. Existing shapes stay untouched.

**L — Liskov Substitution.** A subclass must be usable wherever the parent is expected, without surprises. If `Vehicle` has `startEngine()`, a `Bicycle` that throws `UnsupportedOperationException` is not a valid substitute — polymorphism breaks. Put shared behavior on `Vehicle` (`move()`), split `EngineVehicle` vs `NonEngineVehicle`, and only put `startEngine()` on the engine side. Same smell as forcing `Dog` to implement `fly()`.

**I — Interface Segregation.** Do not force a class to implement methods it does not use. A fat `Machine` with `print()`, `scan()`, `fax()` makes `BasicPrinter` stub out scan and fax. Split into small interfaces: `Printer`, `Scanner`, `FaxMachine`. All-in-one implements all three; a basic printer implements `Printer` only. Same idea as `Flyable` / `Swimmable`.

**D — Dependency Inversion.** High-level code should not depend on low-level details. Both should depend on abstractions. `OrderService` should not `new EmailNotifier()` (and a logger, and inventory) inside its constructor — that locks you to email and makes tests hard. Depend on `NotificationService`, `LoggingService`, `InventoryService`. Pass the implementations in (constructor injection). Swapping email for SMS, or faking them in a test, does not require editing `OrderService`.

Quick map:

| Letter | Failure smell | Fix |
|---|---|---|
| S | One class, many jobs | Split classes |
| O | Edit a switch/if every time a type is added | New class behind an interface |
| L | Subclass throws or no-ops a parent method | Narrow the parent; don't promise what you can't do |
| I | Class implements methods it must dummy out | Many small interfaces |
| D | `new ConcreteThing()` inside the business class | Depend on an interface; inject the impl |

S and I both fight "too much in one place" — S for classes, I for interfaces. O and D both keep you from rewriting callers when details change. L is the contract behind polymorphism: if it is typed as the parent, it must actually behave like the parent.
