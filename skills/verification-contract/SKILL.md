---
name: verification-contract
description: "The shared verification format and rules used by plan-changes and execute-changes. It defines the plan's verification setup, the check block for each step, where evidence is written, and the pass, fail, and inconclusive rules. Load it when writing a plan's verification, or when running a plan's checks."
---

# Verification contract

This is the shared guide for deciding what "done" means in a plan and how to check it during execution. `plan-changes` uses it to write the checks, and `execute-changes` uses it to run them. There is no need for a separate verification skill in each project.

## The idea

Each plan step needs to say what result will show that it is done and how to check that result. The executor then runs the agreed check instead of deciding what counts as done. If there is no observable result or runnable command, flag that while planning.

## Verification setup

For each plan, record how to start the system and check that it is ready to use. Use the repo's own docs first (AGENTS.md, Makefile, README, package.json). Write `n/a: <reason>` for a field that does not apply.

```text
Verification setup
  Launch:      how to start the system
  Ready check: how to tell it is ready
  Use:         how to use the feature (HTTP, CLI, browser, job runner)
  Evidence:    where to save the results
  Stop:        how to stop what you started
```

## Check block

Include one for each implementation step.

```text
Check
  Expected:  what should happen for the step to be done
  Command:   what to run to check it
  Setup:     what needs to be running or available first
  Failure:   what result would show that it did not work
```

## Writing checks and reports

Write checks and report entries in plain, direct language. Keep them easy to scan on a phone or a larger screen, using short headings, lists, and whitespace instead of dense paragraphs.

* **Expected**: Say what a user can observe or what result the system should produce. Do not describe internal implementation details as proof.
* **Command**: Give the exact command or steps to run. Include only the setup needed to run them.
* **Failure**: Say what result would show that the check did not pass.
* **Report**: Include the exact command, observed output, and result. Keep it brief and readable, not a formal report.

Use the same natural voice as the plan. Do not use em dashes, semicolons, or overly formal phrasing.

## Rules

1. Each step needs an Expected result and a Command. Without them, there is no way to tell whether it is done.
2. The Command must be runnable in the repo. If it is not, name the gap for the user to resolve before approving the plan.
3. Record the Command and its output. A check without evidence is not a pass.
4. If a check cannot run, mark it `inconclusive`, not passed.
5. Changes to running services, jobs, databases, or object storage also need a live check. Automated tests can help, but they do not replace it.
6. Keep the evidence after stopping the system so the reviewer can inspect it later.

## Where artifacts go

Write verification output under `.verification/<scope>/` at the root of the repo being changed, where `<scope>` matches the plan's scope summary. Each directory holds a `report.md`. Start a new report with the branch name, starting commit, and initial `git status --short` output, then add one entry per plan step with its expected result, command, observed output, and verdict. When resuming, keep the original report header and starting commit. Put logs, response bodies, screenshots, and other output in its `evidence/` directory. If the plan names another evidence location, record its path in the report and keep the report under `.verification/`.

Keep `evidence/` and `scratch/` out of git. Commit `report.md` only when the work needs an auditable trail. Delete the scope directory once the change is merged and the trail is no longer needed.

## Types of checks

* **Test**: a unit, component, or integration test that checks behavior through an appropriate code interface.
* **Live check**: use the running system the way a user would, and save the result.
* **Regression check**: run an important existing scenario against the old version and the changed version.

Not every step needs all three. Use the expected result to decide which checks apply. Include a live check when the change affects a running service or changes behavior an existing caller depends on.

## Prefer no check over a bad check

A test needs to fail when the behavior is wrong. If a test would pass even when the code under test did nothing useful, it does not prove the behavior. If no practical check exists, name the gap rather than claiming a pass.
