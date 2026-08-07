---
name: git-workflow
description: Safely create, amend, rebase, cherry-pick, or otherwise produce Git commits and create, inspect, or remove Git worktrees. Use whenever an agent is asked to commit changes, prepare commits, manage branches through worktrees, create a workspace for a branch, or clean up a worktree. Preserves the human's configured Git identity, prohibits Git configuration changes and agent attribution, and keeps worktrees in the shared `~/code/worktrees/<repo>/` tree via the `worktree` command.
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

Treat "workspace" as a Git worktree. Worktrees live in a shared tree keyed by repo and name, **outside** the primary checkout:

```text
$WORKTREE_ROOT/<repo>/<name>        # default WORKTREE_ROOT=~/code/worktrees
```

`<repo>` is the primary checkout's directory name (e.g. `servers`), and `<name>` is the worktree's short name; preserve slashes in the name as nested directories. The sentinel name `main` denotes the primary checkout itself, not a directory on disk. Never place a worktree inside the primary checkout, and never fall back to an agent-specific or system-temporary location (e.g. `.claude/worktrees/`).

### Prefer the `worktree` command

The shell environment provides a `worktree` command (alias `wt`) that owns this location convention and switches the active `(client, worktree)` context. When it is available in the shell, use it as the canonical entrypoint:

```bash
worktree                          # list worktrees for the current repo (also -l/--list)
worktree <name>                   # switch to an existing worktree (errors if missing)
worktree --add <name> [branch]    # create at $WORKTREE_ROOT/<repo>/<name>, then switch
worktree --main                   # switch back to the primary checkout (also `-`, `main`)
worktree --rm <name>              # remove a worktree via `git worktree remove`
```

`worktree --add` runs `git -C <primary> worktree add "$WORKTREE_ROOT/<repo>/<name>" [branch]` under the hood, so it always lands in the known location. Creation is explicit: a bare `worktree <name>` only switches, and errors if the worktree is missing.

### Driving Git directly

When the `worktree` function is not sourced (non-interactive shell), or you need to operate on a worktree by absolute path without switching the session's context, run the equivalent Git commands against the same location. Locate the primary checkout from the first `worktree` entry of:

```bash
git worktree list --porcelain
```

Create the worktree at `$WORKTREE_ROOT/<repo>/<name>`, making the parent directory first:

```bash
mkdir -p "$WORKTREE_ROOT/<repo>"
# existing branch:
git -C <primary> worktree add "$WORKTREE_ROOT/<repo>/<name>" "<branch>"
# new branch off a base:
git -C <primary> worktree add -b "<branch>" "$WORKTREE_ROOT/<repo>/<name>" "<base>"
```

Because both the primary checkout (`~/code/<repo>`) and the worktree tree (`~/code/worktrees/<repo>/`) live under `~/code`, the linkage resolves when the repository is reached through different mount points (e.g. a VM path versus the host path). On git ≥ 2.48 you may add `--relative-paths` to keep the linkage files relative; omit it on older git. `--relative-paths` changes no config — the repo-local `extensions.relativeWorktrees` marker it relies on is set automatically and is not a `git config` mutation you perform.

Before adding a worktree, confirm that the branch is not already checked out and the destination does not contain unrelated files.

Remove worktrees through Git (or `worktree --rm`); do not manually delete a registered worktree:

```bash
git -C <primary> worktree remove "$WORKTREE_ROOT/<repo>/<name>"
```

Do not use `--force` unless the user explicitly authorizes discarding the worktree's changes.
