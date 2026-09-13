---
name: review-changes
description: >-
  Reviews a code change using three rigorous, read-only passes: architecture/structure, readability/maintainability, and regression/deployment risk. Use this to get high-quality feedback on your work, or when the user says things like "can you review these changes", "will this break anything", "review this diff", or "I am about to push this code".
---

# **Review changes**

For any non-trivial code change, we want to ensure that the code meets our quality bar. To achieve this, we run a three-pass review intended to emulate a senior software engineer reviewing the given changes. The review passes are scoped to the following areas:

* Architecture / Structure
* Readability / Maintainability
* Regression / Deployment risk

The architecture pass asks whether the change is designed correctly in the first place. The readability pass asks whether the resulting implementation is clear and maintainable. The regression pass asks whether the change can be safely deployed without unexpectedly breaking existing behavior.

Each review pass is expected to be extremely thorough and rigorous. Measure twice, cut once.

**Terminology**: Throughout this skill, the **caller** is the party invoking the skill. The caller may be the human user directly, or another agent acting on the user's behalf. Any instruction to clarify something with, receive instructions from, report something to, or hand findings back to the user should be understood as referring to the caller unless the surrounding context explicitly means an end-user of the software being reviewed.

## **Workflow**

Run these steps in order. Note that this is a read-only operation and you are forbidden from making code changes as part of this process. Your job is to review the changes, not make code changes yourself. You are also forbidden from stashing/committing or modifying the state of the git repo(s) in any way.

### **Step 1: Determine what files are in-scope for the review**

This can be done either by doing one or more of the following:

* Looking at the staged changes
* Looking at the uncommitted changes
* Looking at a specific commit

If you are working across multiple code packages, which may themselves be different git repositories, it's important that you explore every modified file across every code package. Leave no stone unturned, as missing relevant/apposite files will result in an incomplete review.

With this said, a review needs to have a specific scope in mind. If you are unclear or need clarification to align on exactly what changes you will be reviewing, it is expected that you ask the caller before proceeding. You are only allowed to exclude files from your review if they have been explicitly named.

**Completion Criteria**: A list of every modified file which will be in the scope of this review, grouped by the code package which they belong to. No files should be dropped, and excluded files need to be explicitly named.

### **Step 2: Build a complete mental model of the in-scope files for the review**

Now that you understand what files are in-scope for the review, it is important you take your time re-familiarizing yourself with the changed files by reading the files as they are, not how you remember them. You are not allowed to rely on your memory of what was changed or modified in a given file. Instead, you must re-read the file to get its accurate post-change state.

It is also expected that you build out a "dependency tree" of the modified files, as otherwise you will not be able to construct the mental model needed to give a complete review. However, this dependency tree needs to be bounded/capped within the code packages you are working within. You are being asked to get a complete mental model of the files for review, not an understanding of the entire dependency ecosystem you are working in.

**Completion Criteria**: Every file from the prior step has been read in full, and its dependency tree has been constructed. You have a complete mental model of the in-scope files for the review.

### **Step 3: Code architecture / structure**

Before judging how well the code is written, judge whether this is the code which should have been written in the first place. Compare the change against the smallest design which satisfies the requirements of the requested change.

This pass is subtractive by design. You are reviewing the architecture of the change as a whole, not judging each new file, class, helper, or type independently. Reviewing each piece only on its own merits can result in approving a change where every individual piece is defensible, but the resulting system is unnecessarily complicated when taken as a whole.

Note: This design pass is concerned with the code this change introduces or reshapes. Do not use it as an excuse to redesign the surrounding system, and do not hold the change to an architecture which the rest of the codebase does not follow.

It is expected that you internalize and apply the Code architecture / structure guidelines outlined below against the in-scope files.

**Code architecture / structure guidelines**

* **Every new concept survives the deletion test**:

  * For each new class, function, type, helper, service, wrapper, etc. which is added, ask what current production capability would be lost if it were deleted and its behavior moved or co-located into its only consumer.
  * "Nothing", "only a test would break", "a future change will need it", or "it makes another file shorter" are not sufficient reasons for a concept to exist.
  * In those cases, prefer deleting it, inlining it, or moving/co-locating it under the concept which actually owns the behavior.
  * Do not polish an abstraction which should not exist.

* **The implementation uses the fewest concepts needed to express the behavior clearly**:

  * Count conceptual complexity, not lines of code.
  * New layers, services, helpers, adapters, modes, and state all create additional things a reader needs to understand.
  * Prefer a design with fewer moving pieces when it expresses the same behavior just as clearly.
  * Do not introduce structure solely because it might be useful for a hypothetical future requirement.

