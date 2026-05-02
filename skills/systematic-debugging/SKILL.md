---
name: systematic-debugging
description: >
  Systematic 4-phase debugging methodology for any bug, test failure, or unexpected behaviour.
  Use when encountering errors, test failures, build failures, performance problems, or any
  situation where the root cause is unclear. Also invoke when a fix attempt hasn't worked, when
  multiple fixes have been tried without success, or when someone says "I can't figure out why
  this is failing", "this should work", "it was working before", or "why is X happening".
  Do NOT guess or apply fixes before completing Phase 1.
---

# Systematic Debugging

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** Find the root cause before attempting any fix. Symptom fixes are failure.

## The Iron Law

No fixes without completing root cause investigation first (Phase 1). If Phase 1 is not
complete, no fix may be proposed.

## The Four Phases

Complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**Before attempting any fix:**

1. **Read error messages carefully** — do not skim. Stack traces contain the answer more often
   than not. Note line numbers, file paths, error codes.

2. **Reproduce consistently** — can you trigger it reliably? What are the exact steps? If it
   is not reproducible, gather more data rather than guessing.

3. **Check recent changes** — what changed that could cause this? `git diff`, recent commits,
   new dependencies, config changes, environmental differences.

4. **Gather evidence at component boundaries** — when a system has multiple layers (API →
   service → database, CI → build → deploy), add diagnostic instrumentation at each boundary
   before proposing fixes:

   ```bash
   # Log what enters and exits each layer
   # Verify environment and config propagation
   # Check state at each boundary
   # Run once to see WHERE it breaks, then investigate that specific layer
   ```

   This reveals which layer is actually failing, rather than which layer you guessed.

5. **Trace data flow** — when the error is deep in a call stack, trace backward:
   - Where does the bad value originate?
   - What called this function with the bad value?
   - Keep tracing upward until you find the source
   - Fix at the source, not at the symptom

### Phase 2: Pattern Analysis

**Find the pattern before fixing:**

1. Locate similar working code in the same codebase
2. Read any reference implementation **completely** — do not skim
3. List every difference between working and broken, however small
4. Understand what dependencies, config, or environment the working version assumes

### Phase 3: Hypothesis and Testing

**Scientific method:**

1. State a single, specific hypothesis: "I think X is the root cause because Y"
2. Make the **smallest possible change** to test the hypothesis — one variable at a time
3. If the hypothesis was wrong, form a new one — do not add more fixes on top
4. If genuinely uncertain, say so — do not pretend to know

### Phase 4: Implementation

**Fix the root cause, not the symptom:**

1. Write a failing test that reproduces the bug before writing the fix
2. Implement a single fix addressing the identified root cause
3. No "while I'm here" changes — no bundled refactoring
4. Verify: test passes, no other tests broken, issue actually resolved

**If the fix does not work:**

- After **3 failed fix attempts**, stop trying more fixes
- Three failures signal an architectural problem, not an implementation detail
- Diagnose: is each fix revealing new coupled state or a new symptom somewhere else?
- If yes, the architecture needs discussion — continue fixing symptoms will make it worse
- Raise the architectural question explicitly before attempting fix #4

## Rationalisations to Reject

| Excuse | Reality |
|--------|---------|
| "Issue seems simple, skip the process" | Simple bugs have root causes too. The process is fast for simple bugs. |
| "Emergency, no time" | Systematic debugging is faster than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write a test after confirming the fix works" | Untested fixes don't stick. Test first proves the fix. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Stop and question the pattern. |

## Red Flags — Stop and Return to Phase 1

If any of these are happening:

- Proposing a fix before completing Phase 1
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- Making multiple changes simultaneously
- "It's probably X, let me fix that" (without evidence)
- "I don't fully understand but this might work"
- Each fix is revealing a new problem in a different place

**All of these mean: stop, return to Phase 1.**

**Three or more failed fixes:** question the architecture before attempting another.

## Quick Reference

| Phase | Key Question | Done When |
|-------|-------------|-----------|
| 1. Root Cause | What is actually happening and why? | Can state the root cause with evidence |
| 2. Pattern | What does working code look like? | Identified the difference |
| 3. Hypothesis | What specific change should fix it? | Hypothesis is falsifiable and specific |
| 4. Implementation | Does the fix hold? | Test passes, no regressions |
