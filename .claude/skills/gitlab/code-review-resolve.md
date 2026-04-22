# Resolve CR

Address code review feedback on a merge request.

## Steps

### Step 1: Resolve MR number

- If provided as argument → use it
- If not provided → detect from current branch: `glab mr view --output json 2>/dev/null`
- If no MR found → prompt user and **wait for response**

### Step 2: Fetch MR review comments

```bash
glab mr view <mr-number> --output json
glab api projects/:id/merge_requests/<mr-number>/notes --output json
```

Fetch discussion threads to filter out already-resolved ones:

```bash
glab api projects/:id/merge_requests/<mr-number>/discussions --output json
```

Only present **unresolved** top-level reviewer comments. Skip already-resolved discussions and self-authored reply confirmations.

Identify automated reviewers (e.g., bots) vs human reviewers by checking the author username.

### Step 3: Evaluate each comment

Determine if comment should be fixed or skipped based on:

- Does it affect correctness, security, or architecture?
- Does it align with project coding conventions?
- Is it a minor style preference with no impact? → skip

### Step 4: Present list to developer

Group comments by source: human reviewers first, then automated. Automated reviewer comments are typically lower priority.

Display all comments in this format:

```
Human reviewers:

#1 [FIX] Use shared Button component
   Reviewer asks: Replace custom button with shared Button
   Reason: Aligns with reusability standards, shared components exist

#2 [FIX] Add error handling
   Reviewer asks: Handle API error case in fetchUser()
   Reason: Missing error handling could cause runtime crashes

Automated reviewers:

#3 [SKIP] Rename variable
   Reviewer asks: Rename 'x' to more descriptive name
   Reason: Minor naming preference, doesn't affect code quality
```

If there are no comments from one group, omit that section header.

**Wait for developer response.** Developer specifies which to fix or skip (e.g., "fix 1,3 skip 2" or "fix all" or "skip 1").

### Step 5: Implement fixes

**Confirmation gate:** Show the developer what changes you plan to make. If `-y` → proceed. Otherwise → wait for approval before committing or pushing.

### Step 6: Post replies and resolve discussions

After fixes are committed, reply to every comment — both human and automated. Never resolve a discussion without a reply.

#### Discussion resolution rules

- **Human reviewer discussions** → reply only, do **NOT** resolve. The reviewer resolves their own discussions.
- **Automated reviewer discussions** (bots) → reply **and** resolve.

| Action   | Reply format                                                            |
| -------- | ----------------------------------------------------------------------- |
| **FIX**  | "Applied — fixed in `<sha>`: <what changed>"                           |
| **SKIP** | "Skipped — <reason why the suggestion doesn't apply or is unnecessary>" |

Reply to comment via API:

```bash
glab api projects/:id/merge_requests/<mr-number>/discussions/<discussion-id>/notes -f body="<reply message>"
```

Resolve automated discussion via API:

```bash
glab api -X PUT projects/:id/merge_requests/<mr-number>/discussions/<discussion-id> -f resolved=true
```

### Step 7: Output summary

- Fixed: list with changes made and commit SHAs
- Skipped: list with reasons posted
- Resolved: count of automated discussions resolved
