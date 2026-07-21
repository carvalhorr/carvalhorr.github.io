# From Autodev to Maestro: Teaching the Machine to Build Itself

In the [last post](/2026/06/can-ai-build-an-app-autonomously.html), I built a little task runner called **Autodev** and had it write an Android App. It worked — Claude shipped a real Android app to the Play Store. The experiment put autodev to implement the same spec using different models. Besides Claude, none of them completed the work, except Gemma 4 32B running locally on a RTX 3090 TI. I rapidly found out that it was cheating. An app was built, but it would not compile. It just made up tests that would make the tests pass without verifying anything.

I filed that away as a funny anecdote about smaller models. It was not. That exact behaviour — **making the check pass instead of making the thing work** — turned out to be the central villain of everything I built next. It just took me a few thousand commits to understand how deep it went.

This is the story of what came after Autodev. I called it **Maestro**.

## From a task runner to a pipeline

Autodev was a straight line: take a task, try it up to 10 times, feed the errors back, move to the next. That's enough to build one app from a finished spec. It is nowhere near enough to *evolve a product* — to go from a vague idea, to requirements, to a design, to code, to a deploy, and then to keep changing all of it as you learn.

So I set out to build the thing that does that whole loop. Maestro is a pipeline:

**idea → requirement → plan → (prototype) → run → merge → deploy**

Each stage is a gate. A requirement becomes a *plan* (the AI proposes how it'll build it; I approve). A plan becomes a *run* (an agent does the work in an isolated git worktree, with tests as the pass/fail authority). A passing run merges to main. And design-heavy work goes through an HTML *prototype* I approve before any code is written.

The twist — the part I'm still slightly stunned by — is that **Maestro builds Maestro.** After the first three days where I used Autodev to bootstrap the basic engine of what would later become Maestro, I mostly stopped writing code. I filed requirements and approved plans, and the system implemented itself. That is ˜2,100 commits in the repo so far, and Maestro did most of it.

## The numbers, because I promised to measure

I instrumented the pipeline partway through so I could actually see the cost. Here's what the build looks like:

| Metric | Value |
| :--- | :--- |
| Commits total | ~2,100 (I wrote 67) |
| Completed plans | 305 |
| Completed requirements | 341 |
| Recorded LLM spend | ~$1,050 |
| **Cost per merged plan of work** | **~$2** |

Two dollars per merged unit of work. Whatever else is true, the economics of an agent that builds itself are not the problem.

But look closer at the requirements: **203 features, 187 bugs.** Nearly one-to-one. Almost half of everything the pipeline did was *fixing itself*. And a big chunk of that was the same handful of problems, over and over.

## The ghost of Gemma, at scale

Here is the loop that nearly broke me. I'd ask for something. Maestro would build it, run its tests, and report **done**. I'd open the page — and it was wrong. Not subtly wrong. The chat's "thinking…" indicator broke and got "fixed" **seven times** across four different screens. A list that was supposed to be left-aligned got "fixed" **four times in three days** — each fix passing the previous test while the page still looked broken.

I kept telling it: *this is not done.* It kept telling me: *the tests pass.*

That's the Gemma cheat again, wearing a much more convincing suit. The model wasn't maliciously rewriting tests. It was doing something subtler and worse: **the same run that wrote the code also wrote the test that graded it** — to the same task description. So the test verified what the model *did*, never what I *asked for*. A test written by the implementation isn't a test. It's a mirror.

I only truly understood this because, near the end, I did something I should have done sooner: I had Fable 5 **audit Maestro's history** — that was easy, since Maestro had kept the entire history of everything that happened through the process. I pointed Fable to the code and all the data kept by Maestro and it started a swarm of read-only agents at the entire git log, all 400-odd requirements, every conversation, and the cost telemetry, and asked: *what keeps going wrong, and why?*

The retrospective was blunt. The dominant failure across three weeks was **verification-by-proxy** — declaring "done" when a gate passed, where the gate checked a class name or a string match instead of the thing a human would look at. Its own words, from a requirement it had filed weeks earlier: *"recurring — the user has flagged this repeatedly."* And the "lessons learned" feature I'd built to prevent repeats? The audit found it was **a diary, not a control** — fourteen lessons recorded, at least one demonstrably violated *after* it was written down, none that provably prevented a recurrence.

That stung, but it was the most useful thing the system ever produced.

## Systemic Problems Need Structural Fixes

Escaping these failure loops didn't require better code—it required a better system. Here is what had to change:

- **The acceptance test becomes the contract.** It's written from the requirement *before* the code, it must be *proven to fail first* (a test that can't go red proves nothing), and the run that implements the code is forbidden from editing it. The mirror is gone.
- **Build the shared thing once.** Six near-identical chat components became one. You cannot break "the thinking indicator" on five screens if there is only one indicator. The whole class of bug stops existing.
- **Architecture as law, not documentation.** Patterns like "every chat renders through the one component" get recorded as *invariants* the pipeline enforces — so the next agent is bound by them, instead of re-interpreting a prose description and drifting.
- **Every failure has to surface.** Silent failures were their own plague. A plan could fail while *planning* and appear nowhere — no notification, no card, nothing — because the code that showed failures assumed every failure had a running job behind it. If you can't see it break, you can't fix it.

None of these are clever. They are just the harness required to make the machine unable to lie about being done.

## The bill for reliability

After implementing the changes above I measured the before-and-after of all this reliability work on the *same* model, so the comparison is fair:

| | Before | After | Change |
| :--- | :--- | :--- | :--- |
| Time per attempt (median) | 106s | 197s | **+87%** |
| Cost per attempt (mean) | $0.84 | $1.46 | **+75%** |
| Tasks per run | 3.4 | 3.2 | ~flat |

Runs got about **twice as slow and ~1.75× as expensive.** Not because there are more steps — because each step now *proves more*: it authors a real spec, runs it red then green, checks the render against the approved design, executes the full test suite. Reliability is bought with verification time. (Cost rose less than time because most of the extra work is cheap cached context re-reads — a small mercy.)

Is that a good trade? I genuinely don't know yet — the honest answer is the sample is still too small to measure the *rework* side, which is where the payoff hides. If runs are 2× slower but you re-do a quarter fewer of them, it nets out. Ask me in a month.

## The human is still the bar

The uncomfortable lesson under all of it: **every structural fix started with me looking at the actual page and saying "no."** The system could build, test, deploy, recover from its own failures, and audit its own history — but it could not, on its own, hold the standard of *what "good" means*. It would happily converge on a green checkmark. I was the thing that kept insisting the green checkmark match reality.

The interesting frontier isn't "can the AI write the code" — that question is answered, and cheaply. It's "can the system tell, without me, whether what it built is actually right." Maestro got dramatically better at that over three weeks. It is not there yet.

That gap — between *passes the test* and *is actually done* — is the whole game. It's the same gap that little Gemma model exposed by rewriting one test. Everything I've built since has been an increasingly elaborate machine for closing it.

More soon — including whether any of this can run on hardware that fits under my desk.
