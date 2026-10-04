---
name: diagnose-changes
description: >-
  Diagnoses a failed check or a runtime symptom before fixing it: reproduce on the real surface, build a tight feedback loop, isolate the cause, confirm the mechanism with runtime evidence, fix the root cause, and re-run the original check. Use when a check goes red and the cause is not obvious, when a bug resists a first look, or for a flake, leak, performance regression, or wrong output. Do not use it to redesign, or to fix by guessing.
---

# Diagnose changes

For any failed check or runtime symptom, we want to understand the cause before changing code. To achieve this, reproduce the problem, find a quick way to check it, and use runtime evidence to confirm the cause before making the fix.

Use this skill when a check fails during implementation or when a bug is reported on its own. The cause is not clear yet, so first gather enough evidence to understand what is going wrong.

**Terminology**: Throughout this skill, the **caller** is the party invoking the skill. The caller may be the human user directly, or another agent acting on the user's behalf. Any instruction to clarify something with, receive instructions from, report something to, or hand work back to the user should be understood as referring to the caller unless the surrounding context explicitly means an end-user of the software being diagnosed.

## Workflow

Run these steps in order. Do not make the fix before Step 5. If the approved plan or Harness allows temporary instrumentation, keep it isolated from the real system and remove it when you are done.

### Step 1: Reproduce on the real surface

Before reproducing the problem, read the approved plan or Harness and use only the tools and actions it permits. If neither one says what you may use, ask the human user before taking action. Run the failing check on the real surface, not against a mock, and capture the exact command and output. If it does not reproduce with the approved setup, ask before adding a probe or changing the setup.

Try to reproduce it yourself before asking the caller for help. If you get blocked, explain what you tried and what you could not reach.

**Completion Criteria**: The failure is captured with the command and output which produce it, or reproduction is reported unreachable with everything that was tried.

### Step 2: Build a tight feedback loop

Find one fast, repeatable command that fails on this bug for the right reason. Start with checks and commands allowed by the approved plan or Harness. If the available check is slow or flaky, or no suitable check exists, ask before creating or changing a test, script, or setup.

**Completion Criteria**: One command, red on this bug, which fails for the intended reason and runs fast enough to repeat.

### Step 3: Isolate the cause

Narrow down the possible causes until one mechanism remains. Use a method only if the approved plan or Harness permits it. If it does not, ask the human user before proceeding.

* **History bisection.** If the repo has a usable git history and a clean baseline, and the approved plan or Harness permits it, use `git bisect` to find the first bad commit and read that diff. Getting to a clean tree must not destroy in-progress work. See **Stay within the approved scope**.
* **Compare hypotheses.** Form candidate causes and rule them out with the failing check. Each round should eliminate as many possibilities as you can.
* **Differential comparison.** Compare a known-good input, config, or environment against the failing one, changing one variable at a time.
* **Instrumentation.** If the plan or Harness permits it, add temporary logging or a probe in an isolated local copy to read the values along the path. Remove it before the fix lands. Otherwise, ask before adding it.

Read the source of anything you depend on instead of assuming how it behaves. Follow the data across boundaries that a search might miss, such as an API format, a database column, a feature flag, or another service that reads the same data.

**Completion Criteria**: The cause is narrowed to a specific mechanism, not a region of the code.

### Step 4: Confirm the mechanism with runtime evidence

Use runtime evidence allowed by the plan or Harness. Depending on the symptom, compare output with a known-good result, inspect logs or a stack trace, or compare a performance profile with a baseline. State the cause in one sentence and point to the evidence. If the evidence does not support it, return to Step 3.

If you cannot observe the cause, it is still a guess, however plausible it sounds.

**Completion Criteria**: One sentence naming the mechanism, backed by an artifact which a reviewer can inspect.

### Step 5: Fix the root cause

Make the smallest change that removes the confirmed cause, in the part of the system that owns it. Before editing, confirm the fix is within the approved plan. If there is no approved plan covering the fix, or the fix changes the plan's scope, stop and ask the human user to approve the change. Do not add extra safeguards without evidence that they are needed. If the evidence rules out an earlier hypothesis, remove any changes made because of it.

**Completion Criteria**: The change addresses the confirmed mechanism rather than a symptom, and nothing speculative ships.

### Step 6: Re-verify

Re-run the check from Step 2 and confirm that it passes for the right reason. Run nearby tests only when the approved plan or Harness permits them. If the fix affects shared code and checking another caller is not covered by that approval, ask the human user first.

Before calling the fix verified, explain what caused the failure and point to the evidence that supports the explanation.

**Completion Criteria**: The approved check is green and the original failure is gone. Any other checks are run only when the plan or Harness permits them.

## Stay within the approved scope

The approved plan or Harness defines what you are allowed to do during diagnosis. Stay within that scope.

Read `AGENTS.md` to learn what tools the repo provides, but do not treat a tool being installed or documented as permission to use it. If the next action would expand what the plan or Harness allows, stop and ask the human user for explicit approval before proceeding. This applies even when the action seems harmless or reversible.

## Things to keep in mind

* **Evidence before edits**: Do not change code until you have evidence for the cause, however obvious the fix may seem.
* **One hypothesis at a time**: Rule it out or confirm it before forming the next one.
* **Revert refuted hypotheses**: Remove changes made for explanations the evidence ruled out.
* **Inconclusive is an outcome**: If the cause cannot be confirmed, report what was ruled out rather than ship a guess.
* **Context Preservation**: For any non-trivial investigation, code deep-dive, or log-reading task, it is expected that you delegate to targeted subagents to ensure that your context is preserved.
* **Writing style**: Write the diagnosis and your messages the way a person would normally speak. Do not include em dashes, semicolons, or other overly formal punctuation.
