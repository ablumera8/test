# Codebase Conventions

> **Note:** Conventions below are based on the PRD (`docs/prds/supabase-auth.md`). Re-run `/init-dev` after scaffolding to verify against actual code.

## Project Structure

```
src/
  components/
    auth/
      LoginForm.module.css
      LoginForm.tsx
      OAuthButtons.module.css
      OAuthButtons.tsx
      ProtectedRoute.tsx
  hooks/
    useAuth.ts
  lib/
    supabase.ts
  pages/
    Login.tsx
    Dashboard.tsx
  store/
    authStore.ts
  types/
    auth.ts
  App.tsx
  main.tsx
```

## Architecture Pattern

Flat frontend architecture — no feature modules or layered slices. Components are grouped by domain (`auth/`) under `src/components/`. Pages are top-level route components in `src/pages/`.

Import rules:
- Components import from `lib/`, `store/`, `hooks/`, `types/`
- Hooks import from `lib/`, `store/`, `types/`
- Stores import from `lib/`, `types/`
- `lib/` has no internal imports (only external packages)
- Pages import from `components/`, `hooks/`, `store/`

## Naming Conventions

- Files: PascalCase for components (`LoginForm.tsx`), camelCase for utilities (`authStore.ts`, `useAuth.ts`)
- CSS Modules: PascalCase matching component name (`LoginForm.module.css`)
- Components: PascalCase exports (`export function LoginForm`)
- Hooks: camelCase with `use` prefix (`useAuth`)
- Stores: camelCase with `Store` suffix (`authStore`)
- Types/Interfaces: PascalCase (`AuthState`, `User`)
- Directories: kebab-case for feature groups (`auth/`)

## TypeScript Config

- Strict mode enabled
- Path aliases: `@/` → `src/` (if configured)
- Target: ES2020+ (Vite handles bundling)

## Commands

```bash
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Production build
npm run preview      # Preview production build
npm run lint         # Run linter (if configured)
npx tsc --noEmit     # Typecheck without emitting
```

## Environment Variables

```bash
VITE_SUPABASE_URL=https://<project>.supabase.co
VITE_SUPABASE_ANON_KEY=<anon-key>
```

- Stored in `.env.local` (gitignored)
- Template in `.env.local.example`
- Access via `import.meta.env.VITE_*`

## Before Modifying

1. AST `map` — verify current project structure before adding files
2. AST `conventions` — check naming patterns before creating new symbols
3. LSP `workspaceSymbol` — verify no naming conflicts for new exports
4. `documentSymbol` on files being changed — understand full structure
