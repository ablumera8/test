# PRD: Supabase Authentication Scaffold

---

## Implementation Sequence

| Phase | Priority | Dependencies | Prerequisites |
|-------|----------|--------------|---------------|
| **Phase 1: Project Setup** | CRITICAL | None | Node.js 18+, Supabase account |
| **Phase 2: Auth Core** | CRITICAL | Phase 1 | Supabase project + credentials |
| **Phase 3: Auth UI** | HIGH | Phase 2 | None |
| **Phase 4: Route Protection** | HIGH | Phase 2 | None |

**This PRD implements:** A React + Vite + TypeScript project scaffold with Supabase authentication (magic link + social logins), Zustand state management, and CSS Modules styling.

**Previous PRDs:** N/A
**Next PRDs:** N/A (standalone scaffold)

---

## Reference

- **Supabase Auth docs:** https://supabase.com/docs/guides/auth
- **Supabase JS client:** `@supabase/supabase-js` v2
- **Zustand:** `zustand` v5

---

## 1. Overview

A minimal, production-ready authentication scaffold built with React, Vite, and TypeScript. Uses Supabase as the auth provider with magic link (passwordless email) and social OAuth login (Google, GitHub). Auth state is managed via Zustand and persisted across sessions. The scaffold provides protected route guards, a clean login/signup flow, and a basic authenticated landing page — ready to extend into a full application.

## 2. Goals

- Working auth flow (magic link + social OAuth) within 30 minutes of setup
- Zero custom auth backend logic — Supabase handles all token management
- Persistent sessions across browser refreshes via Zustand + Supabase session listener
- Clean separation of auth logic from UI components
- Type-safe Supabase client and auth state

## 3. User Stories

### US-001: Sign in with magic link
**Description:** As a user, I want to sign in by entering my email and clicking a link sent to my inbox so that I don't need to remember a password.

**Acceptance Criteria:**
- [ ] User enters email on login page
- [ ] Confirmation message shown after submission ("Check your email")
- [ ] Clicking the link in the email redirects back to the app and authenticates the user
- [ ] Supabase `signInWithOtp()` is used (no custom email logic)
- [ ] Error state shown if email send fails

### US-002: Sign in with social provider
**Description:** As a user, I want to sign in with Google or GitHub so that I can authenticate with one click.

**Acceptance Criteria:**
- [ ] Google and GitHub OAuth buttons visible on login page
- [ ] Clicking a provider redirects to its OAuth flow
- [ ] Successful OAuth returns to app with authenticated session
- [ ] Error state shown if OAuth fails or is denied
- [ ] Supabase `signInWithOAuth()` is used

### US-003: Sign out
**Description:** As an authenticated user, I want to sign out so that my session is terminated.

**Acceptance Criteria:**
- [ ] Sign out button visible when authenticated
- [ ] Clicking it clears the session via Supabase `signOut()`
- [ ] User is redirected to the login page
- [ ] Zustand auth state resets to unauthenticated

### US-004: Protected routes
**Description:** As a developer, I want unauthenticated users to be redirected to the login page when accessing protected routes.

**Acceptance Criteria:**
- [ ] `ProtectedRoute` component wraps authenticated routes
- [ ] Unauthenticated access redirects to `/login`
- [ ] Original destination is preserved in `redirect` query param
- [ ] After auth, user is redirected back to original destination

### US-005: Session persistence
**Description:** As a user, I want to stay logged in when I refresh the page or close and reopen the browser.

**Acceptance Criteria:**
- [ ] Supabase `onAuthStateChange` listener initializes on app mount
- [ ] Zustand store hydrates from Supabase session on load
- [ ] Refreshing the page maintains authenticated state
- [ ] Expired sessions are detected and user is logged out

## 4. Functional Requirements

### 4.1 Project Scaffolding

- FR-1: Vite project initialized with React + TypeScript template (`npm create vite@latest`)
- FR-2: Directory structure follows:
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
- FR-3: Environment variables configured via `.env.local`:
  ```
  VITE_SUPABASE_URL=https://<project>.supabase.co
  VITE_SUPABASE_ANON_KEY=<anon-key>
  ```

### 4.2 Supabase Client

- FR-4: Single Supabase client instance exported from `src/lib/supabase.ts`
  ```typescript
  import { createClient } from '@supabase/supabase-js'

  export const supabase = createClient(
    import.meta.env.VITE_SUPABASE_URL,
    import.meta.env.VITE_SUPABASE_ANON_KEY
  )
  ```
