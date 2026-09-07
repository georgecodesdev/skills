---
name: review-changes
description: >-
  Reviews a code change using two rigorous, read-only passes: readability/maintainability and regression/deployment risk. Use this to get high-quality feedback on your work, or when the user says things like "can you review these changes", "will this break anything", "review this diff", or "I am about to push this code".
---

# **Review changes**

For any non-trivial code change, we want to ensure that the code meets our quality bar. To achieve this we want to run a two-pass review which is intended to emulate a senior software engineer reviewing the given changes. Specifically, the senior software engineer will be performing two "review passes" on the given changes, scoped to the following areas:

* Readability
* Regression / Deployment risk

As part of this review we need to improve abstractions, modularity, reduce Spaghetti code, improve succinctness and legibility. Each of these review passes is expected to be extremely thorough and rigorous. Measure twice, cut once.

**Terminology**: Throughout this skill, the **caller** is the party invoking the skill. The caller may be the human user directly, or another agent acting on the user's behalf. Any instruction to clarify something with, receive instructions from, report something to, or hand findings back to the user should be understood as referring to the caller unless the surrounding context explicitly means an end-user of the software being reviewed.

## **Workflow**

Run these steps in order. Note that this is a read-only operation and you are forbidden from making code changes as part of this process. Your job is to review the changes, not make code changes yourself. You are also forbidden from stashing/committing or modifying the state of the git repo(s) in any way.

### **Step 1: Determine what files are in-scope for the review**

This can be done either by doing one (or more) of the following:

* Looking at the staged changes
* Looking at the uncommitted changes
* Looking at a specific commit.

If you are working across multiple code packages (which may themselves be different git repositories) it's important that you explore every modified file across every code package. Leave no stone unturned, as missing relevant/apposite files will result in an incomplete review.

With this said, a review needs to have a specific scope in mind. If you are unclear or need clarification to align on exactly what changes you will be reviewing, it is expected that you ask the caller before proceeding. You are only allowed to exclude files from your review if they have been explicitly named.

**Completion Criteria**: A list of every modified file which will be in the scope of this review, grouped by the code package which they belong to. No files should be dropped, and excluded files need to be explicitly named.

### **Step 2: Build a complete mental model of the in-scope files for the review**

Now that you understand what files are in-scope for the review, it is important you take your time re-familiarizing yourself with the changed files by reading the files as they are (not how you remember them). You are not allowed to rely on your memory of what was changed or modified in a given file. Instead, you must re-read the file to get its accurate post-changed state.

It is also expected that you build out a "dependency tree" of the modified files, as otherwise you will not be able to construct the mental model needed to give a complete review. However, this "dependency tree" needs to be bounded/capped within the code packages you are working within. You are being asked to get a complete mental model of the files for review, not get an understanding of the entire dependency ecosystem you are working in.

**Completion Criteria**: Every file from the prior step has been read (in full), and their "dependency tree" has been constructed. You have a complete mental model of the in-scope files for the review.

### **Step 3: Readability pass**

It is expected that you internalize, and apply, the readability guidelines (outlined below) against the in-scope files.

Note on existing/reference implementations: Take a look at your modified files against any similar or existing implementations present in the codebase, and use their implementation as a reference. We should be adhering to the existing architecture and code style unless explicitly instructed not to do so.

**Readability guidelines**

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

* **Changes are consistent with the existing codebase style and architecture**:

  * Similar problems should be solved in similar ways.
  * Before introducing a new pattern, abstraction, or structure, look for the closest existing implementation in the codebase.
  * Use that implementation as the reference unless there is a clear reason not to.

* **Keep logic in the canonical layer and reuse existing helpers**:

  * Logic should live in the package/module/service which already owns the concept.
  * Do not leak feature-specific behavior into unrelated/shared codepaths.
  * If the codebase already has a canonical helper or utility for something, reuse it instead of creating another version.

* **When it makes sense, logic is co-located and not spread out**:

  * A reader should not need to jump through a bunch of unrelated files to understand one behavior.
  * Keep closely related behavior together.
  * Only split behavior across files when there is a real architectural boundary which gives us a reason to.

* **When it makes sense, co-locate types with their consumers**:

  * A type which only exists for one module/component should generally live near that module/component.
  * Move types into shared locations when they actually represent a shared contract.
  * Do not make types globally shared simply because they are types.

* **Layers of abstraction are easy to follow, with clear one-directional flow**:

  * It should be easy to follow the code from the entry point down into the implementation.
  * Avoid designs where understanding one behavior requires jumping backwards and forwards across layers.
  * Avoid hidden callbacks, indirect ownership, or abstraction layers which make it unclear where behavior actually lives.

* **Reuse existing abstractions**:

  * Before adding a new helper, service, wrapper, type, or abstraction, first check whether there is an existing abstraction which already owns the concept.
  * Prefer reusing or reasonably extending the existing abstraction.
  * We should not end up with multiple competing abstractions for the same thing.

* **Abstractions need to earn their keep**:

  * An abstraction should make the code meaningfully easier to understand, remove duplication, or give us a useful boundary.
  * Be suspicious of thin wrappers, pass-through helpers, identity functions, or abstractions which only move code somewhere else.
  * Prefer deleting an unnecessary abstraction over polishing it.

* **Control flow stays simple**:

  * Be highly suspicious of new nesting, repeated conditionals, boolean modes, nullable states, and one-off special cases.
  * If a change needs weird `if` statements in a bunch of random places, treat that as a design smell.
  * Repeated branching around the same concept may mean the implementation needs to be reframed.

