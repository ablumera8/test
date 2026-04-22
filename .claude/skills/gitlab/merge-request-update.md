# Update MR

Update an existing MR's title and description from GitLab issue.

## Steps

### Step 1: Resolve MR number

- If provided as argument → use it
- If not provided → detect from current branch: `glab mr view --output json 2>/dev/null`
- If no MR found → prompt user and **wait for response**

### Step 2: Resolve issue number

Extract from MR branch name (see [references/rules.md](./references/rules.md) > Resolving Issue Number).

### Step 3: Fetch GitLab issue

```bash
glab issue view <number>
```

Get issue title, description, and labels.

### Step 4: Analyze current branch

!`git log main..HEAD --oneline`
!`git diff main..HEAD --stat`

### Step 5: Generate MR title

See [references/rules.md](./references/rules.md) > MR Title Format.

### Step 6: Generate MR description

If the project has a merge request template (e.g., `.gitlab/merge_request_templates/default.md`), use it as the structure. Otherwise, use this format:

```markdown
## Summary
<1-3 bullet points describing the changes>

Closes #<issue-number>

## Test plan
- [ ] <testing step>
```

Fill in each section based on the GitLab issue and branch analysis.

### Step 7: Confirm and update MR

**Confirmation gate:** Show the new MR title and summary of changes. If `-y` → proceed. Otherwise → ask "Update MR !<number>?" and wait.

```bash
glab mr update <mr-number> --title "<title>" --description "<description>"
```

### Step 8: Output the MR URL

Show the MR URL returned by `glab mr update` to the developer.
