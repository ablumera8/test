---
name: dev
description: Develops GitHub issues end-to-end following project conventions. Accepts an issue number or URL, fetches details, analyzes codebase with LSP and AST, creates a feature branch, implements the solution, runs quality checks, and commits. Use when user says "dev 42", "dev #42", or pastes a GitHub issue URL.
user-invocable: true
argument-hint: "<issue> [-y]"
metadata:
  author: test-project
  version: 1.0.0
  category: development
  tags: [github, issue, implementation, full-cycle, conventions]
---

# /dev $ARGUMENTS

Develop a GitHub issue end-to-end: fetch, analyze, branch, implement, verify, commit.

## Usage

```
/dev <issue>          Develop issue (number, #number, or full GitHub URL)
/dev <issue> -y       Skip confirmation gates
```

| Argument | Format | Default | Effect |
|----------|--------|---------|--------|
| `issue` | `42`, `#42`, or full GitHub URL | — | GitHub issue to implement |
| `-y`, `--yes` | Flag | `false` | Skip confirmation gates |

## Rules

Before writing ANY code, read and internalize these references:

- [codebase-conventions.md](./references/codebase-conventions.md) — project structure, naming, TypeScript config, commands
- [state-boundaries.md](./references/state-boundaries.md) — Zustand stores, auth state, session management
- [styling-conventions.md](./references/styling-conventions.md) — CSS Modules patterns, file structure
- [supabase-conventions.md](./references/supabase-conventions.md) — Supabase client setup, auth methods, API patterns

CRITICAL: Always use LSP and AST tools to understand existing code before modifying or extending it. Never guess at types, signatures, or patterns — verify with `hover`, `findReferences`, `documentSymbol`, and AST search.

### Mandatory LSP Checks

Before modifying ANY existing file:
1. `documentSymbol` — understand the full file structure before making changes
2. `hover` on symbols being used or extended — verify types and signatures match expectations
3. `findReferences` on any export being changed — assess downstream impact
4. `goToDefinition` on unfamiliar imports — trace to source and understand the contract

Before creating ANY new file:
1. AST `search` for similar patterns — ensure no duplication of existing functionality
2. AST `map` for the target directory — understand existing structure and naming
3. LSP `workspaceSymbol` for the new symbol name — avoid naming conflicts

During implementation:
1. `hover` on every imported symbol in new code — confirm types align
2. `findReferences` before renaming or removing any shared export
3. `incomingCalls` when modifying function signatures — trace all callers
4. `goToImplementation` when extending interfaces — verify all implementations

After implementation:
1. Run typecheck — if errors, use `hover` on flagged lines for diagnosis
2. `findReferences` on all new exports — verify they are wired correctly

## Instructions

### Step 1: Parse Arguments

Extract from `$ARGUMENTS`:

1. `issue` — first non-flag token. Accept formats:
   - `42` or `#42` → issue number
   - `https://github.com/<owner>/<repo>/issues/<number>` → extract number
2. `-y` or `--yes` → `skip_confirmations = true`
3. If no issue provided → error: "Issue required. Usage: `/dev <issue-number>`" and stop.

Repo: !`git remote get-url origin | sed 's/.*[:/]\([^/]*\/[^.]*\).*/\1/'`

### Step 2: Fetch Issue

```bash
gh issue view <number> --repo <owner/repo> --json title,body,labels,assignees,milestone
```

Parse the issue. If it fails → error: "Could not fetch issue #<number>. Verify it exists and `gh` is authenticated." and stop.

Display a summary:

> **Issue #<number>**: <title>
> <first 3 lines of body>
> **Labels**: <labels>

### Step 3: Analyze Codebase

Read all reference files from `references/` directory to internalize project conventions.

Then perform codebase analysis using LSP and AST:

1. **AST: Map the relevant area** — based on the issue, identify which areas are affected:
   - Search for related symbols: `ast-index search "<domain keyword>"`
   - Map project structure: `ast-index map`
2. **LSP: Understand existing code** — for each file that will be modified or extended:
   - `documentSymbol` to understand file structure
   - `hover` on key symbols to verify types and signatures
   - `findReferences` on symbols being extended to check impact
   - `goToDefinition` to trace imports and understand dependencies