- FR-5: TypeScript types generated from Supabase schema (optional, via `supabase gen types`)

### 4.3 Auth Store (Zustand)

- FR-6: Auth store at `src/store/authStore.ts` with shape:
  ```typescript
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
- FR-7: `signOut` action calls `supabase.auth.signOut()` and resets state
- FR-8: Store initialized by `useAuth` hook which subscribes to `supabase.auth.onAuthStateChange`

### 4.4 Auth Hook

- FR-9: `useAuth` hook at `src/hooks/useAuth.ts`:
  - Subscribes to `supabase.auth.onAuthStateChange` on mount
  - Updates Zustand store on `SIGNED_IN`, `SIGNED_OUT`, `TOKEN_REFRESHED` events
  - Sets `loading: false` after initial session check
  - Returns `{ user, session, loading, signOut }` from the store

### 4.5 Login Page

- FR-10: Login page at `src/pages/Login.tsx` with email input + magic link button
- FR-11: Email input validated before submission (non-empty, basic format check)
- FR-12: Calls `supabase.auth.signInWithOtp({ email })` on submit
- FR-13: Shows success state: "Check your email for a login link"
- FR-14: Shows error state from Supabase response
- FR-15: Social OAuth buttons for Google and GitHub via `supabase.auth.signInWithOAuth({ provider })`
- FR-16: Styled with CSS Modules (`LoginForm.module.css`, `OAuthButtons.module.css`)

### 4.6 Protected Routes

- FR-17: `ProtectedRoute` component at `src/components/auth/ProtectedRoute.tsx`
- FR-18: Reads `loading` and `user` from auth store
- FR-19: While `loading === true`, renders a loading spinner or skeleton
- FR-20: If `loading === false && !user`, redirects to `/login?redirect=<current path>`
- FR-21: If `loading === false && user`, renders `<Outlet />` (nested routes)

### 4.7 Dashboard Page

- FR-22: Simple authenticated landing page at `src/pages/Dashboard.tsx`
- FR-23: Displays user email and provider (from `user.app_metadata`)
- FR-24: Sign out button that calls `signOut()` from auth store

### 4.8 Routing

- FR-25: React Router v6 with route structure:
  ```
  /login       -> Login (public)
  /            -> ProtectedRoute -> Dashboard
  ```
- FR-26: Authenticated users visiting `/login` are redirected to `/`

## 5. Non-Goals

- Email/password authentication (magic link only)
- User profile management or settings
- Role-based access control (RBAC)
- Email verification flow (beyond magic link)
- Password reset flow
- Multi-factor authentication (MFA)
- User registration form / onboarding flow
- Database schema or user metadata tables
- Rate limiting or brute-force protection (handled by Supabase)
- E2E tests (unit tests only for store and hook logic)

## 6. Technical Considerations

### Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `react` | ^19 | UI framework |
| `react-dom` | ^19 | DOM rendering |
| `react-router-dom` | ^7 | Client-side routing |
| `@supabase/supabase-js` | ^2 | Supabase client (auth, realtime) |
| `zustand` | ^5 | Auth state management |

### Supabase Configuration

- Enable Email (magic link) provider in Supabase dashboard
- Enable Google OAuth: configure OAuth client ID/secret in Supabase + Google Cloud Console
- Enable GitHub OAuth: configure OAuth app in Supabase + GitHub Developer Settings
- Set **Site URL** to `http://localhost:5173` for local development
- Set **Redirect URLs** to `http://localhost:5173/**` for local development

### Security

- Never expose `service_role` key — only `anon` key in client-side code
- Row Level Security (RLS) should be enabled on all Supabase tables (when added later)
- Magic link emails expire after configured TTL (Supabase default: 24h)
- OAuth state parameter prevents CSRF — handled automatically by Supabase

### Session Handling

- Supabase manages JWT tokens and refresh automatically
- `onAuthStateChange` fires on: `INITIAL_SESSION`, `SIGNED_IN`, `SIGNED_OUT`, `TOKEN_REFRESHED`
- Zustand store is the single source of truth for auth state in the React tree
- No need for React Context — Zustand handles subscriptions and re-renders

## 7. Reusable Code

N/A — greenfield project. All code is new.

## 8. Open Questions

- Which social providers beyond Google and GitHub? (Easy to add — just configure in Supabase dashboard and add a button)
- Custom email template for magic link emails? (Configurable in Supabase dashboard)
- Deployment target? (Affects redirect URL configuration in Supabase)
