---
name: clean-code
description: Apply Robert C. Martin's Clean Code framework to write, review, and refactor source code for readability and maintainability. Use this whenever the task involves writing new functions, classes, modules, or services; reviewing or critiquing existing code; refactoring or cleaning up messy code; or improving code quality, readability, naming, structure, design, or architecture — even when the user never says the words "clean code". Trigger on requests like "review my code", "is this good code", "refactor this", "make this more readable", "clean this up", "why is this so hard to maintain", "this function is too long", "help me name this", "is my class doing too much", "how should I structure this module", or any ask to improve naming, function or class design, error handling, comments, test quality, dependencies, or architectural boundaries. Applies across languages (Python, JavaScript/TypeScript, Java, Go, C#, Rust, and others).
---

# Clean Code

Clean code is code optimized for the next human who reads it — usually a teammate, or you in six months. The machine doesn't care about names, structure, or comments; people do, and people are who maintain software. The job of this skill is to help you **write, review, and refactor** code so that its intent is obvious and changing it is cheap.

This skill follows the framework of Robert C. Martin's *Clean Code, Second Edition*, which is organized into four ascending levels:

- **Code** — the line-level craft: names, functions, comments, formatting, objects, classes, error handling.
- **Design** — how units fit together: simple design, SOLID, component cohesion and coupling.
- **Architecture** — the shape of the whole system: dependency direction, boundaries, keeping options open.
- **Craftsmanship** — the disciplines and conduct that keep all of the above honest over time.

The principles are heuristics, not laws. The guiding question behind every one is the same: **will the next reader understand this quickly, and can they change it safely?**

## Read this first: calibration

The fastest way to do harm with these principles is to apply them as absolute rules. A skill that mechanically "cleans" code often makes it worse. The Second Edition itself stages a debate (its appendix, "The Clean Code Debate," with John Ousterhout) in which both sides agree that **over-decomposition is real** and that the first edition gave too little guidance on avoiding it — they disagree mainly on *how far* to go. So treat the famous guidance as directional, not absolute. Watch for these traps:

- **Over-decomposition.** "Functions should be small" does not mean every function becomes three lines. Shredding a coherent function into many one-liners that are each called once usually *hurts* readability — the reader now jumps around to reassemble a story that used to read top to bottom, and entangled fragments are harder to follow than the whole. Extract a function when it earns its name: it does one nameable thing, removes real duplication, or separates a level of abstraction. Not to hit a line count.
- **Comment purism.** "Good code explains itself" is an aspiration, not a ban on comments. Delete comments that merely restate the code; keep comments that explain *why*, warn of consequences, document a public API, or capture a non-obvious decision.
- **Premature abstraction.** Don't add interfaces, layers, configuration, or design patterns for flexibility nobody has asked for (YAGNI). Clean is not the same as maximally abstract. The simplest design that is clear and passes the tests wins.
- **Context sets the bar.** A throwaway script, a hot inner loop, and a widely-used public library are held to different standards. There's even a deliberate "dual standard" between test code and production code. Match your suggestions to the situation.

Rule of thumb: when you propose a change, you should be able to state **why it helps the reader or the maintainer**. If you can't articulate that, don't make the change.

## The three modes

Most requests are one of three jobs. Identify which, then follow that flow.

### 1. Reviewing code

Goal: surface the issues that genuinely matter, prioritized, with concrete fixes — not a nitpick avalanche that buries the important problems.

1. **Read for intent first.** Work out what the code is trying to do before judging it. A review that misunderstands the purpose is worse than none.
2. **Find the issues that matter most.** Scan against `references/code-smells.md`. Weight by impact: a misleading name or a hidden side effect that will cause a real bug matters far more than inconsistent spacing.
3. **Prioritize.** Group findings so the reader knows where to spend attention:
   - **Blocking** — bugs, hidden side effects, broken error handling, misleading names that will trip people up.
   - **Should fix** — structure and design issues that will slow future change (a function doing three things, duplication, leaky abstractions, wrong dependency direction).
   - **Nits** — small readability and consistency points, clearly labeled as optional.
4. **Be concrete and explain why.** Show the specific spot, name the consequence ("this returns null, so every caller has to remember a null check or it crashes"), and offer a fix or short before/after. "Improve naming" is not actionable.
5. **Acknowledge what's already good.** If the code is solid, say so plainly rather than inventing problems.

### 2. Writing code

Apply the principles as you go — lightly, in service of clarity, not as a checklist you announce.

- Start from clear names and small, single-purpose functions.
- Keep each function at one level of abstraction so it reads like a short paragraph, and arrange functions so the file reads top-down (the "stepdown rule").
- Handle errors and edge cases honestly; don't paper over them with silent catches or returned nulls.
- Write the code you'd want to inherit. If logic is subtle, a one-line *why* comment beats a clever name that hides the subtlety.
- Don't over-build. Solve the problem in front of you cleanly; resist speculative generality.

### 3. Refactoring code

Refactoring means improving structure **without changing behavior**. Order of operations matters:

