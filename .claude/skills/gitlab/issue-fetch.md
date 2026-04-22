# Fetch Issue

Fetch a GitLab issue's title, description, and comments as structured context for other skills and agents.

### Step 1: Fetch Issue

```bash
glab issue view {issue-number} --output json
```

Alternatively, use the API for full control:

```bash
glab api projects/:id/issues/{issue-number} --output json
glab api projects/:id/issues/{issue-number}/notes --output json
```

If the command fails → error: "Could not fetch issue #{issue-number}. Verify it exists and `glab` is authenticated." and stop.

### Step 2: Format Output

Return structured context:

```markdown
# Issue #{iid}: {title}

**State:** {state}
**Labels:** {labels}

## Description

{description}

## Comments ({count})

### Comment by {author} — {created_at}

{body}

---

### Comment by {author} — {created_at}

{body}
```

If no comments → omit the Comments section entirely.

### Step 3: Output

Print the formatted context. Do not summarize or truncate — return the full content so the consuming skill has complete information.
