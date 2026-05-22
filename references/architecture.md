# Architecture (Part III)

The shape of the whole system: which way dependencies point, where the lines between parts go, and how to keep the system's structure cheap to change. Reach for this when the task is structuring a module, drawing a service or layer boundary, or deciding what should depend on what.

## Contents
- [The two values of software](#the-two-values-of-software)
- [Independence](#independence)
- [Architectural boundaries](#architectural-boundaries)
- [Clean boundaries: third-party code](#clean-boundaries-third-party-code)
- [The Clean Architecture and the Dependency Rule](#the-clean-architecture-and-the-dependency-rule)

---

## The two values of software

Every system delivers two distinct values:

- **Behavior** — what it does right now: the features that make it work today. This is urgent and visible; stakeholders feel it immediately.
- **Structure** — how easy it is to *change* what it does. This is important but invisible, and it's what "architecture" protects.

The trap is that the urgent (behavior) constantly crowds out the important (structure), because broken structure isn't felt until the day a change that should take an hour takes a week. The architect's job is to keep the system **soft** — to preserve the ability to change it — even under pressure to just ship behavior. A program that works but cannot be changed will eventually become worthless; a program that doesn't work yet but is easy to change can be made to work.

**Keeping options open.** Good architecture defers and protects decisions: it lets you delay commitments to details (which database, which framework, which delivery mechanism) and change them later cheaply. The longer you can keep an option open, the more information you'll have when you finally have to decide.

---

## Independence

A good architecture supports the system's **use cases** first and foremost — you should be able to look at the structure and see what the system *does*, not just what frameworks it uses. Beyond that, it keeps several concerns independent so they don't drag on each other:

- **Independent operation** — different use cases that scale or run differently can be separated so they don't interfere.
- **Independent development** — teams can work on different components without stepping on each other, because the components are decoupled along clear seams.
- **Independent deployment** — components can be deployed (or not) on their own schedules, rather than forcing one big-bang release.

You buy this independence by decoupling at the right lines and pointing dependencies the right way (below). Decoupling has a cost, so you balance how much you invest against how much independence the system actually needs — and you can leave the *option* to decouple further open without paying for it all up front.

---

## Architectural boundaries

A boundary is a line across which dependencies are controlled. You draw a boundary between things that change for different reasons or at different rates — business rules vs. the database, core logic vs. the UI, your code vs. a vendor's.

- **Draw boundaries around what's volatile and likely to change**, separating it from what's stable. The expensive things to get wrong are the boundaries between high-level policy and low-level detail.
- **The plug-in pattern** is the model: arrange the system so that details (the database, the GUI, a third-party service) are *plug-ins* to the core business rules. The core defines an interface; the detail implements it. The source-code dependency points from the plug-in toward the core, even though at runtime the core calls out to the plug-in (this inversion is achieved with DIP).
- **Don't draw every possible boundary up front** — full boundaries cost real effort. Some can be left as cheaper "partial" seams (a well-separated set of classes, an interface you haven't split out yet) and hardened into full boundaries only if and when the need materializes. Over-drawing boundaries is its own form of over-engineering.

---

## Clean boundaries: third-party code

Where your code meets a library, framework, or external API, keep the seam clean so changes on the other side don't ripple through everything.

- **Wrap volatile third-party interfaces.** Route calls to a vendor API through a thin adapter you control rather than scattering them everywhere. When the vendor changes its interface (or you swap vendors), you fix one place. The adapter also lets you express the API in *your* domain's terms.
- **Don't let third-party types leak through your core.** If a library's data type appears in every layer, you're coupled to that library everywhere. Convert at the boundary.
- **Write learning tests.** When adopting an unfamiliar library, small tests that exercise *your understanding* of its behavior both teach you the API and warn you when an upgrade changes that behavior — you find out at build time, not in production.

---

## The Clean Architecture and the Dependency Rule

The Clean Architecture (and its relatives — hexagonal/ports-and-adapters, onion) organizes a system as concentric layers, from the most general and stable at the center to the most specific and volatile at the edge:

- **Entities** — enterprise-wide business rules and the core data they operate on. The most stable, least likely to change for any one application.
- **Use cases** — application-specific business rules that orchestrate the entities to accomplish what this system does.
- **Interface adapters** — controllers, presenters, gateways that translate between the use cases and the outside world's formats.
- **Frameworks and drivers** — the database, the web framework, the UI, external services. The most volatile, outermost ring; ideally these are details you could swap.

**The Dependency Rule is the heart of it: source-code dependencies must point only inward, toward higher-level policy.** Inner circles know nothing about outer ones — the business rules don't import the database, the web framework, or the UI. Nothing in an inner circle names anything in an outer circle. When an inner circle needs to invoke something an outer circle provides (e.g., use case needs to save a record), it does so through an interface the *inner* circle defines and the *outer* circle implements — so the dependency still points inward (DIP again).

The payoff is exactly the "two values" goal: because the core depends on nothing volatile, you can change the database, the framework, or the delivery mechanism without touching the business rules — the system stays soft. A reliable review heuristic for any layered system: **trace the dependencies and check they all point toward the stable core. An arrow from a business rule to a framework or a database type is the smell.**

This is the architectural-scale expression of the same idea that runs through the whole framework: keep high-level policy ignorant of low-level detail.
