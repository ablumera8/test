# Styling Conventions

> **Note:** Conventions below are based on the PRD (`docs/prds/supabase-auth.md`). Re-run `/init-dev` after scaffolding to verify against actual code.

## Approach

CSS Modules — scoped styles per component, no global class name collisions.

## Component Styling Pattern

Each component with styles has a co-located `.module.css` file:

```
components/auth/
  LoginForm.tsx
  LoginForm.module.css
  OAuthButtons.tsx
  OAuthButtons.module.css
```

Import and apply:

```typescript
import styles from './LoginForm.module.css'

function LoginForm() {
  return <form className={styles.container}>...</form>
}
```

## Class Naming

Within CSS Modules, use camelCase or kebab-case class names (pick one convention and stay consistent):

```css
/* camelCase (preferred for JS access) */
.submitButton { }
.errorText { }

/* kebab-case also valid */
.submit-button { }  /* accessed as styles['submit-button'] */
```

## No Shared CSS Utilities

This project uses CSS Modules without a utility framework (no Tailwind, no Bootstrap). Shared styles (colors, spacing, typography) should use CSS custom properties defined in a root file if needed.

## File Structure

- Page-level components (`src/pages/`) may also have `.module.css` files
- No global CSS except `index.css` for resets and root variables
- Style files live alongside the component they style — never in a separate `styles/` directory

## Before Modifying

1. `hover` on style utility calls — verify parameter types
2. `findReferences` on shared CSS classes — check usage before renaming
3. `documentSymbol` on component files — check existing variant props
4. Verify the corresponding `.module.css` file exists alongside any new component
