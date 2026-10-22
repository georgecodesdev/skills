---
name: delegate-work
description: "Shared guidance for delegating work to subagents. Use it when deciding whether to delegate or before handing work to a subagent. Covers when to delegate, what context and instructions to provide, what the subagent should report back, when to start fresh, and how to check its work."
---

# Delegate work

This skill gives `plan-changes`, `execute-changes`, `review-changes`, `diagnose-changes`, and `walkthrough-changes` one shared set of delegation rules.

## The idea

Delegation helps in two ways. A subagent can handle a long read or search so it does not crowd the main thread. It can also take an independent task so the work can move in parallel. You still own the result.

Give the subagent one clear job, the context it needs, and a clear description of what to send back. Ask for findings and evidence, not a transcript.

## When to delegate

Delegate when the work is:

* **Exploration**: tracing a call path, understanding a subsystem, or checking documentation.
* **Bulk**: reading a large diff, a long log, many files, or many sources.
* **Independent**: work that can run alongside yours without touching the same edits.
* **A check**: verifying a built artifact, branch, or running service.

Size matters when you are moving a lot of reading out of the main thread. A small, scoped write or edit can also go to a subagent when it is independent and you can check the result.

## Give the subagent a brief

A subagent does not have the conversation that led to the task. Give it the context it needs, or point it to something it can read.

```text
Brief
  Goal:      what to answer or produce
  Scope:     the files, paths, services, commits, or sources to cover
  Pointers:  where the context lives
  Method:    how to do the work and what counts as evidence
  Report:    what to return and how much detail to include
```

Point to the plan, spec, notes, commit, or diff command instead of pasting a file the subagent can read. Give each subagent one job. Two jobs in one brief make it easier for one to get less attention.

If later subagents will need the results of an exploration, ask the first subagent to save its notes in a file. Point the next brief to that file instead of repeating the same reading.

When several subagents are writing at once, give each a separate set of files or a clearly separate part of the work. If their scopes overlap, run them in order or give the shared files to one subagent. A separate branch or worktree can help, but is not required.

## Sequence the work

Start the work that is ready. When a result comes back, check what it unblocked and start that next. Work that depends on an unfinished result waits.

Run independent work at the same time. Keep going on the parts that do not depend on it.

When a round of findings needs fixes, treat the round as one job and send it to one subagent. Sending each finding to a different subagent splits the context and can create conflicts.

## Ask for a clear report

Ask the subagent to return one of these outcomes, with evidence:

* **PASS**: The check ran and the result matched what was expected.
* **ISSUES**: The subagent found a defect it can prove. It should list every provable issue, not stop at the first.
* **BLOCKED**: It could not run the check or reach the information. It should say what was missing.

A missing check or missing evidence is a gap, not a pass. Evidence can be a command and its output, a file and line, or a source citation, depending on the task.

Keep the report short. Ask for findings and evidence, not a transcript of everything the subagent read.

For a write task, also ask which files it changed, which checks it ran and their results, and what remains unfinished.

## Start fresh or resume

Start a fresh subagent for a fix, retry, follow-up, or the next item in a queue. Give it the original brief, later instructions, the previous report, and the branch or commit it should start from.

Resume an existing subagent only when you need state that is hard to move, such as its uncommitted changes or a process it is still running. A stop order alone is not a reason to resume it.

Instructions can get lost when work continues across interruptions. If in doubt, start fresh with the full brief.

## Check the result yourself

You own the result. Bring the returned changes together and resolve any conflicts. Then read the diff, rerun the check, or open the artifact before reporting what happened. Write your own summary instead of repeating the subagent's conclusion.

## Keep delegated work contained

A subagent assigned to run a skill should finish that job and return. It should not invoke the same skill and hand the same work to another subagent. The harness controls how deeply subagents can delegate.
