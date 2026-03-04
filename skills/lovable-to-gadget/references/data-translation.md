# Data Translation

Replace all backend data queries and mutations with Gadget's `@gadgetinc/react` hooks.

## Querying Multiple Records

**Typical Lovable pattern** (e.g., Supabase):
```ts
const { data, error } = await supabase
  .from('events')
  .select('id, title, date, background_image_url')
  .order('target_date', { ascending: true });
```

**Gadget replacement:**
```ts
import { useFindMany } from "@gadgetinc/react";

const [{ data: events, fetching, error }] = useFindMany(api.event, {
  sort: { targetDate: "Ascending" },
  select: {
    id: true,
    title: true,
    date: true,
    backgroundImage: { url: true },  // File fields return { url: string }
  },
});
```

## Querying a Single Record

```ts
import { useFindOne } from "@gadgetinc/react";

const [{ data: event, fetching, error }] = useFindOne(api.event, id as string, {
  pause: !id,  // Skip query if id not ready
});
```

## Filtered Queries

**Typical Lovable pattern:**
```ts
const { data } = await supabase
  .from('event_registrations')
  .select('id')
  .eq('event_id', eventId)
  .eq('user_id', userId);
```

**Gadget replacement:**
```ts
const [{ data: registrations }] = useFindMany(api.eventRegistration, {
  filter: {
    eventId: { equals: eventId },   // Use the scalar ID field
    userId: { equals: userId },
  },
  select: { id: true },
  pause: !userId,  // Pause until value is available
});
```

> **CRITICAL**: For `belongsTo` relationship fields, filter on the **scalar ID field**, NOT the relationship name. Gadget auto-creates a scalar field named `{relationshipName}Id` for every `belongsTo` relationship (e.g., `event` → `eventId`, `createdBy` → `createdById`, `author` → `authorId`). Using the relationship name directly (e.g., `{ event: { equals: id } }`) will fail with a TypeScript error about `RelationshipFilter`.

### Conditional Queries with `pause`

Use `pause` to prevent a query from running until a required value is available:
```ts
const [{ data }] = useFindMany(api.post, {
  filter: { authorId: { equals: userId! } },
  pause: !userId,  // Don't run query until userId exists
});
```

### Nested Selection (Through Relationships)

```ts
const [{ data }] = useFindMany(api.eventRegistration, {
  filter: { userId: { equals: user.id } },
  select: {
    id: true,
    event: { id: true, title: true, date: true, backgroundImage: { url: true } },
  },
});
```

## Pagination

`useFindMany` returns a maximum of 250 records per call (default varies). Use `first` and cursor-based pagination for larger datasets:

```ts
const [{ data }] = useFindMany(api.event, {
  first: 20,  // Page size
});

// data.hasNextPage, data.endCursor available for cursor pagination
```

For real-time updates, add `live: true`:
```ts
const [{ data }] = useFindMany(api.event, {
  live: true,  // Auto-refetches when data changes on the server
});
```

## Creating Records

```ts
const [{ fetching }, createEvent] = useAction(api.event.create);

await createEvent({
  event: {
    title: "My Event",
    description: "...",
    backgroundImage: imageFile ? { file: imageFile } : undefined,
  },
});
```

## Updating Records

```ts
const [{ fetching }, updateEvent] = useAction(api.event.update);

await updateEvent({
  id: eventId,
  event: { title: "Updated Title" },
});
```

## Deleting Records

```ts
const [{ fetching }, deleteEvent] = useAction(api.event.delete);

await deleteEvent({ id: eventId });
```

## Mutation Error Handling

`useAction` returns an `error` object — check it after the call:
```ts
const [{ error }, createEvent] = useAction(api.event.create);

await createEvent({ event: { title: "..." } });
if (error) {
  toast.error(error.message);
}
```

> **Note:** `useFindMany` and `useFindOne` are **declarative** hooks — they run automatically on mount and re-render. `useAction` returns an **imperative** function you call explicitly. This is different from TanStack Query where `useMutation` is also a hook but works differently.

## Relationship Linking

**Typical Lovable pattern:**
```ts
await supabase.from('event_registrations').insert({
  event_id: eventId,
  user_id: userId,
});
```

**Gadget replacement:**
```ts
await createRegistration({
  eventRegistration: {
    event: { _link: eventId },  // _link syntax for belongsTo
  },
  // user is auto-linked via session in the action's run function
});
```

The `{ _link: id }` syntax links to an existing record by ID. User relationships are typically handled automatically via session linking in the action (see [models-and-actions.md](models-and-actions.md)).

## File Upload

**Typical Lovable pattern** (multi-step):
```ts
// Step 1: Upload file to storage
const uploadResult = await backend.storage.upload(file);
// Step 2: Get URL
const url = backend.storage.getPublicUrl(uploadResult.path);
// Step 3: Save URL to record
await backend.from('events').update({ image_url: url }).eq('id', id);
```

**Gadget replacement** (single step):
```ts
await createEvent({
  event: {
    title: "My Event",
    backgroundImage: { file: imageFile },  // Pass File object directly
  },
});

// Display:
<img src={event.backgroundImage?.url} />
```

Gadget handles upload, storage, and URL generation automatically.

## Filter Syntax Reference

```ts
// Equality
{ fieldName: { equals: value } }
{ fieldName: { notEquals: value } }

// In list
{ fieldName: { in: [value1, value2] } }

// String operations
{ fieldName: { startsWith: "prefix" } }
{ fieldName: { contains: "substring" } }

// Relationship (use scalar ID field, NOT relationship name)
{ createdById: { equals: userId } }   // Correct
{ createdBy: { equals: userId } }     // Wrong

// Multiple conditions (AND by default)
{ eventId: { equals: eventId }, userId: { equals: userId } }
```

## Hook Translation Reference

| Typical Lovable Pattern | Gadget |
|---|---|
| `useQuery(['events'], () => backend.from(...))` | `useFindMany(api.event, { ... })` |
| `useQuery(['event', id], () => backend.from(...).eq('id', id))` | `useFindOne(api.event, id)` |
| `useMutation(() => backend.from(...).insert(...))` | `useAction(api.event.create)` |
| `useMutation(() => backend.from(...).update(...))` | `useAction(api.event.update)` |
| `useMutation(() => backend.from(...).delete(...))` | `useAction(api.event.delete)` |
| `backend.auth.getSession()` / `onAuthStateChange` | `useUser()` from `@gadgetinc/react` |

## Ownership Check

```ts
if (eventData.createdById !== user.id) {
  toast.error('You do not have permission to edit this event');
  navigate('/my-events');
  return;
}
```

## See Also

- [auth-translation.md](auth-translation.md) - Auth-specific translations
- [typescript-fixes.md](typescript-fixes.md) - Type issues with Gadget's generated types
