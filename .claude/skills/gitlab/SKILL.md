---
name: gitlab
description: GitLab commands for issues, merge requests, code reviews, and shipping. Use when developer needs to create issues, fetch issue context (title, description, comments), create/update merge requests, address review feedback, or ship experimental changes on a GitLab project. Also use when another skill or agent needs ticket context — invoke issue fetch. Requires glab CLI.
user-invocable: true
argument-hint: "issue|merge-request|code-review|ship [action] [args] [-y]"
license: MIT
compatibility: "Requires glab CLI (GitLab CLI) installed and authenticated"
metadata:
  author: supa-magic
  version: 1.0.0
  category: development
  tags: [gitlab, merge-requests, issues, code-review, workflow]
  requires: glab
---

# /gitlab $ARGUMENTS

GitLab workflow commands using `glab` CLI.

## Sub-skills

- **issue create** — conversational issue creation from rough ideas
- **issue fetch** — fetch issue title, description, and comments as structured context. Use when any skill or agent needs ticket information (e.g., for planning, coding, or MR creation)
- **merge-request create** — create MR with description derived from the linked issue
- **merge-request update** — update existing MR title and description
- **code-review resolve** — classify and resolve review comments on an MR
- **ship** — ship experimental changes: create issue from diff, branch, commit, and open MR

## Usage

```
/gitlab
    issue create                    Conversational issue creation
    issue fetch [number]            Fetch issue title, description, and comments for context
    merge-request create            Create MR from current branch
    merge-request update [number]   Update MR title and description
    code-review resolve [number]    Resolve code review feedback
    ship                            Ship experimental changes (issue -> branch -> commit -> MR)

    -y, --yes                       Skip confirmations
```

| Argument | Format | Default | Effect |
|----------|--------|---------|--------|
| `command` | Positional (1st token) | — | Entity to operate on (`issue`, `merge-request`, `code-review`, `ship`) |
| `action` | Positional (2nd token) | — | Subcommand (e.g., `create`, `update`, `resolve`). Can be omitted when only one action exists (`code-review` defaults to `resolve`) |
| `args` | Positional (remaining tokens) | — | Passed to the subcommand (e.g., MR number) |
| `-y`, `--yes` | Flag | `false` | Skip all confirmation gates |

## Rules

See [references/rules.md](./references/rules.md) — applies to ALL gitlab operations.

## Instructions

### Step 1: Parse Arguments

Extract from `$ARGUMENTS`:

1. First non-flag token → `command` (one of: `issue`, `merge-request`, `code-review`, `ship`)
2. If `command` has subcommands: next token → `action`. If omitted and only one action exists for the command, default to it (e.g., `code-review` defaults to `resolve`)
3. Remaining non-flag tokens → passed to the subcommand as positional args (e.g., MR number)
4. `-y` or `--yes` anywhere → skip all confirmation gates

If no command is provided, list the available commands and ask the developer which one to run.

If the command or action is not recognized, show:

> Unknown command `<command> <action>`. Available commands: `issue create`, `merge-request create`, `merge-request update`, `code-review resolve`, `ship`.

### Step 2: Route to Subcommand

Read the command-specific instruction file and follow it exactly:

- **issue create** → Read [issue-create.md](./issue-create.md) and follow all steps
- **issue fetch** → Read [issue-fetch.md](./issue-fetch.md) and follow all steps
- **merge-request create** → Read [merge-request-create.md](./merge-request-create.md) and follow all steps
- **merge-request update** → Read [merge-request-update.md](./merge-request-update.md) and follow all steps
- **code-review resolve** → Read [code-review-resolve.md](./code-review-resolve.md) and follow all steps
- **ship** → Read [ship.md](./ship.md) and follow all steps

## Examples

### Example 1: Create an issue from a rough idea

User says: `/gitlab issue create` (or `/gitlab issue`)

Actions:
1. Ask developer to describe the problem or feature
2. Refine into structured issue with acceptance criteria
3. Confirm and create via `glab issue create`

Result: `Created issue #42: https://gitlab.com/org/repo/-/issues/42`

### Example 2: Create an MR

User says: `/gitlab merge-request create`

Actions:
1. Resolve issue number from branch name
2. Fetch issue details
3. Generate MR title and description
4. Confirm and create via `glab mr create`

Result: `Created MR !43: https://gitlab.com/org/repo/-/merge_requests/43`

### Example 3: Ship experimental changes

User says: `/gitlab ship`

Actions:
1. Analyze uncommitted changes on main
2. Create issue, branch, commit, and MR in sequence
3. Each step invokes the appropriate skill

Result: Issue, branch, commit, and MR created in one flow

## Troubleshooting

### Error: glab CLI not found
Cause: GitLab CLI is not installed.
Solution: Install manually: `brew install glab` (macOS), `scoop install glab` (Windows), or see https://gitlab.com/gitlab-org/cli

### Error: glab not authenticated
Cause: `glab auth status` shows not logged in.
Solution: Run `glab auth login` to authenticate.

### Error: No MR found for current branch
Cause: `merge-request update` or `code-review resolve` was used but no MR exists for the branch.
Solution: Create an MR first with `/gitlab merge-request create`.
