# Auth Translation

Gadget has built-in authentication with email/password and Google OAuth. This replaces whatever auth system the Lovable app uses.

## Auth Settings

Update `settings.gadget.ts` to match your Lovable app's routes:
```ts
plugins: {
  authentications: {
    settings: {
      redirectOnSignIn: "/dashboard",   // Where to go after sign in (match your app's main route)
      signInPath: "/auth",              // Your auth page route
      unauthorizedUserRedirect: "signInPath",
      defaultSignedInRoles: ["signed-in"],
    },
    methods: {
      emailPassword: true,
      googleOAuth: { scopes: ["email", "profile"], offlineAccess: false },
    },
  },
},
```

## Auth State

**Typical Lovable pattern** (e.g., Supabase):
```ts
const [user, setUser] = useState(null);
useEffect(() => {
  backend.auth.getSession().then(session => setUser(session?.user ?? null));
  const subscription = backend.auth.onAuthStateChange(session => {
    setUser(session?.user ?? null);
  });
  return () => subscription.unsubscribe();
}, []);
```

**Gadget replacement:**
```ts
import { useUser } from "@gadgetinc/react";
const user = useUser();
// Returns: undefined (loading) | null (not authed) | User object
```

This single hook replaces ALL auth state management — session fetching, auth listeners, context providers, etc.

**Important:** `useUser()` returns **three** distinct states:
- `undefined` — still loading (don't redirect yet)
- `null` — not authenticated
- User object — authenticated

## Sign In / Sign Up / Sign Out

**Gadget replacement:**
```ts
import { useAction } from "@gadgetinc/react";
import { api } from "@/api";

// Each hook is typically in its own component; shown together here for reference
const [{ fetching: signingIn, error: signInError }, signIn] = useAction(api.user.signIn);
const [{ fetching: signingUp, error: signUpError }, signUp] = useAction(api.user.signUp);
const [{ fetching: signingOut }, signOut] = useAction(api.user.signOut);

await signIn({ email, password });      // TOP-LEVEL params, NOT nested
await signUp({ email, password });      // TOP-LEVEL params, NOT nested
await signOut({ id: user.id });         // signOut requires the user's id
```

### Error Handling

Auth actions return an `error` object on failure (wrong password, duplicate email, etc.):
```ts
const [{ error }, signIn] = useAction(api.user.signIn);

await signIn({ email, password });
if (error) {
  // error.message contains the human-readable error
  toast.error(error.message);
}
```

> **CRITICAL**: Auth actions take `{ email, password }` as TOP-LEVEL params. Do NOT wrap them as `{ user: { email, password } }`. This is the #1 mistake — Gadget's auth actions expect flat params.

> **CRITICAL**: `signOut` requires `{ id: userId }`. It does NOT accept an empty object `{}`.

## Param Shape Rule

Auth actions and model CRUD actions have **different** param shapes:
```ts
// Auth actions — FLAT params (no nesting)
signIn({ email, password })
signUp({ email, password })
signOut({ id: userId })
verifyEmail({ code })
resetPassword({ code, password })

// Model CRUD actions — nested under model name
api.event.create({ event: { title, description, ... } })
api.event.update({ id, event: { title, ... } })
api.event.delete({ id })
```

## Google OAuth

Gadget provides Google OAuth out of the box (when enabled in `settings.gadget.ts`). Add a Google sign-in button to your auth forms:

```tsx
<a
  href="/auth/google/start"
  className="w-full flex items-center justify-center gap-3 ..."
>
  <img
    src="https://assets.gadget.dev/assets/default-app-assets/google.svg"
    width={20}
    height={20}
    alt="Google"
  />
  Continue with Google
</a>
```

**Key details:**
- The URL is always `/auth/google/start` — Gadget handles the full OAuth redirect flow
- Must use an `<a>` tag (full page navigation), NOT a button with `onClick` — OAuth requires a server redirect
- Works for both sign-in AND sign-up — Gadget auto-creates the user if they don't exist
- After OAuth, Gadget redirects to `settings.gadget.ts → redirectOnSignIn`

## Auth Guards

**Typical Lovable pattern:**
```ts
useEffect(() => {
  backend.auth.getSession().then(session => {
    if (!session) navigate('/auth');
  });
}, []);
```

**Gadget replacement:**
```ts
const user = useUser();

useEffect(() => {
  if (user === undefined) return; // Still loading — don't redirect yet
  if (!user) navigate('/auth');
}, [user, navigate]);
```

## Admin Role Check

```ts
const isAdmin = user && (user as any).roles?.some(
  (role: any) => role.key === 'admin' || role.name === 'admin'
);
```

## Email Verification Page

Gadget's `sendVerifyEmail` action generates email links pointing to `/verify-email?code=xyz`. Create a page to handle this:

```tsx
// web/pages/VerifyEmail.tsx
import { useEffect, useState } from 'react';
import { useSearchParams, useNavigate } from 'react-router-dom';
import { useAction } from '@gadgetinc/react';
import { api } from '@/api';

const VerifyEmail = () => {
  const [searchParams] = useSearchParams();
  const navigate = useNavigate();
  const code = searchParams.get('code');
  const [{ data, fetching, error }, verifyEmail] = useAction(api.user.verifyEmail);
  const [status, setStatus] = useState<'verifying' | 'success' | 'error'>('verifying');

  useEffect(() => {
    if (!code) { setStatus('error'); return; }
    verifyEmail({ code }).then(() => setStatus('success')).catch(() => setStatus('error'));
  }, [code]);

  if (status === 'verifying') return <div>Verifying your email...</div>;
  if (status === 'error') return (
    <div>
      <p>Verification failed. The link may have expired.</p>
      <button onClick={() => navigate('/')}>Go Home</button>
    </div>
  );
  return (
    <div>
      <p>Email verified successfully!</p>
      <button onClick={() => navigate('/')}>Go Home</button>
    </div>
  );
};
```

Add to your routes:
```tsx
<Route path="/verify-email" element={<VerifyEmail />} />
```

Similarly, create a `/reset-password` page if your app supports password reset.

## Removing Old Auth Infrastructure

Delete the Lovable app's existing auth context/provider (e.g., `AuthContext.tsx`, `AuthProvider.tsx`, `useAuth.ts`). Replace all consumers with `useUser()` from `@gadgetinc/react`. The `GadgetProvider` in `main.tsx` handles session management — no additional auth context is needed.

## See Also

- [data-translation.md](data-translation.md) - Translating data queries
- [pitfalls.md](pitfalls.md) - Common auth mistakes
