---
name: plan-changes
description: >-
  Plans a code change before implementation: Use this at the start of any non-trivial change, or when the user says phrases like "before we start", "no coding yet", "write a plan", or asks to get familiar with the code before a specific change / wants to understand how something works before making a change. The outputted implementation-plan HTML file will specify the current vs new/proposed approach, the specific files/methods to touch, implementation steps, tests per implementation step, and open questions. Persist the HTML file in the repo you are working in so later sessions can easily resume from it. It is important that you do not begin making changes until the user has approved the plan.
---

# Plan changes

For any non-trivial change, we want to lead with a planning phase which results in a written artifact and a hard stop for the user's express permission to continue. The urge to start coding right away as soon as you understand the task is exactly what this skill overrides.

This is a pre-change workflow. It is different from a post-change workflow. With this skill, we are producing a plan before touching/editing any code, not auditing/reviewing the diff after.

## Workflow

Run these steps in order. It is important that you do not edit code in Steps 1-4. The first code edit happens only after the user has allowed you to progress.

### Step 1: Reach a shared understanding

This is where you build a full end-to-end understanding and mental model of the following:

* Any reference documents/links/artifacts that the user has given you.
* The intent behind the change that the user is planning.
* If applicable: The relevant parts of the codebase(s) which may be part of this change. You are expected to start at the relevant entry points, then dive deeper to build a complete understanding of the following:

  * How is the current codebase architected/structured?
  * What are the codebase's conventions?
  * The existing codepath(s) which this change will start from, or interact with.
  * If applicable: The closest existing example present in the codebase to follow.
  * Any shared code, utils, or types which this change will interact with / touch / or change.

To achieve this step's outcome, it is expected that you clarify your understanding by interviewing the user until you reach a shared understanding. Map this as a design tree: Every decision branches into the decisions that hang off it.

The tree needs to be worked in rounds. The frontier is every decision whose prerequisites are already aligned on: the questions you can ask now without guessing the answers you have not heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like this:

```text
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree. Settled decisions push the frontier outward and unblock questions which depend on them. Recompute the frontier and ask the next round. A question whose answer depends on an unanswered question still open in this round belongs to a later round, not this one.

Finding facts is your job, never the user's. When a frontier question needs a fact, dispatch a sub-agent to find it. Do not ask the user anything which you could look up yourself. Do not block on your exploration: a running exploration is an unsettled prerequisite, so only the questions which are downstream of it need to wait for the sub-agent to report. You can ask the rest of the frontier now. The decisions are the user's. Put each of them to the user and wait for their answer.

**Completion Criteria**: The frontier is empty: every branch of the design tree has been visited, and nothing is left assumed. Do not progress until the user has confirmed that you have reached a shared understanding.

### Step 2: Scope the change

Simply/clearly the specific scope in 1-2 sentences: What are the entry point(s)?, what are the methods which we will be creating/modifying?, or what is the specific set of features which this plan will cover?. Just saying that the scope is "the whole feature" is too broad, we need to narrow it to specifics.

**Completion Criteria**: The scope is a well defined and grounded. It is expected that the we can describe the scope as a 1-3 word "scope summary", and that summary would be specific and meaningful.

### Step 3: Draft the plan

Fill every section of the plan template below. Do not code. It is expected that you save your plan as a self-contained `.html` file in the repo/folder you are working in named `<SCOPE_SUMMARY>_PLAN.html`.

Your north star here is to create a proposal which is easily digestible and understandable to the user, with clear visual clarity and separation. Inline CSS (Tailwind) is expected, and, if needed, inline JS, as it is expected that these plan documents be self-contained and viewable without the need for any bundling.

**Completion Criteria**: The draft plan is saved as expected.

### Step 4: Wait on user review/approval

Present the plan and stop. You must wait for the user to move forward.

**Completion Criteria**: An explicit user approval ("start implementing", "go for it", etc.) is received before any code can be edited. You are forbidden from making any modifications to code before you receive user approval.

## Plan template (in-body reference)

Note that it is expected that you populate/fill in each section. This is the aligned-on shape, and it's expected that it is kept consistent run to run.

* **Background**: A brief overview of the changes, and intent behind those changes, as you understand them.
* **Context / How does the current approach work today**: This section acts as an easy-to-digest summary which establishes the baseline, from which your proposed changes will be framed against.
* **Proposed approach**: The changes you are proposing. When introducing your proposed changes, they need to be framed against how the current approach works today. As you are introducing your changes, you are expected to give Mermaid diagrams and tree diagrams, specify types/interfaces, and pseudocode.
* **Files and methods to change**: A scoped list of only what will be added/changed/modified as part of this plan. Do not list any code which you are not planning on changing.
* **Implementation order**: A set of sequenced implementation steps. The goal of a given step is to ensure that the associated changes are easy to review and read. 
* **Tests**: The tests which you intend to perform at each step. The nature and style of the tests are expected to be consistent with the codebase you are working in.
* **Open Questions**: Anything that will need the user's input in order to progress.

## Things to keep in mind

* **Resume from an existing plan**: For a change on an already-planned feature, read the existing plan/progress first rather than re-driving from scratch.
* **Persist the plan when asked, or to save your work**: Save your plan to the repo/folder you are working in so a later session can read it back and resume.
* **Writing style**: This should not be thought of as a formal report, but instead as a mechanism to clearly and simply articulate the plan before getting sign-off from the user. Do not include em dashes (-), semicolons, or other overly formal punctuation. Your writing style and tone should reflect the way a person would normally speak.
