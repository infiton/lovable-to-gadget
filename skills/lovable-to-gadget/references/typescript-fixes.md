# TypeScript Fixes

Gadget auto-generates strict types for all models and actions. Common issues when migrating:

## Nullable Fields

Gadget returns optional string fields as `string | null` and optional dateTime as `Date | null`. Handle with fallbacks:

```ts
// Display nullable strings
<EventMeta date={event.date || ''} time={event.time || ''} />
<EventDescription description={event.description || ''} />

// Nullable dates — Gadget returns Date objects (not strings)
// Use directly when not null; assert with ! only after checking
if (event.targetDate) {
  const targetDate = event.targetDate;  // Already a Date object
}

// Set state from nullable fields
setDescription(eventData.description || '');
setLocation(eventData.address || '');
```

## Route Params

`useParams()` returns `string | undefined`. Assert when you know the param exists:
```ts
const { id } = useParams();
await updateEvent({ id: id!, event: { ... } });
```

## Interface Alignment

When defining local interfaces that match Gadget's return types, account for nullability:
```ts
interface Event {
  id: string;
  title: string;
  date: string | null;        // Gadget returns null for optional fields
  time: string | null;
  backgroundImage: { url: string } | null;
  targetDate: Date | null;    // Gadget returns Date objects, not strings
  address: string | null;
}
```

## Using Gadget's Generated Types

Gadget auto-generates types for all models. You can import them from the client package instead of defining local interfaces:

```ts
import type { Event } from "@gadget-client/your-app-slug";
```

This eliminates most type mismatches. If you do define local interfaces, `useFindMany`/`useFindOne` return `GadgetRecord` types that include extra metadata. Cast when passing to components expecting your local interface:
```ts
<EventCard event={event as unknown as LocalEventType} />
```

When using `select` in queries, the return type narrows to only the selected fields — you may need to adjust interfaces accordingly.

## Unused Components Importing Removed Packages

Lovable apps often include shadcn/ui components that import packages not included in your dependencies (e.g., `chart.tsx` importing `recharts`, `sonner.tsx` importing `next-themes`). Either:

1. Delete unused components that reference uninstalled packages
2. Remove the problematic imports and hardcode defaults (e.g., replace `next-themes` theme detection with `theme="light"`)

## See Also

- [pitfalls.md](pitfalls.md) - Quick reference of all common issues
