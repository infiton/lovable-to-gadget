# Common Pitfalls

Quick-reference summary of mistakes documented in detail in the other reference files. See each linked file for full context.

| # | Pitfall | Fix | Details |
|---|---------|-----|---------|
| 1 | Auth actions wrapped as `{ user: { email, password } }` | Use flat params: `{ email, password }` | [auth-translation](auth-translation.md) |
| 2 | `signOut({})` with no args | Must pass `{ id: user.id }` | [auth-translation](auth-translation.md) |
| 3 | Filter on relationship name: `{ author: { equals: id } }` | Filter on scalar ID field: `{ authorId: { equals: id } }` | [data-translation](data-translation.md) |
| 4 | Admin role `default` granting access to existing models | `default` only applies to future models — add explicit `models` grants | [access-control](access-control.md) |
| 5 | Nullable fields (`string \| null`, `Date \| null`) causing type errors | Add `\|\| ''` fallbacks or `!` non-null assertions | [typescript-fixes](typescript-fixes.md) |
| 6 | Lovable scaffolds unused shadcn/ui components that import missing packages | Audit `web/components/ui/` for broken imports; delete unused components or remove the import | [typescript-fixes](typescript-fixes.md) |
| 7 | Missing `/verify-email` page | Create it — Gadget's `sendVerifyEmail` action generates links to it | [auth-translation](auth-translation.md) |
| 8 | Multi-step file upload pattern from original backend | Replace with single `{ file: File }` in action params | [data-translation](data-translation.md) |
| 9 | Old backend's data fetching patterns (`useQuery`, `useMutation`, etc.) | Replace with `useFindMany`, `useFindOne`, `useAction` | [data-translation](data-translation.md) |
| 10 | `useParams()` returns `string \| undefined` | Use `id!` or `id as string` when param is guaranteed | [typescript-fixes](typescript-fixes.md) |
| 11 | Gadget `dateTime` returns `Date` objects, not strings | Use `Date \| null` in interfaces; already a `Date` — no need for `new Date()` | [typescript-fixes](typescript-fixes.md) |
| 12 | Google OAuth button using `onClick`/`fetch` | Must use `<a href="/auth/google/start">` for full-page redirect | [auth-translation](auth-translation.md) |
| 13 | Import paths break after moving `src/` to `web/` | Update `@` alias in Vite config and `tsconfig.json` to point to `./web` | [frontend-setup](frontend-setup.md) |
| 14 | Accidentally deleting or overwriting `web/api.ts` | Keep this file — it's Gadget's auto-generated API client | [frontend-setup](frontend-setup.md) |
| 15 | Old backend config/types files left in `web/` after copy | Delete backend client files, generated types, auth providers | [frontend-setup](frontend-setup.md) |

## See Also

- [auth-translation.md](auth-translation.md) - Detailed auth patterns
- [data-translation.md](data-translation.md) - Detailed query/mutation patterns
- [typescript-fixes.md](typescript-fixes.md) - TypeScript-specific issues
