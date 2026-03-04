# lovable-to-gadget

A [skill](https://npmjs.com/package/skills) for migrating Lovable-generated React SPAs to the [Gadget](https://gadget.dev) platform.

## What it does

Guides a coding AI agent through migrating a Lovable app's frontend into a Gadget app, replacing the existing backend (Supabase, Firebase, etc.) with Gadget models, actions, and `@gadgetinc/react` hooks. The frontend code — components, pages, styling, routing — is preserved with minimal changes.

## Getting started

### 1. Sign up for Gadget

Create a free account at [gadget.dev](https://gadget.dev).

### 2. Create your Gadget app

Create a new app in the Gadget dashboard — this will be the target for your Lovable port. Choose the **Web app** template (with auth if your Lovable app has authentication). The default frontend will be replaced during migration.

### 3. Pull your Gadget app locally

Install the [Gadget CLI](https://docs.gadget.dev/reference/ggt) and pull your app:

```bash
npx ggt@latest dev ~/my-gadget-app --app=your-app-slug
```

This syncs your Gadget app to a local directory and watches for changes.

### 4. Install and use the skill

From your Gadget app directory, install the skill:

```bash
cd ~/my-gadget-app
npx skills add infiton/lovable-to-gadget
```

Then open the project in your AI coding agent (e.g., Claude Code) and ask it to migrate your Lovable app. The agent will use the skill's reference files to guide the migration.

## Topics covered

| Reference | What it covers |
|---|---|
| [Overview](skills/lovable-to-gadget/references/overview.md) | Migration approach, phases, and analysis checklist |
| [Models & Actions](skills/lovable-to-gadget/references/models-and-actions.md) | Field type mapping, CRUD actions, session linking |
| [Access Control](skills/lovable-to-gadget/references/access-control.md) | Permissions, Gelly filters, row-level security |
| [Frontend Setup](skills/lovable-to-gadget/references/frontend-setup.md) | Moving the Lovable frontend into Gadget's `web/` directory |
| [Auth Translation](skills/lovable-to-gadget/references/auth-translation.md) | Sign in/up/out, Google OAuth, auth guards, email verification |
| [Data Translation](skills/lovable-to-gadget/references/data-translation.md) | Queries, mutations, file uploads, relationship linking, pagination |
| [TypeScript Fixes](skills/lovable-to-gadget/references/typescript-fixes.md) | Nullable fields, generated types, GadgetRecord casting |
| [Pitfalls](skills/lovable-to-gadget/references/pitfalls.md) | Quick-reference table of 15 common mistakes |
| [Checklist](skills/lovable-to-gadget/references/checklist.md) | Complete migration checklist with verification steps |

## License

MIT
