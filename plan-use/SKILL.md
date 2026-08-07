---
name: plan-use
description: 'Create and structure execution plans in the global $PLANS_ROOT tree (default ~/code/plans) with clear scope, phases, milestones, acceptance criteria, target repo/worktree context, and test delta declarations before writing implementation code.'
argument-hint: 'Describe the initiative or feature you want to plan'
user-invocable: true
reusable: true
---

# Plan Use

Use this skill to draft clear, execution-ready planning documents and keep them structurally accurate as scope evolves.

Companion skills: plan-implementation for phase-by-phase execution, git-workflow for the worktree a plan targets, function-naming for naming standards, immutable-types for immutable-model constraints, decision-writing for approach-selection docs, and readme-writing for README/docs ownership and update flow.

## When To Use

- Starting a multi-step feature or refactor
- Aligning existing APIs to conventions before a major change
- Breaking large work into phases and milestones
- Creating prerequisite plans and dependency ordering
- Defining success criteria before coding
- Re-scoping phases when requirements change
- Planning public API or behavior changes that require documentation updates

## Out Of Scope

- Executing implementation phase checklists step-by-step (use plan-implementation)
- Preparing per-phase PR/commit choreography and commit messages (use plan-implementation with commit-message-writing)
- Recording day-by-day execution logs while coding (use plan-implementation implementation notes flow)

## Plan Location and Naming

Plans live in a single global tree, **outside** any repo, so all in-flight work
across every checkout is visible in one place. Each plan records the repo and
worktree it targets in its own Context section (see below) rather than living
inside that repo.

- Store plan files in `$PLANS_ROOT` (default `$HOME/code/plans`, which is
  `/workspace/plans` inside the dev container — the same directory through two
  mounts). If `$PLANS_ROOT` is unset, use that default path.
- Keep active (Drafting, Approved, or Implementing) plans at the `$PLANS_ROOT` root.
- Archive finished plans under `$PLANS_ROOT/archive/`.
- Because the tree is flat and shared across repos, prefix each filename with the
  target repo (client) name to keep names unique and greppable: `<repo>_<name>_plan.md`.
- Use concise snake_case names ending in `_plan.md` when possible.
- Keep one primary goal per plan.
- Do not create per-repo `plans/`, `.plans/`, or `.claude/plans/` directories; those are legacy locations.
- Add cross-links when one plan depends on another (relative paths within `$PLANS_ROOT`).

Examples:
- `$PLANS_ROOT/servers_conventions_alignment_plan.md`
- `$PLANS_ROOT/nextrec_coverage_gap_plan.md`
- `$PLANS_ROOT/archive/servers_degree_support_plan.md`

## Required Plan Sections

Include these sections in order unless there is a strong reason not to:

1. Status
2. Context (target repo/worktree/branch)
3. Goal
4. Why this comes first (optional but recommended)
5. Scope
6. Out of scope
7. Technical design details
8. Testing approach
9. Documentation approach
10. Progress checklist
11. Phases (numbered, with concrete deliverables)
12. Execution order recommendation
13. Implementation notes (append-only log; may be initialized empty during drafting)
14. Risks and mitigations (optional)
15. Acceptance criteria

## Status Section Rules

- Include an explicit section named "Status" at the top of every plan, immediately before "Goal".
- Allowed values are exactly:
	- Drafting
	- Approved
	- Implementing
	- Done
	- Rejected
- Lifecycle rules:
	- Default new-plan state is Drafting.
	- When a draft is accepted and ready to execute, update status to Approved.
	- Approved indicates implementation can start without further design edits unless scope changes.
	- When implementation begins, update status to Implementing as the first implementation step.
	- When implementation completes successfully, update status to Done and move the plan to `$PLANS_ROOT/archive/`.
	- If the plan is declined, update status to Rejected and move the plan to `$PLANS_ROOT/archive/`.
- Active plans in `$PLANS_ROOT` should use Drafting, Approved, or Implementing only.
- Archived plans in `$PLANS_ROOT/archive/` must use Done or Rejected only.
- Update status whenever checklist state or plan lifecycle changes.
- For plans that remain in Drafting, include at least 1-3 concrete draft-improvement suggestions in planning updates.

## Context Section Rules

Because plans now live outside the repos, every plan must state where its work
lands. Include a `Context` section immediately after `Status` and before `Goal`.

