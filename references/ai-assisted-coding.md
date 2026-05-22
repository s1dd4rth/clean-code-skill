# Applying Clean Code When an AI Writes It

The Second Edition adds a chapter (Ch. 17, "AIs, LLMs, and God Knows What") that the first edition couldn't have, and it matters directly here: this skill frequently runs *inside* an AI that is generating the very code in question. The point of this reference is to turn Martin's argument into working caution for that situation.

## The historical framing

Martin's stance is calm, not alarmed. AI coding is, in his view, the next step in a long line of abstraction increases — machine code to assembler to Fortran to C to Python — each of which raised programmer productivity and each of which provoked predictions that programming was finished. Those predictions were always wrong: easier programming produced *more* projects and *more* demand, not less. "Programming by prompt" is simply the newest rung, and like every previous one, we're in the early, fumbling phase where nobody yet knows how to do it well.

So the right attitude toward AI-generated code is neither dismissal nor blind trust. It's the same craftsmanship applied one level up.

## The core limitation: fluent is not the same as correct

The sharpest warning in the chapter is about what an LLM fundamentally is. An LLM generates code by statistical association over patterns it has seen — it does not, at generation time, *reason* through the specification the way a programmer does. Martin demonstrates this concretely: he prompts a model with a detailed spec heavily loaded with test scenarios, and the model produces plausible code **and a set of tests that its own code then fails** — having apparently not honored parts of the spec (a definition of "word," a case-insensitivity constraint) that were stated plainly.

The lesson isn't "AI is useless" — the model was genuinely helpful and got most of it right. The lesson is that **plausibility is not proof**. Code that reads fluently, compiles, and looks idiomatic can still quietly fail to do what was asked, and the failure won't announce itself. You cannot tell correctness by looking at how confident the output seems.

## What this means when you (the model) are writing the code

Hold your own output to the standard you'd apply to a stranger's pull request:

- **Don't trust it because you wrote it, or because it reads well.** Re-derive correctness against the actual requirement, not against how natural the code looks. The fluency of your own generation is exactly the signal Martin warns is unreliable.
- **Check against concrete examples and tests.** Walk the code through the specific cases in the request — especially the edge cases and the constraints stated in words ("case-insensitive," "ignore blank lines," "no more than three"). Where you can, write and mentally (or actually) run tests; tests are how you convert "looks right" into "is right." Note that generating tests is not enough on its own — as the chapter shows, a model can write tests it then fails, so the tests have to actually be *run* and reconciled.
- **Surface your assumptions.** When a spec is ambiguous, say which interpretation you chose and why, so the human can correct a wrong guess before it costs anything. Silent assumptions are where prompt-driven code goes wrong.
- **Prefer precise, formal statements of intent.** The chapter's forward-looking claim is that effective AI programming will depend on precise, almost contract-like specifications — and that the *reasoning* an LLM lacks must be supplied by a person. Martin's image is that programmers become "the lawyers of AI": responsible for drawing up exact requirements and for the judgment the model can't perform. When you generate code, make the contract explicit so a human can ratify it.
- **Keep the human as the final arbiter.** However good the generated code, the person remains responsible for what ships. Present your work so it can be judged and overridden — show the reasoning, flag the risky parts, don't bury decisions. Help the human review; don't ask them to rubber-stamp.

## The through-line

None of this is special pleading about AI. It's the whole framework — small, well-named, well-tested, intent-revealing code, verified rather than assumed — applied to a new author who happens to be a statistical model. The principles in this skill exist precisely because confident-looking code that nobody verified is how systems rot, whether a human or a model wrote it. Clean code, tests you trust, and a human accountable for the result are what make AI assistance an increase in productivity rather than an increase in undetected defects.
