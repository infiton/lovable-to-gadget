# Models and Actions

## Field Type Mapping

Map your existing backend's column types to Gadget field types:

| Common Backend Type | Gadget Type | Notes |
|---|---|---|
| `text`, `varchar`, `string` | `string` | |
| `int`, `integer`, `number` | `number` | |
| `bool`, `boolean` | `boolean` | |
| `timestamp`, `datetime` | `dateTime` | Set `includeTime: true` |
| Foreign key (UUID/ID) | `belongsTo` | Creates relationship |
| `json`, `jsonb` | `json` | |
| `enum` / string union | `enum` | Define `acceptedValues` list |
| File/blob/storage URL | `file` | Set `allowPublicAccess: true` for public images |

## Model Schemas

Create `api/models/{modelName}/schema.gadget.ts` for each table.

**Example:**
```ts
// api/models/event/schema.gadget.ts
import type { GadgetModel } from "gadget-server";

export const schema: GadgetModel = {
  type: "gadget/model-schema/v2",
  storageKey: "DataModel-Event",  // Unique, stable storage key
  fields: {
    title: {
      type: "string",
      validations: { required: true },
      storageKey: "event-title",
    },
    description: { type: "string", storageKey: "event-description" },
    createdBy: {
      type: "belongsTo",
      parent: { model: "user" },
      storageKey: "event-createdBy",
    },
    backgroundImage: {
      type: "file",
      allowPublicAccess: true,
      storageKey: "event-backgroundImage",
    },
    targetDate: {
      type: "dateTime",
      includeTime: true,
      storageKey: "event-targetDate",
    },
    registrations: {
      type: "hasMany",
      children: { model: "eventRegistration", belongsToField: "event" },
      storageKey: "event-registrations",
    },
  },
};
```

### Storage Keys

- Every model and field needs a unique `storageKey`
- These are permanent database identifiers — changing them after creation will cause data loss
- Convention: `"DataModel-{ModelName}"` for models, `"{model}-{field}"` for fields
- The user model already exists with pre-assigned storage keys — only add new fields to it

> **Tip:** You can use `ggt add model {modelName}` to scaffold models with auto-generated storage keys and default actions, then customize from there.

## CRUD Actions

Create standard actions for each model:

### Create Action
```ts
// api/models/event/actions/create.ts
import { applyParams, save, ActionOptions } from "gadget-server";

export const run: ActionRun = async ({ params, record, session }) => {
  applyParams(params, record);
  // Auto-link to current user via session
  if (session?.get("user") && !record.createdById) {
    record.createdBy = { _link: session.get("user") };
  }
  await save(record);
};

export const options: ActionOptions = { actionType: "create" };
```

### Update Action
```ts
// api/models/event/actions/update.ts
import { applyParams, save, ActionOptions } from "gadget-server";

export const run: ActionRun = async ({ params, record }) => {
  applyParams(params, record);
  await save(record);
};

export const options: ActionOptions = { actionType: "update" };
```

### Delete Action
```ts
// api/models/event/actions/delete.ts
import { deleteRecord, ActionOptions } from "gadget-server";

export const run: ActionRun = async ({ params, record }) => {
  await deleteRecord(record);
};

export const options: ActionOptions = { actionType: "delete" };
```

## Session Linking Pattern

For models where records should be auto-associated with the logged-in user (replacing backend RLS like Supabase's `auth.uid()`):

```ts
if (session?.get("user") && !record.userId) {
  record.user = { _link: session.get("user") };
}
```

Add this to the `create` action for any model that tracks ownership.

## Extending the User Model

Gadget already has a `user` model with auth fields (`email`, `password`, `roleList`, etc.). Add your app-specific fields to the existing schema:

```ts
// Add to api/models/user/schema.gadget.ts (existing file)
displayName: { type: "string", storageKey: "user-displayName" },
events: {
  type: "hasMany",
  children: { model: "event", belongsToField: "createdBy" },
  storageKey: "user-events",
},
```

**Never modify core auth fields** — they power the auth system.

## See Also

- [access-control.md](access-control.md) - Permissions and row-level security
- [data-translation.md](data-translation.md) - How frontend code calls these actions
