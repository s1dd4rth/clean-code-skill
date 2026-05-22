# Code Smells Checklist

A scannable catalogue of warning signs, organized by category. A "smell" is not always a defect — it's a place that *might* repay a closer look. Use this as a diagnostic pass during reviews and to decide what to refactor first. The taxonomy is inspired by the heuristics chapter of *Clean Code*; the descriptions and examples are restated here in plain terms.

**How to use it in a review:** skim the categories, note what's present, then weight by impact. A misleading name or a swallowed exception is a real problem; an inconsistent blank line is a nit. Lead your review with the things that will cause bugs or slow future change.

## Naming smells
- **Vague or non-descriptive name** — `data`, `tmp`, `obj`, `do()`, `handle()`. The name doesn't say what the thing is or does. → Rename to reveal intent.
- **Disinformative name** — the name implies something false (`accountList` that's a map; `isReady` that has side effects). → Make the name match reality.
- **Meaningless distinction** — `a1`/`a2`, or `Customer`/`CustomerData`/`CustomerInfo` where no real difference exists. → Collapse or rename so differences are meaningful.
- **Encoded or cryptic name** — type/scope prefixes (`strName`, `m_count`), unpronounceable abbreviations (`genymdhms`). → Use plain, pronounceable names; let the type system carry type info.
- **Inconsistent vocabulary** — `fetch`/`get`/`retrieve` mixed for the same operation, or one word reused for two ideas. → One word per concept; one concept per word.
- **Magic number or string** — a bare `7`, `0.85`, or `"ADMIN"` embedded in logic. → Name it as a constant so it's searchable and self-explaining.

## Function smells
- **Too long / does several things** — you can't summarize it without "and"/"then". → Split along abstraction boundaries into nameable steps.
- **Mixed abstraction levels** — high-level policy interleaved with byte-twiddling. → Separate altitudes; let the high-level function call down.
- **Too many parameters** — three is a smell, more usually means a missing object or too much responsibility. → Group related params into a value object, or split the function.
- **Flag argument** — a boolean (or enum) that switches behavior, e.g. `render(true)`. → Split into two clearly named functions.
- **Hidden side effect** — the function does something its name doesn't advertise (a "get" that mutates, a "check" that initializes). → Make the effect explicit, or separate command from query.
- **Output argument** — a parameter mutated to return a result, instead of a return value. → Return the result; prefer immutability.
- **Dead/unreachable code** — branches that can never run, functions never called. → Delete it; version control remembers.
- **Duplicated logic** — the same rule or structure in several places. → Extract to one named home (but confirm it's *true* duplication, not lookalikes that change for different reasons).

## Comment smells
- **Redundant comment** — restates what the code already says. → Delete.
- **Stale / misleading comment** — no longer matches the code. → Fix or delete; a wrong comment is worse than none.
- **Commented-out code** — left "just in case." → Delete; retrieve from history if ever needed.
- **Comment compensating for bad code** — a paragraph explaining a tangle. → Often better to clarify the code (rename, extract) than to annotate the mess.
- **Missing the *why*** — code makes a non-obvious choice with no explanation. → Add a short rationale comment (this is a comment worth *adding*).

## Design and general smells
- **Misplaced responsibility** — logic lives where it's convenient, not where it conceptually belongs (a method that uses another object's data more than its own — *feature envy*). → Move it to the class that owns the data.
- **God class / module** — one class knows and does everything. → Split by responsibility and cohesion.
- **Inappropriate coupling** — a module reaches into another's internals; train-wreck getter chains (`a.getB().getC()`). → Apply "tell, don't ask"; expose what callers actually need.
- **Leaky abstraction** — implementation details (SQL, a vendor type, a file format) bleed through an interface meant to hide them. → Keep the abstraction honest; wrap the detail.
- **Speculative generality** — interfaces, hooks, or config for needs that don't exist yet. → Remove it; add flexibility when the need is real.
- **Primitive obsession** — modeling a concept (money, an email, a date range) as a bare string or int everywhere. → Introduce a small type that centralizes the rules.
- **Long if/elif or switch on type** that grows whenever a new case appears, scattered across the codebase. → Consider polymorphism or a strategy/table so new cases are additive.
- **Inconsistent convention** — one corner does it differently for no reason. → Follow the prevailing pattern; surprises cost reader time.
- **Boolean returned, then re-checked** vs. an enum/state that would read clearer. → Model the state explicitly when there are more than two meaningful cases.

## Architecture and boundary smells
- **Dependency pointing the wrong way** \u2014 a business rule imports a database type, a web framework, or a UI detail. \u2192 Invert it: depend on an interface the core defines and the detail implements, so source dependencies point inward toward stable policy.
- **Third-party type leaking through the core** \u2014 a vendor/library class appears in every layer. \u2192 Wrap the library behind a thin adapter; convert to your own types at the boundary.
- **Dependency cycle between components/modules** \u2014 A depends on B depends on A. \u2192 Break it (dependency inversion, or extract a shared component); cycles destroy independent build/test/release.
- **Policy convolved with detail** \u2014 SQL fused with HTML, business rules tangled with report formatting. \u2192 Separate the high-level intent from the low-level mechanism behind an abstraction.
- **Volatile thing many things depend on** \u2014 something that changes often sits at the center of the dependency graph. \u2192 Depend in the direction of stability; push the volatile thing to the leaves.

## Concurrency smells
- **Business logic tangled with thread management.** \u2192 Isolate concurrency as its own responsibility so each can be tested alone.
- **Widely shared mutable state.** \u2192 Shrink the shared surface; prefer copies, immutability, and small protected critical sections.
- **Hand-rolled locking where a proven construct exists.** \u2192 Use the language's concurrent collections / higher-level models (channels, actors, async) instead of bespoke locks.
- **Intermittent failure dismissed as a "fluke."** \u2192 Treat it as a real race; reproduce under load and varied configurations.

## Error-handling smells
- **Error codes instead of exceptions** braiding the happy path with checks. → Throw; keep the main logic linear.
- **Swallowed exception** — empty `catch`/`except` that hides failure. → Handle it, rethrow with context, or at minimum log meaningfully.
- **Returning or passing null** that forces defensive checks everywhere. → Return empty collections, optionals, or special-case objects.
- **Catch-all that loses information** — `except Exception` that masks the real cause. → Catch what you can handle; preserve context for the rest.
- **Context-free error** — "Something went wrong." → Include what failed and enough detail to locate it.

## Test smells
- **No tests around code you're about to change.** → Add characterization tests before refactoring risky code.
- **Test asserting many concepts at once** — a failure doesn't localize the cause. → One concept per test, clearly named.
- **Fragile / flaky test** — depends on timing, ordering, or shared state. → Make tests independent and repeatable (the I and R of F.I.R.S.T).
- **Slow test suite** that discourages running it. → Keep the common path fast; isolate slow integration tests.
- **Test that needs manual inspection** to know if it passed. → Make it self-validating: a clear pass/fail assertion.
- **Untested edge cases / error paths** while only the happy path is covered. → Add tests for boundaries and failure modes, where bugs hide.
