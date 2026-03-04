# Migration Checklist

## Backend Setup
- [ ] Map all existing tables to Gadget model schemas with proper field types
- [ ] Create CRUD actions for each model with session linking where needed
- [ ] Set up access control with Gelly filters for row-level security
- [ ] Add explicit model grants to admin role (don't rely on `default`)
- [ ] Extend user model with app-specific fields
- [ ] Update `settings.gadget.ts` with correct auth redirect paths and OAuth config

## Frontend Migration
- [ ] Delete Gadget's existing frontend files (keep `web/api.ts`)
- [ ] Copy Lovable `src/` to Gadget `web/`
- [ ] Delete old backend files copied into `web/` (client init, generated types, auth providers)
- [ ] Add `gadget()` plugin to Vite config (`vite.config.mts` or `vite.config.mjs`)
- [ ] Update `@` path alias from `./src` to `./web` in both Vite config and `tsconfig.json`
- [ ] Update `index.html` script src to `/web/main.tsx`
- [ ] Update Tailwind content paths to `./web/**`
- [ ] Copy `postcss.config.js` if using Tailwind v3
- [ ] Update `components.json` css path to `web/index.css` (if using shadcn/ui)
- [ ] Update `package.json`: remove backend SDK, add `@gadgetinc/react`
- [ ] Replace `main.tsx` providers with `GadgetProvider`
- [ ] Remove old auth context/provider (replaced by `GadgetProvider` + `useUser()`)
- [ ] Remove old backend environment variables

## Auth Translation
- [ ] Replace all auth state management with `useUser()` hook
- [ ] Replace sign in/up/out calls with `useAction()` hooks
- [ ] Ensure auth actions use flat params (not nested under model name)
- [ ] Ensure `signOut` passes `{ id: user.id }`
- [ ] Add Google OAuth button (`<a href="/auth/google/start">`) if desired
- [ ] Add `/verify-email` page
- [ ] Add `/reset-password` page if app supports it

## Data Translation
- [ ] Replace all data queries with `useFindMany` / `useFindOne`
- [ ] Replace all mutations with `useAction()`
- [ ] Replace all file uploads with `{ file: File }` pattern
- [ ] Fix relationship filters to use scalar ID fields (`{relationshipName}Id`)
- [ ] Add `pause` option to queries that depend on async values

## TypeScript & Cleanup
- [ ] Handle nullable fields (`string | null`, `Date | null`)
- [ ] Fix `useParams()` returns (`string | undefined`)
- [ ] Audit `web/components/ui/` for unused components importing missing packages
- [ ] Run TypeScript check (`npx tsc --noEmit`) and fix all errors

## Verification
- [ ] Start dev server and confirm app loads without errors
- [ ] Test unauthenticated flows (public pages, sign up, sign in)
- [ ] Test authenticated flows (create/update/delete records, file uploads)
- [ ] Test access control (users can only modify their own records)
- [ ] Test Google OAuth flow (if enabled)
- [ ] Test email verification flow