* **Files/modules/functions remain focused**:

  * A change should not turn an existing cohesive file or module into a dumping ground for unrelated behavior.
  * If a file is becoming difficult to scan or is taking on multiple responsibilities, consider whether the change belongs somewhere else.
  * Decompose code when doing so creates clearer conceptual boundaries, not just to make files shorter.

* **Names clearly communicate intent**:

  * Names should make it obvious what something represents or what a function does.
  * Avoid vague names when there is a more specific domain name available.
  * A reader should not need to inspect the implementation to understand what a variable, type, or method is supposed to represent.

* **Types make the implementation clearer, not harder to follow**:

  * Be suspicious of unnecessary `any`, `unknown`, casts, optional fields, nullable values, or loosely-shaped objects.
  * If downstream code repeatedly needs to check or rediscover an invariant, consider whether the type boundary should make that invariant explicit.
  * Prefer types which communicate the real shape and constraints of the data.

* **Do not duplicate concepts**:

  * Look beyond literal copy/paste duplication.
  * If the same business rule, decision, or behavior is being implemented in multiple places, there should usually be one canonical source of truth.
  * Avoid creating near-duplicate helpers which represent the same underlying concept.

* **Do not add incidental complexity unless the problem actually requires it**:

  * Every new helper, abstraction, mode, branch, state, or layer is another concept which somebody needs to understand.
  * Prefer implementations which remove moving pieces instead of reorganizing them.
  * A refactor should reduce the number of concepts a reader needs to hold in their head, not just move those concepts into different files.

* **Prefer testing behavior over implementation details**:

  * Tests should assert the externally meaningful behavior of the code, not mirror its internal structure.
  * Be suspicious of tests which break simply because a helper was renamed, extracted, or reorganized while behavior stayed the same.
  * A refactor which preserves behavior should generally not require rewriting a large portion of the test suite.

* **Tests themselves should remain simple and maintainable**:

  * Avoid bespoke test harnesses, excessive setup, or complicated helpers for simple behavior.
  * Reuse the existing fixtures/helpers/testing patterns in the codebase when they already solve the problem.

* **Tests do not over-mock**:

  * Be suspicious of tests which require large amounts of mock setup to exercise simple behavior.
  * We do not want tests which only prove that our mocks behave the way we configured them.

**Refactoring north star**: The clearest expression of true understanding is being able to represent a complex problem in a simple, and easy-to-follow way.

**Completion Criteria**: A list of findings (which may be empty). Each finding in the list needs to be associated with one of the above readability guidelines with a recommended fix.

### **Step 4: Regression / Deployment risk**

Unless otherwise instructed by the caller, assume that the existing code is running without issue in production. We want to reason from the diff outward and determine whether the proposed changes could regress existing behavior, introduce instability, or make the deployment unsafe.

For every meaningful change, ask: what behavior existed before, what behavior exists after, and what else depends on the behavior which changed?

It is expected that you internalize, and apply the regression/deployment risk guidelines outlined below:

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

**Completion Criteria**: A list of findings (which may be empty). Each finding in the list needs to be associated with one of the above regression / deployment risk guidelines with a recommended fix.

### **Step 5: Hand-off findings**

Now that both review passes are complete, hand the findings back to the caller in a clear and actionable way.

Findings should be grouped by review pass:

* Readability
* Regression / Deployment risk

Within each pass, findings should be ordered from highest to lowest impact. Do not bury meaningful issues underneath low-value cleanup.

Each finding must include:

* **What the issue is**: Clearly explain the problem you found.
* **Where it is**: Reference the relevant file, method, or codepath.
* **Category**:

  * **Blocking**: Should be addressed before the changes are merged/deployed.
  * **Nit**: A worthwhile improvement, but does not need to block the changes from being merged/deployed.
* **Why it matters**: Explain the readability, regression, deployment, testing, or security impact.
* **Recommended fix**: Give a specific direction for how the issue should be addressed. Do not make the code change yourself.

Only report findings which you have high confidence in and can ground in the code you reviewed. Do not manufacture feedback simply to make the review look thorough.

If multiple findings are symptoms of the same underlying design problem, group them together and call out the root cause instead of reporting each symptom independently.

If there are no findings for a given review pass, explicitly say so.

At the end of the review, give an overall assessment:

* **Approved**: The changes meet the readability and regression/deployment quality bar defined by this skill. Nits may still be present, but they do not collectively represent a meaningful quality concern.

* **Changes requested**: The changes do not currently meet the readability and/or regression/deployment quality bar defined by this skill. This may be because of one or more Blocking findings, or because the findings in aggregate indicate that the implementation needs another pass before it should be re-reviewed.

Do not determine the overall assessment mechanically from the number or category of findings. Use engineering judgment and evaluate whether the resulting code, taken as a whole, meets the standards defined by the two review passes.

**Completion Criteria**: The complete set of review findings has been handed back to the caller, grouped by pass, with clear reasoning and recommended fixes, along with an overall assessment of whether the changes are ready to merge.

## **Things to keep in mind**

* **Context Preservation**: For any non-trivial review, or code deep-dive task it is expected that you delegate to targeted subagents to ensure that your context is preserved.
* **Be rigorous**: Code which has passed your review will eventually be reviewed by the user. To reduce churn, you are expected to hold changes to a high engineering bar and be extremely thorough and rigorous throughout this process.
