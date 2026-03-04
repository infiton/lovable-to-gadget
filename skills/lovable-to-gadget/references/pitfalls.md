# Common Pitfalls

Quick-reference table of mistakes encountered during Lovable → Gadget migrations.

| # | Pitfall | Fix |
|---|---------|-----|
| 1 | Auth actions wrapped as `{ user: { email, password } }` | Use flat params: `{ email, password }` |
| 2 | `signOut({})` with no args | Must pass `{ id: user.id }` |
| 3 | Filter on relationship name: `{ event: { equals: id } }` | Filter on scalar ID field: `{ eventId: { equals: id } }` |
| 4 | Admin role `default` granting access to existing models | `default` only applies to future models — add explicit `models: { ... }` grants |
| 5 | Nullable fields (`string \| null`, `Date \| null`) causing type errors | Add `\|\| ''` fallbacks or `!` non-null assertions |
| 6 | `next-themes` import in shadcn `sonner.tsx` | Remove import, hardcode `theme="light"` |
| 7 | Unused component importing uninstalled package (e.g., `chart.tsx` → `recharts`) | Delete unused components or remove the import |
| 8 | Missing `/verify-email` page | Create it — Gadget's `sendVerifyEmail` action generates links to it |
| 9 | Multi-step file upload pattern from original backend | Replace with single `{ file: File }` in action params |
| 10 | TanStack Query patterns (`useQuery`, `useMutation`) | Replace with `useFindMany`, `useFindOne`, `useAction` |
| 11 | `useParams()` returns `string \| undefined` | Use `id!` or `id as string` when param is guaranteed |
| 12 | Gadget `dateTime` returns `Date` objects, not strings | Use `Date \| null` in interfaces, `new Date(field!)` for conversion |
| 13 | Lovable apps lack Google OAuth buttons | Add `<a href="/auth/google/start">` — Gadget has it built-in |
| 14 | Google OAuth button using `onClick`/`fetch` instead of `<a>` | Must use `<a>` tag for full-page redirect — OAuth requires server-side flow |

## See Also

- [auth-translation.md](auth-translation.md) - Detailed auth patterns
- [data-translation.md](data-translation.md) - Detailed query/mutation patterns
- [typescript-fixes.md](typescript-fixes.md) - TypeScript-specific issues
