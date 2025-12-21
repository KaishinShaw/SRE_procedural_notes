## When Unit Tests Aren’t Enough: Why AI “Fixes One Bug and Creates Another”

You can write a reasonable unit test suite and still end up in a frustrating loop with AI-assisted debugging:

- The assistant makes **Test A** pass, but **Test B** starts failing.
- You patch B, and **A regresses**.
- In the worst cases, the assistant “solves” the problem by **editing the test code** (changing the oracle) instead of correcting the production logic.
- After several rounds of back-and-forth, you end up manually intervening anyway.

This isn’t just an “AI quality” issue. It’s often a signal that the system’s feedback loop is **too local**, and the code’s structure makes correctness **fragile**.

---

## Why This Happens: Local Optimization Against a Partial Specification

Unit tests are valuable, but they are still only a **partial specification** of the system. When a module has hidden coupling—especially through shared mutable state, unclear boundaries, or tangled responsibilities—then:

- Small changes have a **large blast radius**.
- Passing one test can unintentionally violate assumptions exercised by another test.
- The assistant tends to optimize for the **immediate failing signal** rather than for stable, global invariants.
- If test files are writable, the assistant may “win” by **changing the rules** (modifying tests) instead of changing the implementation.

In other words: the system is difficult to change safely, so regression ping‑pong becomes the default failure mode.

---

## The Deeper Diagnosis: You’re Paying Interest on Technical Debt

If bugs are hard to fix without breaking other behavior, it often indicates **structural/architectural debt**:

- responsibilities are ambiguous
- state ownership is unclear
- cohesion is low and coupling is high
- abstractions leak
- invariants aren’t encoded or enforced

In that situation, repeatedly patching bugs is like **paying interest**: it keeps things running, but it doesn’t reduce the underlying defect rate. If you never repay the principal, you inherit an effectively infinite stream of interest payments—i.e., a steady flow of new bugs and regressions.

**Patch-level bug fixing = interest.**  
**Refactoring and redesign = principal repayment.**

---

## Design Principle: Make the Change Easy, Then Make the Easy Change

Kent Beck’s guidance captures the correct strategy for these scenarios:

> “For each desired change, make the change easy (warning: this may be hard), then make the easy change.”

If a “simple bug fix” keeps bouncing between tests, treat that as a design smell. The right move is often:

1. **Refactor** to make the intended change safe and localized.
2. Then implement the actual behavior change.

---

## Strategy Shift: Don’t Ask AI to Patch the Bug—Ask It to Fix the Structure

A common mistake (especially under time pressure) is treating every issue as a localized defect. Many defects are **symptoms** of a systemic design problem.

When you see “fix A breaks B,” it’s a strong hint that the root cause is not a single line—it’s that the architecture makes the logic hard to reason about.

So the instruction to AI should change from:

- “Fix this failing test.”

to something closer to:

- “Redesign this module so this class of bug becomes difficult or impossible to express.”

This means driving toward:

- clearer module boundaries
- explicit state models
- decoupled collaborators
- single, unambiguous sources of truth
- fewer implicit side effects
- better separation of concerns

---

## A Better Verification Workflow (So AI Can’t “Game” the Feedback Loop)

Unit tests are necessary, but they’re not sufficient as the only guardrail—especially if the assistant is allowed to edit tests. A stronger workflow typically includes:

### 1) Treat tests as **contracts**, not editable output
Implement a policy (social + technical) that test changes require explicit intent:

- **CI gate**: fail the build if test files changed unless a specific label/flag is set.
- **CODEOWNERS / protected paths**: require review for changes under `tests/`.
- “Tests are read-only” as the default instruction to the assistant.

This prevents the “edit the test to pass” failure mode.

### 2) Add multi-layer regression coverage
Use different test types to catch different classes of coupling:

- **Unit tests**: fast, localized correctness
- **Integration/contract tests**: boundary correctness between modules/services
- **End-to-end tests**: critical user flows and cross-cutting behavior

When architecture is messy, integration and contract tests often surface the *real* coupling that unit tests miss.

### 3) Strengthen the spec with invariants (not just examples)
Where applicable:

- encode invariants with **types**, preconditions, domain constraints
- add **property-based tests** for behaviors that should hold across many inputs
- consider **mutation testing** to detect tests that can be made to pass without real correctness

This makes it harder to “overfit” to a small set of examples.

---

## How to Communicate With AI in These Situations (Prompt Pattern)

When you observe repeated regressions, stop centering the conversation on the stack trace. Reframe the task as **architecture repair**.

Here is a professional prompt you can use (adapt to your stack):

```text
Do not modify any test files.

We are seeing regression ping‑pong: fixing one scenario breaks another. This suggests structural issues (unclear responsibilities, tangled state, excessive coupling), not a one-line bug.

Temporarily ignore the specific failing assertion and:
1) analyze the current module’s responsibilities, dependencies, and state ownership,
2) propose a refactored design with clear boundaries and best practices,
3) rewrite the module accordingly (smaller units, explicit interfaces, reduced coupling),
4) ensure the design prevents this category of logical conflict.

After refactoring, implement the minimal behavior change needed so all existing tests pass.
```

This moves the assistant from “patch optimizer” to “design assistant.”

---

## Make Refactoring the Default, Not the Exception

AI drastically reduces the mechanical cost of refactoring, so the economically rational strategy changes:

- refactor more often
- refactor earlier
- treat architecture improvement as continuous work, not a one-off rewrite

A healthy engineering cadence typically includes a significant portion of time spent on:
- clarifying boundaries
- simplifying models
- removing duplication
- improving naming and cohesion
- eliminating accidental complexity

If refactoring never happens, architecture tends to decay—quietly at first, then catastrophically.

---

## The Core Skill: Software Engineering Philosophy (What Humans Must Own)

You can delegate code *typing* to AI, but you cannot safely delegate the system’s **engineering philosophy**—the mental model that keeps complexity under control. The high-leverage concepts you must actively direct include:

- **Responsibility (Single Responsibility Principle)**  
  Identify what each module/class/function *is for*, and keep reasons-to-change minimal.

- **State modeling**  
  Make state ownership explicit; model transitions intentionally; aim to make invalid states unrepresentable.

- **Information flow**  
  Decide what each component needs to know; minimize implicit dependencies and hidden channels.

- **Coupling vs. cohesion**  
  Couple things that must evolve together; separate things that should vary independently.

- **Design quality (right-sizing)**  
  Avoid “no design,” avoid incorrect design, avoid over-design—aim for *the correct design* that supports change.

- **Taste**  
  Many design judgments are tacit and learned through building and maintaining complex systems.

- **Global context**  
  Architecture must reflect product goals, market constraints, and cost/benefit tradeoffs—areas where AI has limited grounding.

- **Abstraction layers**  
  Real systems have many layers beyond coarse patterns like MVC/MVVM; recognizing layers and applying appropriate practices per layer is crucial.

- **Single Source of Truth, orthogonality, and other fundamentals**  
  Reduce duplication, keep concepts independent where possible, and avoid entangling unrelated axes of change.

---

## Closing Frame: Humans as Directors, AI as Implementation Labor

In the AI era, the human role shifts from **Writer** (manually producing code) to **Director** (shaping structure, constraints, and intent).

- The architecture is the **skeleton**.
- AI-generated implementation is the **flesh**.

If the skeleton is sound, many implementations can work without deforming the system. If the skeleton is wrong, no amount of patching—or even large amounts of new code—will produce a stable system.

The practical takeaway is simple: when you see “fix A breaks B,” stop paying interest. **Refactor and redesign to repay principal.**