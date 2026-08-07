---
name: next-work-selection
description: 'Prioritize and select the next plan to work on from the global $PLANS_ROOT tree (default ~/code/plans) by ranking active plans across repos by status, dependencies, remaining checklist items, and planning hygiene tasks.'
argument-hint: 'Describe the current state of your plans or ask for a recommendation'
user-invocable: true
reusable: true
---

# Next Work Selection

Use this skill to choose the best next implementation target from the global `$PLANS_ROOT` tree (default `$HOME/code/plans`, i.e. `/workspace/plans` in the dev container). Plans across all repos share this tree; each plan's `Context` section names its target repo.

Companion skills: plan-use for plan authoring/re-scoping, plan-implementation for in-flight status/checklist updates, and decision-writing when dependency order is blocked by unresolved approach choices.

## When To Use

- User asks "what's next" or "what should we do next"
- User asks to prioritize or rank plans in `$PLANS_ROOT`
- User asks for execution order across multiple active plans
- User asks which plan to start first after recent completions

## Out Of Scope

- Implementing features directly without user confirmation
- Writing full plan contents from scratch (use plan-use)
- Making architecture decisions that require a decision doc (use decision-writing)

## Required Workflow

1. Gather active plan state
- Inspect all root `$PLANS_ROOT/*.md` files.
- Read each plan's Status and Context sections.
- Group candidates by `Context.Repo`. If the user asks about a specific repo (or the current client is set), scope to that repo but still surface high-priority cross-repo items.
- Count open and completed checklist items.

2. Apply prioritization gates
- Exclude plans already marked Done/Complete.
- Prefer Implementing plans over Approved plans over Drafting plans.
- Prefer plans with low remaining checklist count when closeout is cheap.
- Prefer plans explicitly marked "Why this comes first" when present.
- Respect dependency order from cross-plan references and execution-order sections.

3. Identify blocking and hygiene work
- Flag completed plans that still live in the `$PLANS_ROOT` root and recommend archiving.
- Flag plans missing a `Context` section or an accurate `Repo`, and plans still living in legacy per-repo locations (`<repo>/plans`, `<repo>/.plans`, `.claude/plans`) that should migrate to `$PLANS_ROOT`.
- Flag status drift where a plan is Drafting but substantial implementation already landed.
- Flag unresolved dependency gates before recommending downstream work.

4. Produce a ranked recommendation
- Return a numbered list of next candidates with short rationale.
- Include one concrete immediate next action for the top candidate.
- Call out optional alternatives when multiple paths are valid.

## Prioritization Rules

- Rule 1: Finish nearly complete active work before starting broad new initiatives.
- Rule 2: Prioritize by lifecycle readiness: Implementing > Approved > Drafting.
- Rule 3: Resolve foundation plans that reduce cross-module inconsistency before feature expansion.
- Rule 4: Prefer plans that unblock multiple downstream plans.
- Rule 5: Defer large speculative plans when smaller high-leverage plans are available.
- Rule 6: Keep plan lifecycle clean (Done plans archived, active plans in root).

## Output Format

- Top recommendation first with one-sentence reason.
- Follow with 2-5 ranked options.
- For any Drafting option listed, include 1-3 concise suggestions to improve draft readiness.
- End with a proposed "start now" action that can be executed immediately.

## Review Checklist

- Root `$PLANS_ROOT` status was inspected (across repos) before ranking.
- Dependency and execution-order notes were considered.
- Recommendation includes rationale, not just file ordering.
- Completed-plan archiving hygiene is addressed when needed.
- Output includes one concrete immediate next step.