* **Ownership is clear**:

  * Every behavior should have an obvious owner.
  * New logic should live with the module, service, component, or domain concept which is responsible for it.
  * Avoid designs where responsibility is split across several files without a meaningful architectural boundary.
  * A reader should be able to answer "where does this behavior live?" without tracing through unrelated layers.

* **Abstractions represent real boundaries**:

  * An abstraction should correspond to a meaningful domain, lifecycle, dependency, or reuse boundary.
  * Be suspicious of wrappers or interfaces which simply mirror another object without changing ownership or simplifying the mental model.
  * An abstraction should hide meaningful complexity or establish a useful contract, not merely move code into another file.
  * If removing the abstraction makes the flow easier to understand without creating meaningful duplication, it probably should not exist.

* **Dependencies flow in a clear direction**:

  * The entry point should lead naturally toward the code which performs the work.
  * Avoid circular ownership, callbacks into higher layers, or dependencies which force the reader to jump backwards and forwards to understand the behavior.
  * Lower-level/shared code should not need to know unnecessary details about the feature-specific caller above it.
  * New dependencies should follow the architectural direction already established by the codebase.

* **Types make the mental model easier**:

  * Types should model meaningful states, contracts, and domain concepts.
  * Do not create types solely to give intermediate values names or to move an object shape into another file.
  * If the implementation repeatedly checks the same invariant, consider whether the type boundary should make that invariant explicit.
  * A type should reduce the number of states the reader needs to reason about, not increase them.

* **State lives in one canonical place**:

  * Avoid introducing multiple representations of the same state unless there is a clear boundary which requires it.
  * Derived state should generally remain derived rather than being independently stored and synchronized.
  * Be suspicious of changes which require keeping multiple objects, flags, caches, or fields in agreement.
  * If two pieces of state can disagree, understand why both need to exist.

* **The change fits the existing architecture**:

  * Look for the closest existing implementation of a similar feature or behavior.
  * Prefer extending the architectural pattern the codebase already uses rather than introducing a parallel pattern.
  * A locally cleaner design is not necessarily better if it creates a second way of solving the same problem in the repository.
  * Deviating from an existing pattern should require a concrete reason tied to the requirements of the change.

**Architecture north star**: The best architecture is the smallest set of concepts which accurately represents the behavior, gives each responsibility a clear owner, and makes the execution path easy to follow.

**Completion Criteria**: A list of findings, which may be empty. Each finding needs to identify the unnecessary or misplaced architectural complexity, explain why the current structure makes the system harder to understand or maintain, and recommend a simpler structure which remains consistent with the surrounding codebase.

### **Step 4: Readability / maintainability pass**

Now that the architecture of the change has been reviewed, evaluate how clearly and maintainably that design has been expressed in code.

Do not re-litigate architectural decisions from the previous pass here. This pass should focus on the implementation within the chosen structure: whether the code is idiomatic, legible, direct, focused, and easy for another engineer to modify safely.

It is expected that you internalize and apply the readability guidelines outlined below against the in-scope files.

**Readability / maintainability guidelines**

* **Comments are written when appropriate**:

  * Comments should explain why something is being done when that intent is not obvious from reading the code.
  * Do not use comments to explain code which should instead be made clearer.
  * Avoid comments which simply restate what the code is already doing.

* **Idiomatic practices are being used for the language/framework/library in question**:

  * Follow the normal/common patterns of the language, framework, or library we are working in.
  * Do not introduce custom machinery for something which already has a simple/idiomatic solution.
  * Prefer patterns which another engineer familiar with the language/framework/library would immediately recognize.

* **We are not being too clever**:

  * Prefer boring, direct, obvious code over dense, magical, or overly generic code.
  * The goal is not to write the fewest lines possible.
  * Optimize for making the implementation easy for another engineer to understand.

* **The implementation follows the existing codebase's conventions**:

  * Similar code should look and behave similarly.
  * Follow established naming, file layout, error handling, API, and implementation conventions unless there is a concrete reason not to.
  * Avoid introducing a new local style for something the codebase already expresses consistently.

* **Control flow stays simple**:

  * Be highly suspicious of unnecessary nesting, repeated conditionals, boolean modes, nullable states, and one-off special cases.
  * Prefer guard clauses or direct flow when they make the happy path easier to follow.
  * If understanding a function requires mentally evaluating several interacting conditions, look for a clearer expression of the same behavior.

* **Functions and methods remain focused**:

  * A function should have a clear purpose which can be understood without reading every implementation detail.
  * Avoid functions which mix unrelated responsibilities or operate at several levels of abstraction at once.
  * Split a function when doing so creates a meaningful conceptual boundary, not simply because it is long.