3. **Identify the scope** — determine:
   - Which existing files need modification
   - Which new files need creation (following project conventions)
   - Which shared components/utilities to reuse

### Step 4: Plan

Present the implementation plan:

> **Plan for #<number>:**
>
> **Supabase / Data layer:**
> - <changes to client, auth, or API calls>
>
> **State management:**
> - <changes to Zustand stores>
>
> **Components:**
> - <new or modified components>
>
> **Pages / Routes:**
> - <new or modified pages and route config>
>
> **New files:**
> - <list with convention justification>
>
> **Modified files:**
> - <list with what changes>

**Confirmation gate:** If `-y` → proceed. Otherwise → present plan and wait for approval.

### Step 5: Create Branch

```bash
git checkout -b feature/<number>/<short-description>
```

### Step 6: Implement

Follow references strictly. Implementation order for this frontend-only project:

1. **Types** — define TypeScript interfaces in `src/types/`
2. **Supabase client** — auth queries, client setup in `src/lib/`
3. **State** — Zustand stores in `src/store/`
4. **Hooks** — custom hooks in `src/hooks/`
5. **Components** — reusable UI in `src/components/`
6. **Pages** — route-level views in `src/pages/`
7. **CSS Modules** — co-located styles (`.module.css` alongside components)
8. **Routing** — route config in `App.tsx`

Apply all Mandatory LSP Checks from the Rules section continuously during implementation.

**Pause for user attention when:**
- The issue is ambiguous about expected behavior
- A change would break existing references (found via LSP `findReferences`)
- A design decision has multiple valid approaches
- The scope exceeds what the issue describes

### Step 7: Verify

Run quality checks in sequence:

1. **Typecheck:**
   ```bash
   npx tsc --noEmit
   ```

2. **Lint** (if linter configured):
   ```bash
   npm run lint
   ```

3. **Build** (verify no runtime compilation errors):
   ```bash
   npm run build
   ```

If checks fail:
1. Read the error output
2. Use LSP `hover` on the failing line to understand the type mismatch
3. Fix the issue
4. Re-run the failing check
5. Repeat until all checks pass

### Step 8: Commit

Stage and commit with a conventional commit message:

```bash
git add <specific files>
git commit -m "<type>(<scope>): <description>

Closes #<number>

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

### Step 9: Output

> **Done: Issue #<number>**
>
> **Branch:** `feature/<number>/<description>`
> **Files changed:** <count>
> **Quality checks:** All passing

## Examples

### Example 1: New feature — social OAuth login

User says: `/dev 4`

Actions:
1. Fetch issue #4: "Add social OAuth login (Google + GitHub)"
2. AST search "OAuth" → no results, search "login" → finds `LoginForm.tsx`
3. LSP `documentSymbol` on `LoginForm.tsx` → understand current structure
4. Plan: create `OAuthButtons.tsx` + `OAuthButtons.module.css`, modify `Login.tsx` to compose them
5. Branch: `feature/4/social-oauth`
6. Implement: Supabase `signInWithOAuth` calls, error handling, CSS Modules styling
7. Verify: `npx tsc --noEmit` passes
8. Commit: `feat(auth): add Google and GitHub OAuth login`

### Example 2: Bug fix — session not persisting on refresh

User says: `/dev 5 -y`

Actions:
1. Fetch issue #5: "Implement protected routes and session persistence"
2. AST search "session" → finds `authStore.ts`, `useAuth.ts`
3. LSP `findReferences` on `onAuthStateChange` → trace subscription setup
4. Plan: fix `INITIAL_SESSION` handling in `useAuth` hook, add `ProtectedRoute` component
5. Branch: `feature/5/protected-routes`
6. Implement: fix session hydration, add redirect preservation
7. Verify: `npx tsc --noEmit` passes, `npm run build` succeeds
8. Commit: `fix(auth): persist session across browser refreshes`

## Troubleshooting

### Error: gh CLI not authenticated
Solution: Run `gh auth login`.

### Error: Branch already exists
Solution: Ask user whether to switch to existing branch or create with suffix.

### Error: Quality checks fail
Solution: Use LSP `hover` to diagnose errors. Fix iteratively. Never skip checks.

### Error: Supabase types outdated
Solution: Run `npx supabase gen types typescript --linked > src/types/supabase.ts` to regenerate.
