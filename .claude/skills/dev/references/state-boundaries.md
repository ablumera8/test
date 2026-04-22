# State Management

> **Note:** Conventions below are based on the PRD (`docs/prds/supabase-auth.md`). Re-run `/init-dev` after scaffolding to verify against actual code.

## Client State (Zustand)

Auth state is managed with Zustand v5 in `src/store/authStore.ts`.

### Store Shape

```typescript
import { create } from 'zustand'
import type { User, Session } from '@supabase/supabase-js'

interface AuthState {
  user: User | null
  session: Session | null
  loading: boolean
  setUser: (user: User | null) => void
  setSession: (session: Session | null) => void
  setLoading: (loading: boolean) => void
  signOut: () => Promise<void>
}
```

### Pattern

- Stores use `create` from Zustand with inline type parameter
- Actions are defined inside the store (no separate action creators)
- Async actions (like `signOut`) call Supabase client directly

### What Belongs in Zustand

- Auth state (user, session, loading)
- Global UI state (theme, sidebar toggle)

### What Does NOT Belong in Zustand

- Form state (local component state)
- Route params (use React Router hooks)
- Server data (if adding data fetching later, use React Query or similar)

## Auth Flow

1. `useAuth` hook subscribes to `supabase.auth.onAuthStateChange` on mount
2. Events update Zustand store: `SIGNED_IN`, `SIGNED_OUT`, `TOKEN_REFRESHED`
3. Initial `loading: true` → set to `false` after session check completes
4. Components read from store via `useAuth` hook or direct store import

## Navigation State

- React Router v7 manages URL state
- Protected routes redirect to `/login?redirect=<path>` for destination preservation
- Route config lives in `App.tsx`

## Before Modifying

1. `documentSymbol` on store files — understand all actions and selectors
2. `hover` on store type parameter — verify state shape
3. `findReferences` on store hooks — find all consuming components
4. `findReferences` on specific actions — verify usage before renaming
5. Check providers/auth setup before modifying auth flow
