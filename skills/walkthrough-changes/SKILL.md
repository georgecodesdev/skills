---
name: walkthrough-changes
description: "Walks a user through a plan or completed change one part at a time, showing a visual that fits each part and waiting before continuing. Use when the user asks to be walked through a plan or change instead of being given a summary."
---

# Walkthrough changes

For a plan or completed change, explain the work in the order a person would understand it. Show one part, ground it in the real files and methods, and stop before moving on. This is a presentation workflow, not implementation, planning, or review.

Use this skill from `plan-changes` before approval, or from `execute-changes` after the work is staged.

**Terminology**: Throughout this skill, the **caller** is the party invoking the skill. The caller may be the human user directly, or another agent acting on the user's behalf. Any instruction to clarify something with, receive instructions from, report something to, or hand work back to the user should be understood as referring to the caller unless the surrounding context explicitly means an end-user of the software being changed.

## Workflow

Run these steps in order. Do not edit code or change the plan during a walkthrough.

### Step 1: Read the source

Read the plan, diff, or commit in full before presenting anything. Do not work from a summary or memory.

Find:

* the problem and how the current approach works today
* the change in one sentence, using "Given ___, do ___"
* the implementation order or the order the change runs in
* what stays unchanged and which edge cases matter
* how the change is proved

**Completion Criteria**: The change is understood well enough to explain its parts in order, with no missing facts being guessed.

### Step 2: Choose the order and visuals

Break the change into parts. Each part should carry one idea.

Use this order unless the source calls for something clearer:

1. the problem and current path
2. the proposed shape
3. the change, one new thing at a time
4. the boundaries and edge cases
5. the proof

Choose the visual that fits the part. Do not make every part a graph.

**Completion Criteria**: Each part has one idea, a visual, and the real files, classes, or methods it will name.

### Step 3: Present one part

Present exactly one part, then stop. Use:

* a short, spoken introduction
* one visual
* one question asking whether to continue

Talk like an engineer walking a colleague through the change. Do not answer the question yourself or introduce the next part before the caller responds.

**Completion Criteria**: One part has been explained with a visual and a clear stopping point.

### Step 4: Wait and adjust

Wait for the caller. If they confirm, present the next part. If they ask a question or the part did not land, answer or rework that part before continuing.

For a plan, reaching the end of the walkthrough is not approval. Return to `plan-changes` and wait for the user's explicit approval before implementation can begin.

**Completion Criteria**: The caller confirms each part before the next one is presented.

### Step 5: Close

When all parts are understood, close with one sentence about the change and one sentence about what stays the same. Do not repeat the whole walkthrough.

Point to the plan file or commit that the walkthrough came from.

**Completion Criteria**: The caller has a complete understanding of the change and knows where to read it.

## Visuals

Pick the form that fits the idea:

* **Mermaid** for a real call chain, dependency graph, branch, or state change.
* **Sequence diagram** for a runtime interaction across participants.
* **Pseudocode** for a branch, loop, or contract.
* **Directory tree** for where code lives. Use `[new]`, `[modified]`, and `[removed]`.
* **Table** for a comparison or a small set of cases.
* **Before and after** when the important thing is what changed.
* **Screenshot or state sequence** when the surface is visual. Show the approved or built screen, one screenshot per state.
* **Plain text** for a short flow or ordering.

A filter, list, config change, or one-line branch usually reads better as a table or code block than as a graph.

## Grounding and highlighting

Name the actual files, classes, and methods. Show who calls whom when that is part of the idea. A reader should be able to open anything named in the walkthrough.

Highlight the one thing that changes in each visual. Use `classDef` in Mermaid, change markers in a tree, a changed row in a table, or the proposed side of a before-and-after. A baseline visual has nothing highlighted. If everything is highlighted, nothing is.

Always show the new behavior against the current behavior. Do not present a new method or service without showing where it fits.

## Things to keep in mind

* **One part per turn**: If a part introduces two ideas, split it.
* **Show rather than summarize**: Do not put a wall of text around a small visual.
* **Use the plan's language**: Background, Smallest design, How the current approach works today, Proposed approach, and Non-goals are useful when they fit.
* **Keep the check-in plain**: Ask whether to continue. Do not use a recommended answer for a comprehension check.
* **Keep it optional**: A walkthrough should be offered, not assumed.
* **Writing style**: Write the walkthrough and your messages the way a person would normally speak. Do not include em dashes, semicolons, or overly formal phrasing.
* **Context Preservation**: For a large plan or diff, it is expected that you delegate reading and fact-finding to targeted subagents so your context is preserved.