- Fields:
	- `Repo:` (required) — the target client/repo checkout name under `~/code/<repo>` (for example `servers`, `nextrec`, `utilities`). This is also the filename prefix.
	- `Worktree:` (include by default) — the worktree name under `$WORKTREE_ROOT/<repo>/<name>`, or `main` for the primary checkout. Omit only when the target worktree is genuinely undecided, and say so explicitly (for example `- Worktree: TBD (decide at implementation)`).
	- `Branch:` (include by default) — the git branch the worktree uses. Same TBD rule as Worktree.
- Keep `Repo` accurate at all times; it is how next-work-selection groups cross-repo plans and how plan-implementation finds the checkout.
- The `Worktree`/`Branch` pair links the plan to the shared worktree tree owned by git-workflow. When implementation starts, plan-implementation resolves or creates this worktree via git-workflow before coding.
- If a plan targets more than one repo, keep one primary `Repo` for the filename prefix and list the others as `Also touches:` lines in Context, with a note on ordering.

Example:

```md
## Status
Approved

## Context
- Repo: servers
- Worktree: ai-sandbox-docker-broker
- Branch: luke/ai-sandbox-docker-broker
```

## Technical Design Details Rules

- Include an explicit section named "Technical design details" for any plan that changes code.
- This section should be specific enough that implementation can begin without rediscovering major design choices.
- Cover at least:
	- Canonical types/data models and invariants.
	- API signatures for new or changed public methods.
	- Module/file touchpoints (where each change will occur).
	- Error and validation semantics (what raises, accepted forms).
	- Compatibility and migration notes (what breaks, what is removed, and when).
- Prefer explicit examples for ambiguous contracts (for example input and output forms).
- If design uncertainty remains after this section, create/link a decision doc before coding.

### Technical depth requirements

- For non-trivial behavior changes, include light pseudocode representations of core algorithms.
- For new apis, include sample calling code that demonstrates usage.
- For changes that transform a declared input into a generated or derived artifact (code generation, templates/scaffolding, schema-to-file, migrations, serialization), include a worked example: one representative real input and the resulting output artifact(s), shown as before/after diffs. Prefer an existing entity from the codebase over an invented one.

### Diagram requirements

- For non-trivial class relationships, calling sequences, etc. include diagrams that visually represent the logic or relationships.
- Mermaid diagrams and/or ascii art can be used as appropriate.
- Keep diagrams small and focused; brevity and broad understanding are more important than showing every possible detail.

### Worked example requirements

- When a plan introduces or changes an input->output transformation, include a "Worked example" showing the end-to-end result on one concrete case, not just the mechanism.
- Show every artifact the change touches for that case (the edited source, each generated file, and any file the generation removes/rewrites).
- Use a real, named entity from the repo so reviewers can diff against what exists today.
- Call out anything the example reveals that the prose didn't (ordering, escaping, comment/metadata flow, idempotency).
- Keep it to a single representative case; add a second only when it exercises a materially different path (for example multi-entry vs. single-entry).

## Testing Approach Section Rules

- Include an explicit section named "Testing approach" in each plan.
- Describe intended coverage by type (for example unit, integration, regression).
- Identify key test targets and critical edge cases.
- Describe mocking/fixture strategy and whether shared fixtures are needed.
- Include how tests will be run and what constitutes passing validation.
- Include expected test delta classification: new tests, updated tests, both, or no test delta.
- If no test changes are expected, state that explicitly with rationale under a "No test delta rationale" note.

## Documentation Approach Section Rules

- Include an explicit section named "Documentation approach" for plans that change public APIs, parsing behavior, validation behavior, examples, or user-facing workflows.
- Identify documentation touchpoints directly (for example README.md, docs/README.md, docs/api-overview.md, and affected guides).
- Include expected docs delta classification: README updates, docs updates, both, or no docs delta.
- If no documentation changes are expected, state that explicitly with rationale under a "No docs delta rationale" note.
- Include how documentation changes will be validated (for example example snippets updated, link paths checked, terminology consistency verified).

## Progress Checklist Rules

- Use markdown checkboxes.
- Keep items outcome-based, not vague activity labels.
- Include both phase-level and milestone-level checks when relevant.
- Keep checklist wording stable so it can be updated over time.
- Keep checklist status up to date as work progresses.
- Mark checklist items complete immediately when the corresponding outcome is met.
- During implementation, checklist completion is performed via plan-implementation.

Checklist style example:
- [ ] Phase 0: Inventory complete
- [ ] Phase 1: Canonical names selected
- [ ] Milestone A complete

## Phase Design Rules

