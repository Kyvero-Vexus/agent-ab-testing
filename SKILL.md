# Agent A/B Testing Skill

> Build fair comparisons between two agent conditions, measure what actually matters, and do not confuse harness bugs or benchmark leakage for signal.

## Purpose

This skill helps an OpenClaw agent design and run an **A/B comparison** between two agents or two agent conditions.

Use it when you need to compare:
- skill vs no skill
- prompt A vs prompt B
- model A vs model B
- think level A vs think level B
- tool access A vs tool access B
- context package A vs context package B
- workflow A vs workflow B

It is especially useful when the user wants a comparison that is:
- **fair**
- **repeatable**
- **source-grounded**
- **measured on multiple axes**
- **resistant to benchmark leakage and harness artifacts**

---

## Core Principle

**A/B testing is not just about changing one variable. It is about controlling everything else hard enough that the remaining difference means something.**

A weak A/B test usually fails in one of four ways:
1. the benchmark measures the wrong thing
2. the two conditions are not actually symmetric
3. the model is already at ceiling, so no difference can appear
4. the judge is too coarse, biased, or contaminated

This skill exists to prevent those failures.

---

## Definitions

- **A / control**: baseline condition
- **B / treatment**: changed condition
- **Invariant**: something that must be kept the same across A and B
- **Variable under test**: the one thing intentionally changed
- **Harness**: the execution environment, file layout, prompts, and run procedure
- **Benchmark artifact**: the task/test being given to both sides
- **Objective score**: directly checkable score (exact match, pass/fail, accuracy)
- **Judged score**: rubric-based evaluation by a third party or judge agent
- **Ceiling effect**: test is too easy, so both sides max out
- **Floor effect**: test is too hard/noisy, so both sides fail similarly

---

## When to Use This Skill

Use this skill when the user says things like:
- "compare these two agents"
- "do an A/B test"
- "see if this skill helps"
- "benchmark prompt A vs prompt B"
- "measure whether this context package actually improves results"
- "test model X against model Y on this task"

Do **not** use it when:
- the user only wants a one-off informal impression
- the task is too tiny to justify a controlled benchmark
- the two conditions differ in many uncontrolled ways and the user does not care

---

## Workflow Overview

1. **Define the variable under test**
2. **Define invariants**
3. **Build or choose the benchmark**
4. **Validate the harness before scoring**
5. **Run A and B serially unless concurrency is the variable**
6. **Collect raw outputs and runtime metrics**
7. **Score objective sections mechanically**
8. **Score subjective/explanation sections blindly with a rubric**
9. **Interpret results conservatively**
10. **Write lessons learned and iterate**

---

## Step 1: Define the Variable Under Test

State the one intended difference explicitly.

Examples:
- A: no skill, B: with skill
- A: low thinking, B: medium thinking
- A: base prompt, B: revised prompt
- A: no examples, B: 3-shot examples
- A: model X, B: model Y

Write this down plainly:

```text
Variable under test:
- Access to SKILL.md for B only
```

If you cannot say the difference in one sentence, the test is not ready.

---

## Step 2: Define Invariants

Keep these the same unless they are the variable under test:
- agent implementation
- model
- think level
- benchmark file
- output format
- scoring rules
- execution order logic
- time budget
- tool budget
- judge prompt
- file visibility rules

Write a small invariant block like:

```text
Common conditions:
- same base agent
- same model
- same think level
- same benchmark artifact
- same output format
- same scoring rubric
```

---

## Step 3: Choose the Right Benchmark

### Benchmark design rule
**Measure the capability you actually care about, not the easiest proxy.**

Examples:
- If testing a factual reference pack, objective recall may be appropriate.
- If testing a teaching/explanation skill, use audience-calibrated explanation tasks.
- If testing formal reasoning, use derivation/transfer tasks, not quote-the-book clozes.
- If testing workflow/tool use, use executable tasks and outcome checks.

### Good benchmark properties
A good benchmark is:
- **relevant** to the variable under test
- **scorable** with low ambiguity
- **hard enough** to avoid ceiling effects
- **grounded** enough to avoid arbitrary grading
- **cleanly isolated** from leaked answer sources

### Common benchmark types

#### 1. Objective exact-match
Best for:
- value outputs
- labels
- classifications
- theorem names
- multiple choice

#### 2. Executable / outcome-based
Best for:
- code tasks
- proof scripts
- file transformations
- tool-using workflows

#### 3. Rubric-scored explanation
Best for:
- teaching skills
- analogy/metaphor quality
- audience calibration
- formal-to-intuitive translation

#### 4. Hybrid benchmark
Often best in practice:
- objective anchor section
- explanation section
- optional judge section

---

## Step 4: Validate the Harness Before Scoring

Before treating any run as real signal, confirm:
- both sides can read the test artifact
- both sides can read exactly the files they are supposed to read
- neither side can obviously see the answer key
- output format instructions are clear enough to parse mechanically
- relative paths are not breaking one side differently than the other

### Critical harness lessons
- Prefer **absolute paths** to relative ones when using subagents.
- If one side cannot see the test file, **discard the run** as infrastructure failure.
- Do not score harness failures as model failures.

---

## Step 5: Run Serially Unless Concurrency Is the Variable

Default policy:
- **Run A and B in series, not in parallel.**

Why:
- avoids API rate-limit interference
- avoids queueing/caching artifacts
- avoids shared-resource contention
- reduces unexplained latency variance

Only run in parallel when the user explicitly wants to test concurrency effects.

---

## Step 6: Use Subagents for the Tests

