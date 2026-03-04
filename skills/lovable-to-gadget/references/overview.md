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
| 6 | **Missing pages** | Add email verification, password reset pages |

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

Search the Lovable codebase for every instance of the backend SDK being used. For example, with Supabase:
```
supabase.from(
supabase.auth.
supabase.storage.
```

Each call must be translated to a Gadget equivalent. See [auth-translation.md](auth-translation.md) and [data-translation.md](data-translation.md).

## See Also

- [models-and-actions.md](models-and-actions.md) - Creating the Gadget backend
- [frontend-setup.md](frontend-setup.md) - Moving the frontend into Gadget
