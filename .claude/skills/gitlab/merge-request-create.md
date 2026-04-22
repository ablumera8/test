# Create MR

Create a new merge request with description from GitLab issue.

## Steps

### Step 1: Resolve issue number

See [references/rules.md](./references/rules.md) > Resolving Issue Number.

### Step 2: Fetch GitLab issue

```bash
glab issue view <number>
```

Get issue title, description, and labels.

### Step 3: Analyze current branch

!`git log main..HEAD --oneline`
!`git diff main..HEAD --stat`

### Step 4: Generate MR title

See [references/rules.md](./references/rules.md) > MR Title Format.

### Step 5: Generate MR description

If the project has a merge request template (e.g., `.gitlab/merge_request_templates/default.md`), use it as the structure. Otherwise, use this format:

```markdown
## Summary
<1-3 bullet points describing the changes>

Closes #<issue-number>

## Test plan
- [ ] <testing step>
```

Fill in each section based on the GitLab issue and branch analysis.

### Step 6: Confirm and create MR

**Confirmation gate:** Show the MR title, target branch, and commit count. If `-y` → proceed. Otherwise → ask "Push and create MR?" and wait.

```bash
glab mr create --title "<title>" --description "<description>"
```

### Step 7: Output the MR URL

Show the MR URL returned by `glab mr create` to the developer.