When possible, run tests through subagents so the benchmarked condition is isolated from the parent agent’s reasoning stream.

Best practice:
- A and B should each be their own subagent run
- the parent agent should not “help” during the run
- the judge, if used, should be a separate subagent as well

---

## Step 7: Score Objective Sections Mechanically

Whenever possible, avoid hand-grading objective items.

Use scripts or deterministic rules for:
- exact answer matching
- multiple-choice grading
- JSON field presence
- executable test pass/fail
- accuracy percentages

Mechanical grading reduces:
- judge bias
- fatigue mistakes
- accidental movement of the goalposts

---

## Step 8: Score Subjective Sections Blindly

If there is an explanation or writing component:
- anonymize responses
- do not reveal which is A or B
- use a rubric with named criteria
- ask the judge for per-question subscores and a short comparison

### Good rubric dimensions
Pick dimensions that match the capability under test. Examples:
- correctness
- source-faithfulness
- clarity
- audience calibration
- pedagogical usefulness
- analogy/metaphor quality
- formal-to-intuitive bridge
- completeness
- concision/verbosity fit

### Important warning
A judge rubric can still saturate.
If both sides keep scoring near-perfect, either:
- the task is too easy
- the rubric is too coarse
- the judge model is not sensitive enough

---

## Step 9: Metrics to Record

Always record:
- accuracy / score
- runtime
- token usage
- exact common conditions
- exact difference under test

Often also record:
- failure modes
- formatting compliance
- parseability
- number of retries/timeouts
- variance across repeated runs
- objective score vs judged score

Suggested report structure:

```text
Condition A:
- score
- runtime
- token usage
- notable errors

Condition B:
- score
- runtime
- token usage
- notable errors

Shared conditions:
- model
- think level
- harness
- benchmark version

Interpretation:
- what changed
- how strong the evidence is
- what to test next
```

---

## Step 10: Interpret Results Conservatively

### If A = B at ceiling
Likely meaning:
- test too easy
- not enough discriminative power

### If A = B at floor
Likely meaning:
- test too hard or too noisy

### If B wins narrowly
Check:
- was the effect in the capability the skill was supposed to help?
- was the harness clean?
- was the judge blind?

### If B loses
That is still useful.
A/B testing is allowed to prove a change made things worse.

Do not silently rationalize away negative results.

---

## Common Pitfalls

### Pitfall 1: Testing memorization instead of understanding
Example:
- exact fill-in-the-blank from a source text

Fix:
- use transfer, explanation, or executable application tasks

### Pitfall 2: Benchmark not aligned with the skill
Example:
- testing a teaching skill with only multiple-choice recall questions

Fix:
- benchmark explanation quality directly

### Pitfall 3: Ceiling effects
Example:
- both A and B ace the test

Fix:
- make tasks harder, subtler, more adversarial, or more open-ended

### Pitfall 4: Coarse rubric saturation
Example:
- both explanation sets get perfect 5/5 everywhere

Fix:
- refine rubric dimensions or increase task subtlety

### Pitfall 5: Harness asymmetry
Example:
- A cannot see `test.md`, B can

Fix:
- use absolute paths, validate file visibility, discard broken runs

### Pitfall 6: Parallel interference
Example:
- runs share API contention / rate limits

Fix:
- run serially by default

### Pitfall 7: Judge contamination
Example:
- judge sees labels or benchmark key in a way that biases outcome

Fix:
- anonymize responses and isolate the judge packet

### Pitfall 8: Too many variables changed at once
Fix:
- change one main variable at a time

---

## How to Gather Questions for a Benchmark

### For source-grounded benchmarks
1. pick canonical source materials
2. identify the actual learning targets in those materials
3. extract examples/exercises/concepts
4. transform them into tasks that test the desired capability
5. keep an answer key or rubric aligned to source intent

### Transformation patterns

#### Source cloze → understanding question
Bad:
- fill in one missing token from a theorem statement

Better:
- explain why that theorem needs induction rather than reflexivity

#### Source example → transfer question
Bad:
- restate the exact output from the example

Better:
- ask what principle the example illustrates and why

#### Source exercise → executable proxy
Bad:
- ask whether the exercise exists

Better:
- ask the agent to solve a similar but not identical instance

### Benchmark question categories to mix
- recognition/classification
- why-this-is-typed / why-this-fails
- why-this-tactic / why-this-proof gets stuck
- structural equivalence vs equality
- pedagogical explanation
- metaphor/analogy generation
- audience calibration
- executable outcome prediction

---

## Recommended Iteration Strategy

When the benchmark is weak, iterate in this order:
1. fix harness asymmetry
2. align benchmark to the capability under test
3. reduce ceiling/floor effects
4. improve scoring resolution
5. repeat with clear versioning

Version benchmarks explicitly:
- v1, v2, v3, ...
- keep notes on why each version failed or improved

---

## Deliverables This Skill Should Produce

A strong A/B testing workflow should usually create:
- benchmark test file
- answer key or rubric file
- raw outputs from A and B
- grading script for objective sections
- judge packet for blind scoring when needed
- notes / lab notebook
- comparison report

---

## Minimal Template

```text
Variable under test:
- ...

Shared conditions:
- ...

Benchmark version:
- ...

Run order:
- A then B (serial)

Files:
- test
- answer key
- grader
- judge packet

Results:
- A objective
- B objective
- A judged
- B judged
- runtime/tokens

Interpretation:
- ...

Next iteration:
- ...
```

---

## Final Rule

**Do not claim a strong A/B result until you are confident the benchmark measures the intended capability more than it measures artifact, recall, or harness luck.**
