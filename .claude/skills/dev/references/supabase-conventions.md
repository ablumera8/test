# Supabase Conventions

> **Note:** Conventions below are based on the PRD (`docs/prds/supabase-auth.md`). Re-run `/init-dev` after scaffolding to verify against actual code.

## Client Setup

Single Supabase client instance in `src/lib/supabase.ts`:

```typescript
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
)
```

- Never create multiple clients — import from `src/lib/supabase.ts`
- Only use `anon` key client-side — never expose `service_role` key

## Auth Methods

### Magic Link (Passwordless)

```typescript
const { error } = await supabase.auth.signInWithOtp({ email })
```

### Social OAuth

```typescript
const { error } = await supabase.auth.signInWithOAuth({
  provider: 'google' // or 'github'
})
```

### Sign Out

```typescript
await supabase.auth.signOut()
```

### Session Listener

```typescript
supabase.auth.onAuthStateChange((event, session) => {
  // events: INITIAL_SESSION, SIGNED_IN, SIGNED_OUT, TOKEN_REFRESHED
})
```

## Types

- Supabase types come from `@supabase/supabase-js` (`User`, `Session`)
- Optional: generate database types with `npx supabase gen types typescript --linked > src/types/supabase.ts`

## Auth Configuration (Dashboard)

- Enable Email provider (magic link) in Supabase dashboard
- Enable Google OAuth: configure OAuth client ID/secret in Supabase + Google Cloud Console
- Enable GitHub OAuth: configure OAuth app in Supabase + GitHub Developer Settings
- Set **Site URL** to `http://localhost:5173` for local dev
- Set **Redirect URLs** to `http://localhost:5173/**` for local dev

## Security

- `anon` key only in client code — safe to expose
- Row Level Security (RLS) should be enabled on all tables when added later
- Magic link TTL: 24h (Supabase default)
- OAuth state parameter prevents CSRF — handled automatically by Supabase

## Before Modifying

1. `documentSymbol` on `src/lib/supabase.ts` — understand client configuration
2. `hover` on auth method calls — verify parameter types
3. `findReferences` on supabase client import — find all consuming files
4. Check Supabase dashboard settings before adding new auth providers
5. `goToDefinition` on Supabase types to verify available fields