1. **Pin down behavior first.** If there are tests, run or read them so you know what "unchanged" means. If there aren't, say so, and consider adding characterization tests for the part you're touching.
2. **Work in small, reversible steps.** Rename, extract, inline, or reorganize one thing at a time rather than rewriting wholesale. Big-bang rewrites are where regressions hide.
3. **Prefer the highest-leverage moves.** Usually: fix misleading names, then break up functions that do several things, then remove duplication, then untangle dependencies. Cosmetic formatting comes last.
4. **Preserve the interface unless asked.** Don't change public signatures, behavior, or output as a side effect of cleanup unless the user wants that.
5. **Show the delta.** Explain what changed and why, ideally before/after, so the user can see behavior is preserved.

## When *you* are the one writing the code

This skill often runs inside an AI that is itself generating the code. Martin devotes a chapter to this (Ch. 17). The key caution: an LLM produces text by statistical association, not by reasoning through the spec, so it can emit fluent, plausible code that doesn't actually do what was asked — he shows a model writing tests and then failing its own tests. Practical consequences when generating code: don't trust your own output because it reads well; check it against concrete examples and tests; surface the assumptions you made so the human can correct them; and treat the human as the final arbiter of what ships. See `references/ai-assisted-coding.md`.

## Core principles (quick reference)

Enough to handle most cases inline. The references expand each area with worked before/after examples.

**Names** — A name should reveal intent: reading it tells you what the thing is, does, or how to use it. Avoid `d`, `tmp`, `data`, `info`, `manager`, `do()`. Don't mislead. Make distinctions meaningful (`a1`/`a2`, `Product`/`ProductData`/`ProductInfo` are noise). Pronounceable, searchable; wider scope earns a longer name. Classes are nouns, methods are verbs. One word per concept (don't mix `fetch`/`get`/`retrieve`).

**Functions** — Small, doing one thing at a single level of abstraction; the body shouldn't surprise you given the name. Arrange so callers sit above callees and the file reads top-down. Few parameters (0–2 comfortable, 3 a smell, more usually a missing object). No boolean flag arguments — split into two named functions. No hidden side effects; separate commands (do) from queries (answer). Prefer exceptions over returned error codes; don't return null.

**Comments & formatting** — The best comment is the one made unnecessary by clear code. Keep *why*-comments, warnings, and public-API docs; delete restating comments and commented-out code. Read code like a newspaper: top-down, general to specific. Format consistently and follow the team's existing style over personal preference.

**Objects, classes & error handling** — Decide whether a type is an object (hides data, exposes behavior) or a data structure (exposes data, little behavior); hybrids get the worst of both. Don't reach through objects (`a.getB().getC().do()` — Law of Demeter; prefer "tell, don't ask"). A class should have one reason to change (SRP) and be cohesive. Use exceptions over error codes, give errors context, and don't return or pass null.

**Tests** — Test code is real code; keep it clean. One concept per test, clearly named, in arrange/act/assert shape. Aim for **F.I.R.S.T**: Fast, Independent, Repeatable, Self-validating, Timely.

**Design & architecture, in one breath** — Simplest design that passes the tests, expresses intent, has no duplication, and uses the fewest moving parts (YAGNI + Kent Beck's rules). Keep high-level policy ignorant of low-level detail, point dependencies inward toward the stable, abstract core (the Dependency Rule), and wrap volatile third-party detail behind a boundary you control. If a business rule imports a database or framework type, the arrow is pointing the wrong way.

## Going deeper

Load the relevant reference only when the task needs that depth. Each maps to a level of the framework.

- **`references/code-craft.md`** — *Code (line level).* Naming, functions and function heuristics, "one thing" / extract-method, the stepdown rule and newspaper metaphor ("Be Polite"), comments, formatting. Read when naming, function structure, or readability is the main issue.
- **`references/objects-and-design.md`** — *Code + Design.* Objects vs. data structures, the Law of Demeter, clean classes, error handling, simple design (YAGNI + Beck's four rules), SOLID, component cohesion/coupling principles, the Four Cs of continuous design, and concurrency. Read for design-level review and refactoring.
- **`references/architecture.md`** — *Architecture.* The two values of software (behavior vs. structure), independence, architectural and clean boundaries, and the Clean Architecture's Dependency Rule. Read when the task is structuring a system, drawing module/service boundaries, or managing dependency direction.
- **`references/testing-and-discipline.md`** — *Tests + Craftsmanship.* Testing disciplines (TDD, TCR, small bundles), clean tests and F.I.R.S.T, acceptance testing, and the practices that sustain quality over time: do no harm, the Boy Scout Rule, small cycles / CI-CD, relentless improvement, and honest professional conduct. Read when writing tests or advising on process and discipline.
- **`references/ai-assisted-coding.md`** — *Applying all of the above when an AI writes the code.* Why fluent output isn't proof of correctness, and how to keep the human as the final arbiter. Read when generating non-trivial code or advising on AI-assisted workflows.
- **`references/code-smells.md`** — A categorized checklist of code smells and heuristics for diagnosing problems. The backbone for reviews and for deciding what to refactor first.