- Each phase should answer: what changes, where, and how success is verified.
- Prefer small, testable phase boundaries.
- Put compatibility and migration phases before cleanup/removal phases.
- Explicitly call out docs and tests phases.
- Include technical deliverables in each phase (for example signature updates, type migrations, parser behavior).
- For model/API phases, list target files/modules directly.
- Define intended PR boundaries by phase so plan-implementation can prepare one commit message per PR boundary.

## Implementation Notes Section Rules

- Include an explicit `Implementation notes` section in every active plan.
- During drafting, initialize with a placeholder bullet such as `- No implementation notes yet.`
- During execution, plan-implementation appends dated, phase-scoped notes.
- Keep this section append-only except for factual corrections.

## Naming and API Planning Rules

- Use canonical names in new plan sections.
- If aliases are needed, mark them as compatibility aliases with retirement intent.
- Avoid planning parallel permanent APIs that do the same thing.
- For naming decisions, load function-naming.

## Pre-Implementation Dependency Rules

- If a conventions or migration prerequisite exists, state it near the top as Prerequisite.
- Link dependent plans directly.
- Do not start downstream feature phases until prerequisite checklist gates are met.
- If a plan depends on unresolved approach choices, create a decision document in `$PLANS_ROOT/decisions/` as a Markdown file (repo-prefixed like plans) and link it from the plan.

## Plan Review Checklist

- Plan lives in `$PLANS_ROOT` (not inside a repo) and is filename-prefixed with its target repo.
- Context section exists (immediately after Status) with an accurate `Repo`, and `Worktree`/`Branch` set or explicitly marked TBD.
- Status section exists at the top and reflects current lifecycle state.
- Status value is one of: Drafting, Approved, Implementing, Done, Rejected.
- Archived plans explicitly show Done or Rejected status.
- Sections are complete and in logical order.
- Scope and out-of-scope are explicit.
- Technical design details are explicit, concrete, and implementation-ready.
- Technical design details include implementation pseudocode for non-trivial logic.
- Technical design details include usage pseudocode that demonstrates intended capabilities.
- Technical design details include Mermaid/ASCII diagrams when they materially improve clarity, or a rationale for omission.
- Plans that generate or derive files include a concrete worked example (real input + resulting artifacts as before/after), or a rationale for omission.
- The worked example covers every artifact the change touches for that case, including removals.
- Testing approach section is explicit and actionable.
- Testing approach explicitly states expected test delta classification.
- Any "no test delta" claim includes rationale and is consistent with the planned code changes.
- Documentation approach section is explicit and actionable when user-facing behavior or APIs change.
- Documentation approach explicitly states expected docs delta classification.
- Any "no docs delta" claim includes rationale and is consistent with the planned code changes.
- Checklist is actionable and measurable.
- Checklist status is current and reflects real execution progress.
- Phases map to concrete files/modules.
- Test and documentation work are included.
- Acceptance criteria are unambiguous.
- Acceptance criteria include test validation evidence expectations (focused and full test runs, or justified exception).
- Acceptance criteria include documentation validation expectations (updated references/examples or justified exception).
- Dependencies and prerequisites are linked.
- Finished plans are moved to `$PLANS_ROOT/archive/`.

## Update Procedure for Existing Plans

1. Preserve plan intent and existing completed checklist items.
2. Apply minimal edits for consistency with current conventions.
3. Ensure the top Status section exists and is accurate.
4. Update all internal references after renames.
5. Update testing and documentation approach if scope, risk, or validation strategy has changed.
6. Update implementation pseudocode, usage pseudocode, and diagrams when behavior or architecture changes.
7. Update checklist states to reflect the current project state.
8. Re-read full plan to ensure no stale names remain.
9. Note major deltas in commit/PR summary.

## Plan Completion and Archiving

- Treat a plan as finished when acceptance criteria are met and the progress checklist is complete.
- Before archiving a completed implementation, update the plan's top Status section to Done.
- If a plan is not being pursued, update status to Rejected before archiving.
- Add a brief completion note before archiving when useful (for example: completed date and any key follow-up links).
- Move the plan from `$PLANS_ROOT` to `$PLANS_ROOT/archive/` (keeping the repo-prefixed filename).
- Update references from active plans so links point to the archived file path.
- Keep archived plans as historical records and avoid substantive rewrites after archival.
- If implementation notes exist, preserve them verbatim as the execution record.

## Drafting Improvement Guidance

- When a plan is still Drafting, provide concrete suggestions to improve execution readiness.
- Suggest improvements in these categories when applicable:
	- missing API signatures/contracts,
	- unclear validation/test strategy,
	- missing docs delta classification,
	- ambiguous phase boundaries or dependencies.
- Keep suggestions concise and actionable (1-3 items by default).
