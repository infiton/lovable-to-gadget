# TypeScript Fixes

Gadget auto-generates strict types for all models and actions. Common issues when migrating:

## Nullable Fields

Gadget returns optional string fields as `string | null` and optional dateTime as `Date | null`. Handle with fallbacks:

```ts
// Display nullable strings
<EventMeta date={event.date || ''} time={event.time || ''} />
<EventDescription description={event.description || ''} />

// Use nullable dates
const targetDate = new Date(event.targetDate!);  // Non-null assertion when you've already checked

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

## Type Casting for GadgetRecord

Gadget's `useFindMany` and `useFindOne` return `GadgetRecord` types. When passing to components expecting your local interface:
```ts
<EventCard event={event as unknown as Event} />
```

## Unused Components Importing Removed Packages

Lovable apps often include shadcn/ui components that import packages not included in your dependencies (e.g., `chart.tsx` importing `recharts`, `sonner.tsx` importing `next-themes`). Either:

1. Delete unused components that reference uninstalled packages
2. Remove the problematic imports and hardcode defaults (e.g., replace `next-themes` theme detection with `theme="light"`)

## See Also

- [pitfalls.md](pitfalls.md) - Quick reference of all common issues
