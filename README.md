# clean-code

A Claude Agent Skill for writing, reviewing, and refactoring code using **Robert C. Martin's *Clean Code* framework**.

Acts like a thoughtful senior engineer at your shoulder — it leads with the issues that actually matter, explains *why* each one bites, and resists the dogma trap of shredding every function into one-liners or stripping out every comment. It applies the principles with judgment, not as a checklist.

## What it covers

The full four-part framework of *Clean Code, Second Edition*:

- **Code** — meaningful names, small single-purpose functions, function heuristics (few args, no flag arguments, command/query separation, exceptions over error codes), the stepdown rule and newspaper metaphor, comments that earn their place, formatting, objects vs. data structures, the Law of Demeter, clean classes, and error handling.
- **Design** — simple design (YAGNI + Kent Beck's four rules), the SOLID principles, component cohesion and coupling (REP/CCP/CRP, ADP/SDP/SAP), the Four Cs of continuous design, and concurrency defense.
- **Architecture** — the two values of software (behavior vs. structure), independence, architectural and clean boundaries, and the Clean Architecture's Dependency Rule.
- **Craftsmanship** — testing disciplines (TDD, TCR, small bundles), clean tests and F.I.R.S.T, acceptance testing, do-no-harm, the Boy Scout Rule, small cycles / CI-CD, and relentless improvement.

Plus a chapter the first edition couldn't have: **applying clean code when an AI writes it** — why fluent output isn't proof of correctness, and how to keep the human as the final arbiter.

## How it works

The skill recognizes which of three jobs it's doing and follows that flow:

- **Review** — reads for intent first, then surfaces findings grouped by impact (Blocking / Should fix / Nits) with concrete fixes, instead of a nitpick avalanche.
- **Write** — applies the principles lightly in service of clarity, without over-building.
- **Refactor** — improves structure while preserving behavior, in small reversible steps, highest-leverage moves first, showing the delta.

A calibration section runs across all three to keep it honest — grounded in the Second Edition's own debate appendix, where both sides concede over-decomposition is real and that small-functions / few-comments guidance is directional, not absolute. The lean orchestrator (`SKILL.md`) handles the common, line-level cases inline; the deeper references load only when design or architecture is actually in play.

## Install

```bash
npx skills add s1dd4rth/clean-code-skill
```

(Or download the `.skill` file from Releases and load it in your Claude environment.)

## Try it

After installing, try prompts like:

- "Review this function and be straight with me about what's wrong."
- "Refactor this — we keep adding more cases and it's getting messy. Keep behavior the same."
- "Write a clean version of X that handles the edge cases."
- "Is this a clean way to structure our pricing logic in a growing FastAPI app?"
- "Why is this class so hard to change?"

The skill triggers on code-quality work even when you never say the words "clean code".

## Structure

```
clean-code-skill/
├── SKILL.md                              ← orchestrator (lean): the 3 modes + calibration + quick reference
└── references/
    ├── code-craft.md                     ← Part I: names, functions, heuristics, "be polite", comments, formatting
    ├── objects-and-design.md             ← Code + Part II: objects/data, classes, error handling, simple design, SOLID, components, concurrency
    ├── architecture.md                   ← Part III: two values, independence, boundaries, the Dependency Rule
    ├── testing-and-discipline.md         ← Tests + Part IV: TDD/TCR, clean tests, F.I.R.S.T, craftsmanship disciplines
    ├── ai-assisted-coding.md             ← applying clean code when an AI writes the code
    └── code-smells.md                    ← categorized smell/heuristic checklist (the review backbone)
```

The orchestrator is intentionally lean. Reference files load only when that level of the framework is actively in play.

## Other skills by the author

- **[babok-skill](https://github.com/s1dd4rth/babok-skill)** — facilitating business analysis with IIBA's BABOK Guide v3.
- **[ooux-skill](https://github.com/s1dd4rth/ooux-skill)** — object-oriented UX modelling via the ORCA process.

## Credits

- Source methodology: *Clean Code: A Handbook of Agile Software Craftsmanship, Second Edition* by Robert C. Martin (Pearson, 2025).
- This skill **paraphrases and applies** the framework in its own words and original examples — it does **not** reproduce the book's text or code listings. For the full treatment, buy the book.
- Built using the Anthropic [skill-creator](https://github.com/anthropics/skills) framework.

## License

MIT — see [LICENSE](./LICENSE). The MIT license covers this skill's original content only, not the underlying methodology.
