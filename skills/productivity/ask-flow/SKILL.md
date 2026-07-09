---
name: ask-flow
description: Ask which skill or flow fits the task. A router over the local Copilot skills.
disable-model-invocation: true
---

# Ask Flow

You don't remember every skill, so ask.

A **flow** is a path through the skills. Some tasks run through one main flow, some start from a specialist on-ramp, and some are standalone. This skill is the router: it chooses the path before work begins.

## First move

If the user's request already clearly names the right skill or flow, route there immediately.

If it does not, ask one question:

> What type of flow is this task?

Offer the smallest useful set of choices from the map below. Ask one question at a time; do not interview the whole task here. The completion criterion is one chosen primary flow, a one-line reason, and the next skill or mode to use.

## The main flow: idea -> implementation

Use this route when the user has an idea, feature, design, or change they want to sharpen before building.

1. **`/grill-with-docs`** - start here when the task lives in a codebase and the discussion should leave a paper trail. It runs **`/grilling`** and uses **`/domain-modeling`** to keep glossary and ADR decisions clean.
2. **`/grill-me`** - use this instead when there is no codebase, or the user wants a stateless stress-test of a plan or design. It runs **`/grilling`** without writing docs.
3. **Plan mode** - after the design is sharp enough to build, create an implementation plan before editing.
4. **Normal implementation mode** - once the plan is approved, make the code changes and validate them.

Keep the grilling and planning in one context while the idea is still being shaped. If the thread gets too full or the work needs to branch, use **`/handoff`** to write a handoff document before continuing in a fresh session.

## Crossing sessions

Use this when the task needs a fresh context window but the current conversation must be preserved.

* **`/handoff`** - compact the current conversation into a handoff document for another agent or session. Use it before clearing context, opening a fresh session, or branching off into a separate investigation. The document should reference existing artifacts instead of duplicating them and should include suggested skills for the next agent.

## Specialist on-ramps

These routes start from a specific kind of task and may later merge back into the main flow.

* **Builds are failing or slow** -> **`/build-analyzer`**. Use for build status, CI failures, compiler errors, failing pipelines, and timing questions. It is diagnostic-only first: inspect read-only evidence, report the root cause, then stop before edits or builds unless the user approves the next action.
* **PLM Tralin annotations** -> **`/plm-tralin-annotations`**. Use for adding, refreshing, reconciling, or checking Tralin annotations against a PLM RSD branch.
* **Domain language is fuzzy** -> **`/domain-modeling`**. Use when the problem is terminology, ubiquitous language, a glossary term, or an ADR-worthy decision.
* **Copilot cloud agent setup** -> **`/customize-cloud-agent`**. Use for `copilot-setup-steps`, runner setup, preinstalled tools, dependencies, or cloud-agent environment configuration.

## Skill-work routes

Use these when the task is about the skills themselves.

* **Writing or editing a skill** -> **`/writing-great-skills`**. Use for skill design, pruning, invocation choice, descriptions, routing, progressive disclosure, and failure modes.
* **Choosing between skills** -> stay in **`/ask-flow`**. Ask the routing question, pick the primary skill, and stop once the user knows where to go next.

## Standalone flows

Use these outside the main idea -> implementation route.

* **`/grilling`** - the primitive for a relentless interview. Prefer the wrappers **`/grill-me`** or **`/grill-with-docs`** unless another skill explicitly needs the primitive.
* **`/domain-modeling`** - can run alone when the words are the problem, even if no implementation is planned.
* **`/build-analyzer`** - can run alone for diagnosis; do not turn diagnosis into fixing without explicit approval.
* **`/plm-tralin-annotations`** - can run alone for branch-limited traceability work.
* **`/handoff`** - can run alone when the current thread needs to be carried into a fresh session.

## Routing rules

* Prefer the narrowest specialist skill that fits the request.
* Ask for decisions, not facts the workspace can answer.
* Route to one primary flow. Mention a later secondary skill only when the sequence matters.
* For model-invoked skills available in the current environment, invoke the skill when the route is clear.
* For user-invoked skills, tell the user which slash skill to run next; user-invoked skills are intentionally remembered by the user.
