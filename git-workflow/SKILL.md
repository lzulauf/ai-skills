---
name: git-workflow
description: Safely create, amend, rebase, cherry-pick, or otherwise produce Git commits and create, inspect, or remove Git worktrees. Use whenever an agent is asked to commit changes, prepare commits, manage branches through worktrees, create a workspace for a branch, or clean up a worktree. Preserves the human's configured Git identity, prohibits Git configuration changes and agent attribution, and keeps worktrees under the primary checkout's worktrees directory.
---

# Git Workflow

Follow the repository's commit attribution and worktree-location invariants. Commit planning and grouping remain the responsibility of `everlaw-commit-planner`.

## Identity and Configuration

- Treat Git configuration as user-owned, immutable state.
- Permit read-only inspection such as `git config --get user.name` and `git config --get user.email`.
- Never use a mutating `git config` command at any scope.
- Never supply `--author`, `--reset-author`, `git -c user.name`, `git -c user.email`, or `GIT_AUTHOR_*`/`GIT_COMMITTER_*` environment overrides.
- Before creating a commit, inspect the effective identity with:

  ```bash
  git var GIT_AUTHOR_IDENT
  git var GIT_COMMITTER_IDENT
  ```

- If either identity is missing or does not appear to represent the user, stop and ask the user to correct their environment. Do not correct it for them.
- Preserve the original human author when replaying existing commits. Do not rewrite historical authorship unless the user explicitly requests it.

## Creating Commits

Ask for permission before committing unless committing is already authorized. Authorization is standing, not per-commit: once the user grants it — in the current session or through a durable project/user instruction — stage and commit as needed without asking again for each individual commit. Pushing or otherwise publishing commits to a remote always requires an explicit request. Every safeguard in this document still applies.

Whenever you create one or more commits, state clearly in your response that you committed and what each commit contained.

Construct commits so that:

- Each commit is a single logical change, distinct from the others.
- Commits are ordered so each follows logically from the previous one.
- The repository is left in a good state at every commit — it builds and its tests could be run between any two commits (build-safe ordering).

Detailed grouping and ordering of a working set is planned by `everlaw-commit-planner`; these invariants hold regardless of who plans the commits.

1. Inspect the status and relevant staged and unstaged diffs.
2. Stage only the changes intended for the commit.
3. Write the message around the resulting behavior:
   - State what changed in the subject.
   - Explain why in the body when the reason is not evident.
   - Omit implementation-process details that do not help a reviewer understand the change.
4. Do not add `Co-authored-by`, agent attribution, generation notices, or references to the agent or tools used.
5. Create the commit without author or committer overrides.
6. Verify its attribution and message before reporting success:

   ```bash
   git show -s --format='Author: %an <%ae>%nCommitter: %cn <%ce>%n%n%B' HEAD
   ```

Apply the same safeguards to amend, rebase, cherry-pick, merge, and revert operations that create commits.

## Managing Worktrees

Treat "workspace" as a Git worktree. Locate the primary checkout using the first `worktree` entry from:

```bash
git worktree list --porcelain
```

Place every added worktree at:

```text
<primary-checkout>/worktrees/<branch>
```

Preserve slashes in branch names as nested directories. Always pass `--relative-paths` so the linkage files between the primary checkout and the worktree are relative, keeping the worktree resolvable when the repository is reached through different mount points (e.g. a VM path versus the host path). This requires git ≥ 2.48; if `git worktree add --relative-paths -h` does not list the flag, omit it and proceed with absolute paths. `--relative-paths` changes no config — the repo-local `extensions.relativeWorktrees` marker it relies on is set automatically and is not a `git config` mutation you perform.

For an existing branch:

```bash
git worktree add --relative-paths "<primary-checkout>/worktrees/<branch>" "<branch>"
```

For a new branch:

```bash
git worktree add --relative-paths -b "<branch>" "<primary-checkout>/worktrees/<branch>" "<base>"
```

Before adding a worktree, confirm that the branch is not already checked out and the destination does not contain unrelated files. Do not use an agent-specific or system-temporary location as a fallback.

Remove worktrees through Git:

```bash
git worktree remove "<primary-checkout>/worktrees/<branch>"
```

Do not manually delete a registered worktree. Do not use `--force` unless the user explicitly authorizes discarding the worktree's changes.
