# Migration Checklist

## Backend Setup
- [ ] Map all existing tables to Gadget model schemas with proper field types
- [ ] Create CRUD actions for each model with session linking where needed
- [ ] Set up access control with Gelly filters for row-level security
- [ ] Add explicit model grants to admin role (don't rely on `default`)
- [ ] Extend user model with app-specific fields
- [ ] Update `settings.gadget.ts` with correct auth redirect paths and OAuth config

## Frontend Migration
- [ ] Delete Gadget's existing frontend files
- [ ] Copy Lovable `src/` to Gadget `web/`
- [ ] Add `gadget()` plugin to Vite config
- [ ] Update `@` path alias from `./src` to `./web`
- [ ] Update `index.html` script src to `/web/main.tsx`
- [ ] Update Tailwind content paths to `./web/**`
- [ ] Update `components.json` css path to `web/index.css`
- [ ] Update `package.json`: remove backend SDK, add `@gadgetinc/react`
- [ ] Replace `main.tsx` providers with `GadgetProvider`

## Auth Translation
- [ ] Replace all auth state management with `useUser()` hook
- [ ] Replace sign in/up/out calls with `useAction()` hooks
- [ ] Ensure auth actions use flat params (not nested under model name)
- [ ] Ensure `signOut` passes `{ id: user.id }`
- [ ] Add Google OAuth button (`<a href="/auth/google/start">`) to auth forms
- [ ] Add `/verify-email` page
- [ ] Add `/reset-password` page if app supports it

## Data Translation
- [ ] Replace all data queries with `useFindMany` / `useFindOne`
- [ ] Replace all mutations with `useAction()`
- [ ] Replace all file uploads with `{ file: File }` pattern
- [ ] Fix relationship filters to use scalar ID fields (`eventId`, `userId`, `createdById`)
- [ ] Add `pause` option to queries that depend on async values

## TypeScript & Cleanup
- [ ] Handle nullable fields (`string | null`, `Date | null`)
- [ ] Fix `useParams()` returns (`string | undefined`)
- [ ] Remove unused components that import uninstalled packages
- [ ] Remove `next-themes` from `sonner.tsx` if present
- [ ] Run TypeScript check and fix all errors