* **Names clearly communicate intent**:

  * Names should make it obvious what something represents or what a function does.
  * Avoid vague names when there is a more specific domain name available.
  * A reader should not need to inspect the implementation to understand what a variable, type, or method is supposed to represent.

* **Types make the implementation clearer, not harder to follow**:

  * Be suspicious of unnecessary `any`, `unknown`, casts, optional fields, nullable values, or loosely-shaped objects.
  * Prefer types which communicate the real shape and constraints of the data.
  * Avoid type machinery which is technically precise but makes ordinary use of the code difficult to understand.
  * Type assertions should not be used to silence a mismatch which should instead be represented correctly.

* **Local duplication does not obscure the source of truth**:

  * Look beyond literal copy/paste duplication.
  * Do not repeat the same calculation, condition, transformation, or business rule in several places within the changed code.
  * If the repeated behavior represents one concept, make it clear where that concept is defined.

* **Code does not contain incidental noise**:

  * Remove unnecessary temporary variables, pass-through methods, redundant branches, repeated conversions, or other implementation details which do not help communicate intent.
  * Prefer code where each line contributes either behavior or meaningful clarity.
  * Do not trade straightforward code for indirection merely to make individual functions shorter.

* **Tests prefer behavior over implementation details**:

  * Tests should assert the externally meaningful behavior of the code, not mirror its internal structure.
  * Be suspicious of tests which break simply because a helper was renamed, extracted, or reorganized while behavior stayed the same.
  * A refactor which preserves behavior should generally not require rewriting a large portion of the test suite.

* **Tests themselves remain simple and maintainable**:

  * Avoid bespoke test harnesses, excessive setup, or complicated helpers for simple behavior.
  * Reuse the existing fixtures/helpers/testing patterns in the codebase when they already solve the problem.
  * A reader should be able to quickly understand what scenario is being tested and what outcome is expected.

* **Tests do not over-mock**:

  * Be suspicious of tests which require large amounts of mock setup to exercise simple behavior.
  * We do not want tests which only prove that our mocks behave the way we configured them.
  * Prefer exercising meaningful boundaries with realistic values where practical.

**Readability north star**: Another engineer should be able to read the implementation from top to bottom and understand what it does, why it does it, and where they would change it without unnecessary mental translation.

**Completion Criteria**: A list of findings, which may be empty. Each finding needs to be associated with one of the above readability / maintainability guidelines and include a recommended fix.

### **Step 5: Regression / Deployment risk**

Unless otherwise instructed by the caller, assume that the existing code is running without issue in production. We want to reason from the diff outward and determine whether the proposed changes could regress existing behavior, introduce instability, or make the deployment unsafe.

For every meaningful change, ask: what behavior existed before, what behavior exists after, and what else depends on the behavior which changed?

It is expected that you internalize and apply the regression/deployment risk guidelines outlined below.

**Regression / Deployment risk guidelines**

* **Existing behavior is preserved unless the change intentionally modifies it**:

  * Establish what the relevant codepath did before the change.
  * Compare that behavior against the post-change implementation.
  * Call out any behavioral difference which does not appear to be required by the intended change.
  * Do not assume that a small diff means a small behavioral impact.

* **Trace changes through their consumers**:

  * When a method, type, API, event, shared helper, or other contract changes, inspect the code which depends on it.
  * Look for callers which may still rely on the old behavior or shape.
  * A change is not safe merely because the modified file itself is internally correct.

* **Pay particular attention to shared codepaths**:

  * Changes to shared helpers, utilities, middleware, base classes, common types, or infrastructure can have a much larger blast radius than the feature being worked on.
  * Identify every materially different behavior introduced into a shared path.
  * Verify that unrelated consumers continue to behave as expected.

* **Contracts remain compatible**:

  * Look for changes to function arguments, return values, types, API payloads, persisted data, events, configuration, defaults, and error behavior.
  * Check whether existing consumers can continue to use the contract correctly.
  * Be suspicious of changes which are technically type-compatible but semantically mean something different.

* **Default and fallback behavior has not accidentally changed**:

  * Pay close attention to changed defaults, optional values, null handling, fallbacks, and conditional branches.
  * Determine what happens when new inputs or configuration are absent.
  * Existing users/codepaths which know nothing about the new feature should generally continue behaving as they did before.

* **Edge cases around the changed behavior are accounted for**:

  * Consider empty inputs, missing values, duplicate values, partial state, retries, errors, and boundary conditions where relevant.
  * Focus on edge cases introduced or affected by the change rather than attempting to audit the entire existing system.
  * A new branch should be reviewed both for when it executes and when it does not execute.

