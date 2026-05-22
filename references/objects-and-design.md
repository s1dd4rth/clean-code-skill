# Objects and Design (Code + Part II: Design)

How units fit together: the object/data distinction, classes, error handling, and the design principles that keep a growing system from tangling — simple design, SOLID, component cohesion and coupling, continuous design, and concurrency.

## Contents
- [Objects vs. data structures](#objects-vs-data-structures)
- [The Law of Demeter](#the-law-of-demeter)
- [Clean classes](#clean-classes)
- [Error handling](#error-handling)
- [Simple design](#simple-design)
- [SOLID](#solid)
- [Component principles](#component-principles)
- [Continuous design: the Four Cs](#continuous-design-the-four-cs)
- [Concurrency](#concurrency)

---

## Objects vs. data structures

A distinction that prevents a common mess:

- An **object** hides its data behind methods and exposes *behavior*. You tell it what to do.
- A **data structure** exposes its data and has little or no behavior. You read its fields and act on them elsewhere (a DTO is the pure form of this).

There's an **antisymmetry** here. With objects, adding a new *type* is cheap (implement the interface) but adding a new *operation* is expensive (touch every type). With data structures plus functions, it's the reverse: new operations are easy, new types ripple. Neither is "better" — pick the style that matches the axis along which you expect the code to grow, and accept the trade-off knowingly. (This is the OO/procedural trade-off.)

Trouble comes from **hybrids** that expose their internals *and* carry significant behavior. They get the disadvantages of both and resist extension in either direction. Decide which one a given type should be.

---

## The Law of Demeter

A method should only talk to its immediate collaborators: its own fields, its parameters, and objects it creates. It should not reach *through* one object to operate on another it got back.

```java
// "train wreck": this knows the whole object graph and breaks when any link changes
String city = order.getCustomer().getAddress().getCity();

// tell, don't ask — let the object expose what the caller actually needs
String city = order.getShippingCity();
```

Chained getters couple the caller to the internal structure of everything in the chain. This is about *objects*; chaining on a fluent builder or a stream/LINQ pipeline (designed for it) is fine — those are data-structure-like by intent.

---

## Clean classes

The instinct that keeps functions small keeps classes focused: a class should be **about one thing**, named by the class.

- **Single Responsibility.** One reason to change. If business rules change for one reason and the database schema for another, those concerns probably don't belong together. Telltale signs of too much: a name with `And`, `Manager`, `Processor`, or `Utils`, or a class you can't summarize without listing unrelated duties.
- **Cohesion.** In a cohesive class, most methods use most of the fields. A cluster of methods touching one subset of fields and another cluster touching a different subset is a class asking to be split in two.
- **Size by responsibility, not lines.** A class with many tiny methods can still do one thing; a class with two methods that each do three unrelated jobs does not.
- **What a class should contain:** the data and the behavior that operate on that data, kept small and organized so a reader can see its purpose at a glance. Public surface first, supporting detail below.

---

## Error handling

Error handling should be visible and honest, but it shouldn't drown the main logic.

- **Use exceptions, not return codes**, so the happy path reads cleanly (see `code-craft.md`).
- **Provide context.** The message and type should say *what* failed and *where* to look. "Error" helps no one; "failed to parse config at line 12: expected integer for `timeout`" does.
- **Define exception types around the caller's needs.** Group them by how they'll be handled, not by where they were thrown. If every catch does the same thing regardless of type, you have too many types; if one catch inspects the message to decide what to do, too few.
- **Don't return or pass null.** Special-case objects, empty collections, or explicit optionals beat scattering null checks.
- **Don't swallow exceptions silently.** An empty `catch {}` hides failures until they surface somewhere far away and harder to diagnose.

---

## Simple design

The best design is the simplest one that supports the required features while leaving the most room to change. Simple does not mean easy — *simple means untangled*, and untangling is hard. The most expensive entanglement is high-level policy convolved with low-level detail (business rules wired into SQL, report format fused with the calculation). The cure is abstraction: amplify the essential, isolate the irrelevant, and point detail at policy rather than the reverse.

In practice, design is "simple" when it satisfies, in priority order, Kent Beck's four rules:

1. **Passes all the tests.** A design that doesn't work isn't simple, it's broken. Tested behavior comes first.
2. **Expresses intent.** The code reveals what it's doing and why — good names, clear structure, no mystery.
3. **No duplication.** A rule should live in exactly one place (DRY) — but confirm it's *true* duplication, not lookalikes that change for different reasons.
4. **Fewest elements.** With the above satisfied, minimize the number of classes, methods, and moving parts. Don't add structure for hypothetical futures.

And **YAGNI** — "You Aren't Gonna Need It": don't build generality for needs that don't yet exist. Speculative flexibility is itself a form of tangle. Add the abstraction when the second real need arrives, not before.

---

## SOLID

Five object-oriented design principles. Apply them when you can see the change they protect against actually coming — not speculatively.

- **S — Single Responsibility:** one reason to change per module.
- **O — Open/Closed:** add new behavior by adding code, not editing code that already works. When a new "type" means another branch in a growing switch scattered across the codebase, this is the principle being violated (often resolved with polymorphism or a strategy).
- **L — Liskov Substitution:** a subtype must be usable anywhere its base type is, without surprising the caller. A subclass that throws on a supported method, or quietly ignores a call, breaks substitution.
- **I — Interface Segregation:** don't force a client to depend on methods it doesn't use. Prefer several small, role-specific interfaces over one fat one.
- **D — Dependency Inversion:** high-level policy shouldn't depend on low-level detail; both depend on an abstraction. In practice: depend on an interface and inject the concrete implementation, so policy doesn't know (or care) whether it's talking to Postgres or an in-memory fake. This is the engine behind simple, untangled design and the Clean Architecture (see `architecture.md`).

---

## Component principles

Above classes sit *components* — independently deployable/releasable units (a package, library, or service). Two sets of principles govern them.

**Cohesion — which classes belong together:**
- **REP (Reuse/Release Equivalence):** the unit of reuse is the unit of release. Things reused together should be released and versioned together so consumers can depend on them safely.
- **CCP (Common Closure):** gather into one component the classes that change for the same reasons at the same times. When a change comes, you'd rather it hit one component than scatter across many. (This is SRP at the component level.)
- **CRP (Common Reuse):** don't force a component's users to depend on things they don't use. Classes that aren't reused together don't belong together.

These three pull against each other (REP and CCP make components larger; CRP makes them smaller); good component boundaries balance the tension and shift over time as the project matures.

**Coupling — how components depend on each other:**
- **ADP (Acyclic Dependencies):** the dependency graph between components must have no cycles. A cycle means the components can no longer be built, tested, or released independently. Break cycles with dependency inversion or a new component.
- **SDP (Stable Dependencies):** depend in the direction of stability. A component that many things depend on should be hard to change; volatile components should sit at the leaves, depending on stable ones, not the reverse.
- **SAP (Stable Abstractions):** a stable component should be abstract (interfaces, policy) so it can be extended without modification; a concrete, detail-heavy component should be unstable (safe to change). Stability and abstractness should track together.

---

## Continuous design: the Four Cs

Design isn't a phase you finish; it's a thing you keep doing as the system changes. As a memorable lens for "is this still clean," the Second Edition offers four considerations that support and sometimes oppose one another:

- **Clarity** — the programmer's intent is clearly stated throughout.
- **Conciseness** — that intent is implemented in a minimal amount of code (without sacrificing clarity).
- **Confirmability** — every behavior can be easily tested, and those tests double as living documentation of the choices made.
- **Cohesion** — each module's elements maximally relate to one another.

These echo the symptoms of *unclean* code: when it's hard to find the code you want, hard to know where to change it, hard to change it without creating defects, and hard to write tests for it, one of the Four Cs is slipping.

---

## Concurrency

Concurrency is decoupling *what* gets done from *when* it gets done, and it's a source of bugs that are intermittent and brutal to reproduce. Defensive principles:

- **Keep concurrency code separate.** Treat thread management as its own responsibility (SRP), isolated from business logic, so each can be reasoned about and tested alone.
- **Limit the scope of shared data.** The less mutable state is shared between threads, the fewer places things can go wrong. Protect critical sections, and keep them small.
- **Prefer copies and immutability.** Sharing copies of data, or immutable values, avoids whole classes of race conditions.
- **Make threads as independent as possible.** A thread that doesn't share state with others can be reasoned about as if it were the only one.
- **Know your library and your execution models** (producer/consumer, readers/writers, dining philosophers). Use proven concurrent collections and constructs rather than hand-rolling locks.
- **Test relentlessly.** Run concurrent code under load, on different configurations, and instrument it to flush out timing bugs; treat intermittent failures as real, never as flukes. (Modern languages increasingly offer higher-level models — async/await, channels, actors, structured concurrency — that reduce raw lock management, but the principles above still apply.)
