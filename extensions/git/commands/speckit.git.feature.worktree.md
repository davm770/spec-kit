---
description: "Create a feature branch AND a sibling git worktree in one step"
---

# Create Feature Branch + Worktree

Like `__SPECKIT_COMMAND_GIT_FEATURE__`, but creates a **new git worktree** in addition to the new branch. The current working tree is left on its current branch; the new branch is checked out into a separate directory.

**Default worktree location**: `../<repo-name>-<branch-name>` (sibling of the repo root). Override with `--worktree-path <path>` or by setting `WORKTREE_PATH` in the environment.

Why a sibling dir: launching a separate agent session (e.g. Claude Code) inside the worktree gives it a clean, isolated project root with no overlap with the main repo's `.specify/` / `.claude/` / file watchers.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Environment Variable Overrides

- `GIT_BRANCH_NAME` — exact branch name, bypasses prefix/suffix generation (same semantics as `__SPECKIT_COMMAND_GIT_FEATURE__`).
- `WORKTREE_PATH` — exact worktree path, bypasses the default sibling-dir convention.

## Prerequisites

- Verify Git is available: `git rev-parse --is-inside-work-tree 2>/dev/null`
- Git is **required** for this command (worktrees only exist in git). If Git is not available, fail with a clear error — do not silently fall back.

## Branch Numbering Mode

Same as `__SPECKIT_COMMAND_GIT_FEATURE__`:

1. Check `.specify/extensions/git/git-config.yml` for `branch_numbering` value
2. Check `.specify/init-options.json` for `branch_numbering` value (backward compatibility)
3. Default to `sequential` if neither exists

## Execution

Generate a concise short name (2-4 words) for the branch:
- Analyze the feature description and extract the most meaningful keywords
- Use action-noun format when possible (e.g., "add-user-auth", "fix-payment-bug")
- Preserve technical terms and acronyms (OAuth2, API, JWT, etc.)

Run the worktree-variant script:

- **Bash**: `.specify/extensions/git/scripts/bash/create-new-feature-worktree.sh --json --short-name "<short-name>" "<feature description>"`
- **Bash (timestamp)**: `.specify/extensions/git/scripts/bash/create-new-feature-worktree.sh --json --timestamp --short-name "<short-name>" "<feature description>"`
- **Bash (custom path)**: `.specify/extensions/git/scripts/bash/create-new-feature-worktree.sh --json --short-name "<short-name>" --worktree-path "<path>" "<feature description>"`

**IMPORTANT**:
- Do NOT pass `--number` — the script determines the correct next number automatically
- Always include `--json` so output can be parsed reliably
- Run the script exactly once per feature
- The JSON output will contain `BRANCH_NAME`, `FEATURE_NUM`, and `WORKTREE_PATH`

## Output

The script outputs JSON with:
- `BRANCH_NAME`: The new branch name (e.g., `003-user-auth` or `20260319-143022-user-auth`)
- `FEATURE_NUM`: The numeric or timestamp prefix used
- `WORKTREE_PATH`: Absolute path of the new worktree directory

After running, report all three to the user, plus the `cd <WORKTREE_PATH>` command to enter the worktree.

## Cleanup (for reference, not part of this command)

To remove the worktree later:

```
git worktree remove <WORKTREE_PATH>
git branch -d <BRANCH_NAME>   # or -D to force
```
