# Testing and Discipline (Tests + Part IV: Craftsmanship)

Tests are what let you change code without fear, and discipline is what keeps the whole framework honest over time. This reference covers how to test cleanly and the professional practices that sustain quality.

## Contents
- [Why tests come first](#why-tests-come-first)
- [Testing disciplines: TDD, TCR, small bundles](#testing-disciplines-tdd-tcr-small-bundles)
- [Clean tests](#clean-tests)
- [F.I.R.S.T](#first)
- [Acceptance testing](#acceptance-testing)
- [Craftsmanship: the disciplines that sustain quality](#craftsmanship-the-disciplines-that-sustain-quality)

---

## Why tests come first

A test suite you trust is the thing that makes everything else in this framework safe. Refactoring, cleaning, changing design — all of it depends on being able to confirm in seconds that you didn't break anything. Without that confirmation, the rational move is to *not* touch working code, and the system rots because nobody dares improve it. So tests aren't a chore tacked on at the end; they're the enabler of clean code. They're also first-class code: treat test code with the same care as production code, because messy tests rot, get disabled, and take your safety net with them.

---

## Testing disciplines: TDD, TCR, small bundles

The Second Edition frames several disciplines for keeping code continuously tested:

- **TDD (Test-Driven Development).** The rhythm: write a small failing test, write just enough code to pass it, then clean up both with the test green — repeat in tight cycles. The discipline keeps code testable by construction (you can't write untestable code if the test came first) and keeps designs decoupled. It's a powerful default, not a religion: use it where it earns its keep, and recognize that some exploratory or throwaway work doesn't need it.
- **TCR (Test && Commit || Revert).** A stricter variant: every time you run the tests, if they pass the code is committed automatically, and if they fail it is reverted automatically. This forces genuinely tiny, always-working steps — you learn to make changes so small that losing one to a revert costs almost nothing. Useful as a training discipline for working in small increments.
- **Small bundles.** Whatever the ceremony, the underlying value is the same: make changes in small, complete, always-green bundles rather than large risky ones. Small steps localize mistakes and keep the system shippable at all times.

These pull in the same direction as the rest of the framework: small, verified increments are what make small, safe refactorings possible.

---

## Clean tests

Tests need the same readability discipline as production code — arguably more, because they're read constantly when diagnosing failures.

- **One concept per test.** A test should verify a single behavior, named for it. When one test asserts five unrelated things, a failure doesn't tell you what broke.
- **Arrange / Act / Assert.** Structure each test in three clear parts: set up the data, perform the operation, check the result. A reader should grasp the scenario at a glance.
- **Build a domain-specific testing language.** As tests accumulate, refactor repeated setup and checking into small, expressive helpers (`makePages(...)`, `assertResponseContains(...)`) — *composed assertions* and *composed results* that hide irrelevant mechanics so the test reads as a clear statement of intent. This testing API evolves from refactoring, not up-front design.
- **The dual standard.** Test code may legitimately do things you'd never do in production — favoring clarity over efficiency, for instance — because it runs in a generous environment, not on the constrained target. Cleanliness in tests means *readable*, not necessarily *optimal*.

---

## F.I.R.S.T

Good unit tests share five properties:

- **Fast** — slow tests get run less, and tests that aren't run don't protect you.
- **Independent** — no test depends on another's state or ordering; each sets up and tears down its own world.
- **Repeatable** — the same result on your machine, a teammate's, and CI, with no flakiness.
- **Self-validating** — a boolean pass/fail, never "eyeball the log to see if it worked."
- **Timely** — written close in time to the code they cover (test-first, when it suits, guarantees this).

---

## Acceptance testing

Unit tests verify the pieces; acceptance tests verify the system does what the business asked, in the business's terms. The discipline: capture requirements as automated, executable tests written in language the stakeholders can read, so "done" is defined objectively and the same artifact serves as specification, documentation, and regression check. The goal is to remove ambiguity about whether a feature is actually complete.

---

## Craftsmanship: the disciplines that sustain quality

Part IV of the book steps back from code to the professional conduct that keeps a codebase clean for years. These won't change a single line you write today, but they're the context in which clean code survives. For a coding task, the directly relevant ones:

- **Do no harm — to function or to structure.** Don't ship defects in behavior, and don't degrade the structure (don't leave the design worse than you found it). Both kinds of harm compound. Structural harm is the insidious one because nobody sees it until change becomes expensive.
- **The Boy Scout Rule.** Leave the code a little cleaner than you found it. Continuous small improvements — a better name here, an extracted function there — keep entropy at bay far more reliably than occasional big "cleanup" projects.
- **Small cycles.** Work in small, integrated steps: short-lived branches (or none — prefer toggles over long-lived feature branches), continuous integration, a build that stays green, and continuous deployment where possible. The longer code sits unintegrated, the more painful the merge and the more bugs hide.
- **Relentless improvement.** Watch the health signals — test coverage (and *mutation testing* to check the tests actually assert something), the creep of complexity — and steadily clean rather than letting rot accumulate to a crisis.
- **Repeatable proof.** Quality means being able to demonstrate, repeatably, that the system behaves correctly — which is what a trustworthy automated test suite provides. This is the modern descendant of the structured-programming insight that we should be able to reason about correctness, not just hope for it.

The remaining Part IV chapters — maintaining productivity (managing viscosity and distractions), working as a team, estimating honestly, respecting fellow programmers, and never ceasing to learn — are about being a professional rather than about transforming code. They're worth knowing as the human backdrop to everything above, but a code-writing or review task rarely needs to invoke them directly.