* **State transitions remain valid**:

  * When the change reads or modifies state, reason through the possible before/after states.
  * Look for partial updates, stale state, impossible combinations, or paths which can now leave the system in an unexpected state.
  * Where multiple related writes happen, consider what happens if only some of them succeed.

* **Ordering and concurrency assumptions remain valid**:

  * Look for changes to async execution, sequencing, parallelism, retries, queues, transactions, or lifecycle ordering.
  * Determine whether the implementation depends on operations happening in a particular order.
  * Call out races or timing assumptions introduced by the change.

* **Failure behavior remains safe**:

  * Determine what happens when newly introduced operations fail.
  * Look for swallowed errors, changed exception behavior, missing cleanup, incomplete rollback, or failures which can leave state partially applied.
  * A happy-path implementation is not sufficient if realistic failures can destabilize an existing codepath.

* **Deployment compatibility is considered**:

  * Do not assume that every part of a distributed system will update at exactly the same time.
  * When relevant, consider old-code/new-code compatibility during rollout.
  * Pay particular attention to schema changes, persisted data, APIs, messages/events, feature flags, and shared contracts which may temporarily be consumed by different versions of the application.
  * Call out changes which require a specific deployment order or coordinated rollout.

* **Tests protect the behavior which is actually at risk**:

  * Look at the tests associated with the changed codepath and determine whether they exercise the behavior which could regress.
  * New tests should focus on the behavioral boundary introduced or modified by the change.
  * Do not treat test coverage alone as evidence that a change is safe. Understand what the tests actually prove.

* **Security-sensitive behavior has not been weakened**:

  * Look for changes which bypass or weaken existing authentication, authorization, validation, sanitization, or permission checks.
  * Pay particular attention when new or changed input reaches security-sensitive boundaries such as database queries, filesystem paths, shell commands, URLs, or client-visible output.
  * Make sure sensitive data such as secrets, tokens, credentials, or private user data is not newly exposed through responses, logs, or errors.

**Regression north star**: Start at the changed behavior and follow its blast radius. We are trying to answer: "if this code is deployed exactly as written, what existing behavior could unexpectedly stop working?" Do not invent hypothetical risks without a concrete path from the proposed change to the regression.

**Completion Criteria**: A list of findings, which may be empty. Each finding needs to be associated with one of the above regression / deployment risk guidelines and include a recommended fix.

### **Step 6: Hand-off findings**

Now that all three review passes are complete, hand the findings back to the caller in a clear and actionable way.

Findings should be grouped by review pass:

* Architecture / Structure
* Readability / Maintainability
* Regression / Deployment risk

Within each pass, findings should be ordered from highest to lowest impact. Do not bury meaningful issues underneath low-value cleanup.

Each finding must include:

* **What the issue is**: Clearly explain the problem you found.

* **Where it is**: Reference the relevant file, method, or codepath.

* **Category**:

  * **Blocking**: Should be addressed before the changes are merged/deployed.
  * **Nit**: A worthwhile improvement, but does not need to block the changes from being merged/deployed.

* **Why it matters**: Explain the architecture, readability, regression, deployment, testing, or security impact.

* **Recommended fix**: Give a specific direction for how the issue should be addressed. Do not make the code change yourself.

Only report findings which you have high confidence in and can ground in the code you reviewed. Do not manufacture feedback simply to make the review look thorough.

If multiple findings are symptoms of the same underlying problem, group them together and call out the root cause instead of reporting each symptom independently.

If there are no findings for a given review pass, explicitly say so.

At the end of the review, give an overall assessment:

* **Approved**: The changes meet the architecture, readability, and regression/deployment quality bar defined by this skill. Nits may still be present, but they do not collectively represent a meaningful quality concern.

* **Changes requested**: The changes do not currently meet one or more of the quality bars defined by this skill. This may be because of one or more Blocking findings, or because the findings in aggregate indicate that the implementation needs another pass before it should be re-reviewed.

Do not determine the overall assessment mechanically from the number or category of findings. Use engineering judgment and evaluate whether the resulting code, taken as a whole, meets the standards defined by all three review passes.

**Completion Criteria**: The complete set of review findings has been handed back to the caller, grouped by pass, with clear reasoning and recommended fixes, along with an overall assessment of whether the changes are ready to merge.

## **Things to keep in mind**

* **Context Preservation**: For any non-trivial review or code deep-dive task, it is expected that you delegate to targeted subagents to ensure that your context is preserved.

* **Be rigorous**: Code which has passed your review will eventually be reviewed by the user. To reduce churn, you are expected to hold changes to a high engineering bar and be extremely thorough and rigorous throughout this process.
