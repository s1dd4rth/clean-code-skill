# Code Craft (Part I: the line level)

The things you touch every time you write a line: names, functions, the order code reads in, comments, and formatting. Get these right and most other cleanliness follows.

## Contents
- [Naming](#naming)
- [Functions](#functions)
- [Function heuristics](#function-heuristics)
- [One thing and extract-method](#one-thing-and-extract-method)
- [Be polite: how code should read](#be-polite-how-code-should-read)
- [Comments](#comments)
- [Formatting](#formatting)

---

## Naming

A name is a tiny piece of documentation that travels with the code forever. The test of a good name: a reader who has never seen the code can guess what it holds or does from the name alone.

**Reveal intent.** The name should answer why it exists, what it does, and how it's used. If a name needs a comment to explain it, the name has failed.

```python
# unclear
d = 0  # elapsed time in days

# intent-revealing
elapsed_days = 0
```

```python
# what is this list? what do the magic numbers mean?
def get_them(the_list):
    return [x for x in the_list if x[0] == 4]

# the names now carry the meaning the comment used to
def flagged_cells(board):
    return [cell for cell in board if cell.status == FLAGGED]
```

**Don't mislead.** Don't call something `accountList` unless it's a list (`accounts` is usually better anyway). Avoid names that differ in tiny ways (`...HandlingOfStrings` vs `...StorageOfStrings`) — readers skim and confuse them. Avoid lowercase `l` and uppercase `O` as names; they read as `1` and `0`.

**Make distinctions meaningful.** If two things have different names, the names should say how they differ. Number series (`a1`, `a2`) and noise words (`Data`, `Info`, `Object`, `Manager`) carry no information. What separates `getAccount()`, `getAccountInfo()`, and `getAccountData()`? A reader can't tell — that's the problem.

**Pronounceable and searchable.** You discuss code aloud and grep for it. `genymdhms` can't be said and is hard to find; `generation_timestamp` can be both. A bare `7` in code is invisible to search — name it `MAX_RETRIES`.

**Scope sets length.** Short names suit short scopes (a loop's `i`); the wider the scope, the more descriptive the name should be. A module-level constant or public function must stand on its own.

**Right grammar.** Classes and variables are nouns (`Customer`, `parsed_request`); functions and methods are verbs (`save`, `delete_page`); booleans read as predicates (`is_empty`, `has_children`).

**One word per concept.** Pick one verb per idea and use it everywhere. Mixing `fetch`, `get`, and `retrieve` for the same operation makes readers hunt for a distinction that isn't there. Equally, don't reuse one word for two ideas (`add` meaning both "sum" and "append").

---

## Functions

The aim is a function that reads like a short, clear paragraph.

**Small, and one thing.** A function does one thing if everything in it sits at the same level of abstraction, one step below the function's name. A practical signal: try to describe it in a sentence without "and" or "then." If you can't, it's probably doing several things.

**One level of abstraction.** Don't mix high-level policy ("decide which report to run") with low-level mechanics ("append a byte to the buffer") in one function. Mixing altitudes forces the reader to constantly change focus. Keep each function at one altitude and let it call down to the next.

**Switch statements.** A `switch`/long `if-elif` on a type tends to appear, in slightly different forms, in many places — and every new case means editing all of them. When that pattern spreads, consider replacing it with polymorphism (each type implements a common interface), so a new case is a new class rather than edits scattered across the codebase. A single switch buried behind a factory is usually fine; the smell is the *repeated* switch.

---

## Function heuristics

**Few arguments.** Fewer arguments are easier to read and test. Zero is ideal, one or two fine, three a smell worth a look, more than three usually means a concept is missing (several arguments belong together in an object) or the function does too much.

```javascript
// what do these booleans mean at the call site? makeCircle(x, y, true, false)
function makeCircle(x, y, filled, dashed) { ... }

// a small value object makes the call self-describing
makeCircle(new Point(x, y), { fill: true, dash: false });
```

**No flag arguments.** Passing `true`/`false` to steer behavior confesses that the function does two things. `render(true)` tells the reader nothing. Prefer two clearly named functions (`renderForPrint()` / `renderForScreen()`).

**Command-query separation.** A function should either *do* something (a command, changing state) or *answer* something (a query, returning information) — not both. A function that both mutates and reports on state is hard to reason about. `if (set("username", "bob"))` — does that *set* the value, or *check whether* it's set? Split it.

**No surprising side effects.** A function should do what its name promises and nothing hidden. `is_valid_password` that *also* initializes the session will burn whoever calls it expecting only a check.

**Prefer exceptions to error codes.** Returning error codes braids the happy path with checks and tempts callers to ignore the result. Throwing lets the main logic read straight through.

```python
# error codes braid the happy path with checks
if delete_page(page) == OK:
    if registry.delete(page.key) == OK:
        log("deleted")
    else:
        log("config delete failed")
else:
    log("delete failed")

# exceptions let the intent read top to bottom
try:
    delete_page(page)
    registry.delete(page.key)
    log("deleted")
except DeleteError as e:
    log(f"delete failed: {e}")
```

**Don't return or pass null.** Returning `null`/`None` makes every caller responsible for a defensive check; the one that forgets is a crash waiting to happen. Prefer an empty collection, an explicit optional, a special-case object, or an exception. Avoid passing null *in*, too — it forces null-handling everywhere.

**Structured programming.** Keep control flow simple: one entry, and few exits. Early returns to handle guard clauses up front are fine and often clearer; the thing to avoid is deeply nested, tangled branching and jumps that make the flow impossible to follow.

---

## One thing and extract-method

The main tool for taming a large function is **extract-method**: pull a coherent, nameable chunk into its own well-named function. Done well, the original turns into a readable summary of its steps.

```python
# does several things at mixed altitudes
def handle_upload(file):
    raw = open(file).read()
    if len(raw) == 0:
        raise ValueError("empty")
    cleaned = []
    for line in raw.split("\n"):
        parts = line.split(",")
        if len(parts) == 3:
            cleaned.append(parts)
    db.execute("INSERT ...", cleaned)
    return len(cleaned)

# the top-level function now reads as a summary of the steps
def handle_upload(file):
    raw = read_file(file)
    rows = parse_rows(raw)
    save_rows(rows)
    return len(rows)
```

The judgment call (see the calibration section in SKILL.md, and the book's own debate appendix): extract `read_file`, `parse_rows`, `save_rows` because each is a real, nameable step at the same altitude — **not** to make the function shorter for its own sake. If a "step" is one trivial line used nowhere else, inlining it usually reads better. And beware *entanglement*: if the extracted functions share mutable state and have to be read together to be understood, the decomposition has made things worse, not better. Extract along genuine seams.

What counts as "large" is about responsibilities and tangles, not raw line count. A 30-line function that tells one clear story top to bottom can be perfectly clean.

---

## Be polite: how code should read

Code is written once and read many times, so optimize for the reader.

**The newspaper metaphor.** A source file should read like a news article: the most important, highest-level things at the top, details further down. A reader should be able to grasp what a module does from its top without scrolling into the weeds.

**The stepdown rule.** Order functions so that each calls others defined below it, and the level of abstraction descends as you read down the file. The reader can stop whenever they've gone deep enough.

**Don't make the reader ride an abstraction roller coaster.** When adjacent code lurches between high-level intent and low-level fiddling and back, the reader's altitude keeps changing and comprehension suffers. Keep neighboring code at similar levels.

Note that this is often *not* the order in which code is written — we write by discovering details, then should reorganize so the result reads top-down even though it wasn't built that way.

---

## Comments

Prefer code that doesn't need a comment — but some comments earn their place.

**Worth keeping:**
- *Why* comments — the rationale behind a non-obvious decision, a workaround, or a performance trick.
- Warnings of consequences ("not thread-safe", "O(n²) — fine only for small n").
- TODO/FIXME markers flagging known, intentional gaps.
- Public API / docstring documentation for code others call without reading.
- Clarification you genuinely can't express in code (e.g. the meaning of a value from an external system).

**Worth deleting:**
- Comments that restate the code (`i += 1  # increment i`).
- Commented-out code — version control already remembers it; left in place it rots and confuses.
- Misleading or stale comments that no longer match the code (worse than none).
- Mandated noise (a redundant doc block on every trivial getter).
- Journal/changelog comments and banner markers that good structure makes unnecessary.

A comment is often an apology for code that isn't clear enough. Before writing one, ask whether a rename or an extraction would remove the need. But — per calibration — don't swing to the other extreme: when a decision is genuinely non-obvious, a short comment is the responsible choice, not a failure.

---

## Formatting

Formatting is communication, and consistency matters more than any particular rule.

- **Vertical:** keep closely related code close; separate distinct ideas with blank lines. Declare variables near their use. Order top-down per the newspaper metaphor.
- **Horizontal:** keep lines comfortably readable rather than scrolling sideways; use whitespace to group related operands and separate unrelated ones.
- **Team over self.** A consistent style across the codebase beats any individual's preference. Adopt the project's conventions and let an automated formatter (Prettier, Black, gofmt, rustfmt) settle the mechanics so reviews focus on substance, not whitespace.
