# Access Control

Gadget has a built-in RBAC system. Row-level security from your existing backend (e.g., Supabase RLS policies) maps to Gadget's permission filters.

## Permissions File

Configure in `accessControl/permissions.gadget.ts`:

```ts
import type { GadgetPermissions } from "gadget-server";

export const permissions: GadgetPermissions = {
  type: "gadget/permissions/v1",
  roles: {
    "signed-in": {
      storageKey: "signed-in",
      default: { read: true, action: true },
      models: {
        event: {
          read: true,
          actions: {
            create: true,
            delete: { filter: "accessControl/filters/event/owner.gelly" },
            update: { filter: "accessControl/filters/event/owner.gelly" },
          },
        },
      },
    },
    admin: {
      storageKey: "admin",
      default: { read: true, action: true },
      // IMPORTANT: Must have explicit model grants — see pitfall below
      models: {
        event: {
          read: true,
          actions: { create: true, delete: true, update: true },
        },
        user: {
          read: true,
          actions: { changePassword: true, delete: true, signOut: true, update: true },
        },
      },
    },
    unauthenticated: {
      storageKey: "unauthenticated",
      models: {
        event: { read: true },
        user: {
          actions: {
            signIn: true, signUp: true,
            resetPassword: true, sendResetPassword: true,
            sendVerifyEmail: true, verifyEmail: true,
          },
        },
      },
    },
  },
};
```

## Gelly Filters (Row-Level Security)

Gelly filters replace your existing backend's RLS policies. Create `.gelly` files for each access pattern:

**Owner-only access:**
```gelly
// accessControl/filters/event/owner.gelly
filter ($session: Session) on Event [
  where createdById == $session.userId
]
```

**User-scoped access:**
```gelly
// accessControl/filters/user/tenant.gelly
filter ($session: Session) on User [
  where id == $session.userId
]
```

## Critical Pitfall: `default` Does Not Grant Access to Existing Models

`default: { read: true, action: true }` on a role only sets the default for **newly created** models. It does NOT grant access to models that already exist. You MUST explicitly list every model the role needs:

```ts
// Wrong — admin has no access to existing models despite `default`
admin: {
  storageKey: "admin",
  default: { read: true, action: true },
  // No models block = no access to event, user, etc.
}

// Correct — explicit grants for each model
admin: {
  storageKey: "admin",
  default: { read: true, action: true },
  models: {
    event: { read: true, actions: { create: true, update: true, delete: true } },
    user: { read: true, actions: { update: true, signOut: true } },
  },
}
```

## See Also

- [models-and-actions.md](models-and-actions.md) - Model and action setup
- [pitfalls.md](pitfalls.md) - Common mistakes reference
