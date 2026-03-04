# Migration Overview

## Approach

Gadget can deploy **any Vite app**. The migration strategy is:

1. **Preserve the Lovable frontend** — components, pages, styling, routing all stay the same
2. **Replace the backend integration layer** — swap out the existing backend SDK (Supabase, Firebase, etc.) with Gadget models, actions, and `@gadgetinc/react` hooks
3. **Minimize frontend stack changes** — keep React Router DOM v6, Tailwind, shadcn/ui, and all UI dependencies as-is

The Lovable app's SPA architecture maps directly to Gadget — no SSR conversion needed.

## Migration Phases

| Phase | What | Details |
|-------|------|---------|
| 1 | **Analyze** | Map existing tables → Gadget models, identify all backend calls |
| 2 | **Backend** | Create models, actions, permissions in Gadget |
| 3 | **Frontend** | Move Lovable source into Gadget's `web/` directory |
| 4 | **Translate** | Replace all backend SDK calls with Gadget hooks |
| 5 | **TypeScript** | Fix type issues from Gadget's generated types |
| 6 | **Missing pages** | Add any auth flow pages Gadget requires (email verification, password reset) |

## Phase 1: Analyze the Lovable App

Before writing any code, map the existing app's data model and integration points.

### Map Tables → Gadget Models

Read every backend query in the Lovable app to identify:
- **Tables** (become Gadget models)
- **Columns** and their types (become Gadget fields)
- **Relationships** (foreign keys become `belongsTo`/`hasMany`)
- **Row-level security / access policies** (become Gadget access control filters)
- **File storage** (become Gadget `file` fields with `allowPublicAccess: true`)

### Map Auth → Gadget Auth

Lovable apps typically use their backend's auth SDK for sign up, sign in, sign out, session management, and OAuth. These all map to Gadget's built-in `user` model and auth actions.

### Identify All Backend Calls

Search the Lovable codebase for every instance of the backend SDK being used. Adapt the search patterns to whichever backend is present:

| Backend | Search for |
|---|---|
| Supabase | `supabase.from(`, `supabase.auth.`, `supabase.storage.` |
| Firebase | `firebase.`, `getFirestore`, `getAuth`, `collection(`, `doc(` |
| Convex | `useQuery(`, `useMutation(`, `convex/` |
| Custom REST | `fetch(`, `axios.`, API base URL references |

Each call must be translated to a Gadget equivalent. See [auth-translation.md](auth-translation.md) and [data-translation.md](data-translation.md).

### Identify Backend Config to Remove

Lovable apps typically have backend-specific files that should be deleted after migration:
- Client initialization files (e.g., `src/integrations/supabase/`, `src/lib/firebase.ts`)
- Auto-generated type files from the backend (e.g., Supabase's `types.ts`)
- Environment variables for the old backend (`VITE_SUPABASE_URL`, `VITE_FIREBASE_API_KEY`, etc.) — Gadget manages its own connection via the auto-generated API client

## See Also

- [models-and-actions.md](models-and-actions.md) - Creating the Gadget backend
- [frontend-setup.md](frontend-setup.md) - Moving the frontend into Gadget
