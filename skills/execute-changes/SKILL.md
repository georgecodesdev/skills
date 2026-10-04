---
name: execute-changes
description: >-
  Executes an approved plan step by step: builds each planned change, runs the plan's checks on the real system, reviews the changes, and leaves the approved work staged for the caller. Use this after plan-changes has been approved, or when the user says phrases like "start implementing", "execute the plan", "go for it", "build this", or "work through the plan". The plan is the fixed input, so this skill does not re-plan or redesign.
---

# Execute changes

For any approved plan, we want to build the change as agreed and check that it works. To achieve this, we follow the plan one step at a time, run its checks, review the finished changes, and leave them staged for the human to inspect and commit.

**Terminology**: Throughout this skill, the **caller** is the party invoking the skill. The caller may be the human user directly, or another agent acting on the user's behalf. Any instruction to clarify something with, receive instructions from, report something to, or hand work back to the user should be understood as referring to the caller unless the surrounding context explicitly means an end-user of the software being changed.

## Workflow

Run these steps in order. Each work step is one entry from the plan's Implementation order. Do not edit code in Steps 1 and 2, and check each step before moving on to the next one.

### Step 1: Load the approved plan and confirm it is executable

First, resolve which plan you are executing. Use the plan the caller names. If no plan is named, look for a `<SCOPE_SUMMARY>_PLAN.md` in the working repo. Use it if there is exactly one. If there are none or more than one, ask the caller which plan to use. Then read the plan in full rather than working from memory.

Record the current branch, its starting commit, and the output of `git status --short`. If a report already exists, use the starting commit recorded there when resuming. Keep the approved plan and report separate from code changes. If there are other user changes, use a separate worktree or ask the caller how to keep them safe. Do not reset or stage unrelated changes.

Before writing code, call the Skill tool with `verification-contract` and follow its instructions for the check format and report location. Read the repo's own docs for the commands, including `AGENTS.md`, `Makefile`, and `README`. From the plan and those docs, collect:

* The plan's **Verification setup**, which says how to start the system, tell when it is ready, use the feature, save results, and stop what was started.
* The **Check** block for every step in the Implementation order.

A step with no expected result or command we can run is missing a check. Stop and resolve this with the caller before coding.

Turn the Implementation order into a checklist. Create the report and save evidence under `.verification/<scope>/` at the root of the repo being changed, following `verification-contract`. For a new report, put the branch, starting commit, and initial `git status --short` output at the top. When resuming an existing report, keep its original starting commit and header. Check that the evidence and scratch folders are gitignored there. If a report already exists, read it and resume steps marked `fail` or `inconclusive`. Skip steps marked `pass` unless nearby code has changed.

**Completion Criteria**: Every plan step has a command we can run or a clearly named gap, the verification setup is understood, and the checklist and report are ready.

### Step 2: Confirm the verification setup works

Before writing code, check that the planned commands can run. If the plan needs a running system, follow its Launch instructions and run its Ready check. If a step needs test data or authentication, confirm that it is available too.

If the verification setup cannot run, mark the affected steps `inconclusive` and tell the caller what is blocking them. Do not substitute a weaker check just to get a green result.

**Completion Criteria**: The planned checks can run, or the blockers are named and the affected steps are marked inconclusive before any code is written.

### Step 3: Work through one plan step at a time

Work through the checklist in order. For each step:

1. Make the smallest change the plan calls for.
2. Run the check from the plan and compare the result with what the plan says should happen.
3. Record the command and result in the report, following the format in `verification-contract`. For a bug fix, record the failing result before the fix and the passing result after it.
4. Run the repo's lint, typecheck, and nearby tests as specified in the plan or repo instructions.

You may make local commits as checkpoints while you work. Keep them on the current local branch and do not push them. Record the starting commit from Step 1 so you can remove only the commits made for this plan at the handoff.

Do not move on until the current step passes. If a check cannot run, record it as `inconclusive`, never as a pass. When a change affects a running service, job, or stored data, passing tests do not replace the plan's live check.

**Completion Criteria**: Each plan step is checked and recorded before the next begins, and the result matches what the plan says should happen.

### Step 4: Diagnose a failed check

When a check fails and the cause is not obvious, call the Skill tool with `diagnose-changes`. It will help reproduce the failure, find its cause, and verify the fix with runtime evidence.

Do not keep a change just because it might help. If the evidence rules out the hypothesis behind it, revert that change.

**Completion Criteria**: A failed check is either traced to a confirmed root cause with evidence, or reported as inconclusive with what was ruled out.

### Step 5: Clean the diff

Before review, remove narrating comments, dead code, guards for cases which cannot happen, compatibility shims, and unrelated edits. If a comment claims a constraint, encode it as a type, test, or lint instead.

Read the diff against the plan. Check that the planned work is present and that nothing unplanned has crept in.

**Completion Criteria**: The diff contains only what the plan justified and reads as finished.

### Step 6: Hand the working tree to review-changes

Call the Skill tool with `review-changes` and give it the starting commit from Step 1 as the comparison point. The review must include all changes made for this plan, including checkpoint commits. Resolve its findings and run the review again until it approves the changes. Report the exact commands you ran, what they showed, and anything you could not verify, with the reason.

If a fix materially changes the scope, call the Skill tool with `plan-changes` to revise the plan, then confirm the change with the caller before continuing. Do not silently expand an approved plan.

**Completion Criteria**: `review-changes` has approved the changes, and its findings are resolved or explicitly deferred by the caller.

### Step 7: Prepare one staged change set for the human

After review approves the changes, confirm that every local commit since the starting commit is a checkpoint commit for this plan and has not been pushed or shared. Use a soft reset to move the branch back to the starting commit without discarding the changes. Then stage only the files changed for this plan. This removes the checkpoint commits and leaves one complete change set staged for the human to review and commit. Do not create a final commit or push the branch.

If an unrelated commit exists since the starting commit, or any checkpoint commit was pushed or shared, do not rewrite history. Stop and ask the human how to proceed. Do not include unrelated changes, `.env` files, or secrets in the staged diff.

**Completion Criteria**: The plan's changes are staged as one change set, any checkpoint commits are no longer in local history, and no final commit has been created. The human can review the staged changes and make the final commit.

## Things to keep in mind

* **Keep the plan fixed**: If a step cannot be done as planned, stop there and ask the caller how to proceed. Do not silently change the plan.
* **Keep the scope tight**: A bug or cleanup discovered mid-execution becomes its own step. Do not fold it into the step you are working on.
* **Verification is owned by the plan**: The plan defines what counts as done. Run its checks rather than choosing weaker ones.
* **Inconclusive is an outcome**: A check which cannot run is never a pass. Report it as `inconclusive`, and continue only with the caller's call.
* **Context Preservation**: For any non-trivial exploration, broad edit, or code deep-dive task, it is expected that you delegate to targeted subagents to ensure that your context is preserved.
* **Keep the evidence**: Record the results and save the artifacts where a reviewer can find them after the services are stopped.
* **Writing style**: Write the report and your messages the way a person would normally speak. Do not include em dashes, semicolons, or other overly formal punctuation.
