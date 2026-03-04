# Frontend Setup

Gadget can deploy any Vite app. The goal is to move the Lovable frontend into Gadget's `web/` directory with minimal changes.

## Replace Gadget's Default Frontend

Delete Gadget's existing frontend files that will be replaced by the Lovable app. **Keep `web/api.ts`** — this is the auto-generated Gadget API client and must not be deleted or overwritten. Delete everything else in `web/` (routes, components, stylesheets, entry points) and any SSR configuration files at the project root.

All frontend code in Gadget lives in the `web/` directory.

## Copy Lovable Source Files

Copy from Lovable `src/` → Gadget `web/`:
```
src/App.tsx           → web/App.tsx
src/main.tsx          → web/main.tsx
src/index.css         → web/index.css
src/lib/              → web/lib/
src/types/            → web/types/
src/hooks/            → web/hooks/
src/assets/           → web/assets/
src/pages/            → web/pages/
src/components/       → web/components/
```

## Vite Configuration

Gadget requires that the Vite config file is named `vite.config.mts` (TypeScript) or `vite.config.mjs` (JavaScript). The config must include Gadget's Vite plugin, which handles API client generation:

```ts
import { gadget } from "gadget-server/vite";

// Add gadget() to your plugins array alongside your existing Vite plugins
export default defineConfig({
  plugins: [gadget(), /* ...your existing plugins */],
  resolve: {
    alias: { "@": path.resolve(__dirname, "./web") },
  },
});
```

Key points:
- The `gadget()` plugin is required — it generates the API client
- Keep your existing Vite plugins (React, etc.) — just add `gadget()` alongside them
- Update the `@` path alias from `./src` to `./web`
- Remove any SSR-specific plugins if present (Gadget deploys SPAs directly)

## Import Path Alias

After moving `src/` to `web/`, all `@/` imports will break if the path alias is not updated. Ensure the Vite config alias and `tsconfig.json` paths both point to `./web`:

```json
// tsconfig.json — update paths
{
  "compilerOptions": {
    "paths": { "@/*": ["./web/*"] }
  }
}
```

## HTML Entry Point

Update `index.html` to point to the new location:
```html
<script type="module" src="/web/main.tsx"></script>
```

## PostCSS Configuration

If using Tailwind v3, ensure a `postcss.config.js` exists at the project root (copy from the Lovable app if needed):
```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

## Tailwind Configuration

If using Tailwind, update content paths from `./src/**` to `./web/**`:
```ts
content: [
  "./web/pages/**/*.{ts,tsx}",
  "./web/components/**/*.{ts,tsx}",
  "./web/App.tsx",
  "./web/main.tsx",
],
```

## shadcn/ui Configuration

If using shadcn/ui, update `components.json` aliases and paths:
```json
{
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "web/index.css"
  }
}
```

## Main Entry Point

Replace backend-specific providers with Gadget's provider in `web/main.tsx`:

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import { Provider as GadgetProvider } from "@gadgetinc/react";
import { api } from "./api";
import App from "./App";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <GadgetProvider api={api}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </GadgetProvider>
  </StrictMode>
);
```

The `GadgetProvider` replaces whatever backend-specific context providers the Lovable app used (e.g., Supabase client provider, TanStack QueryClientProvider).

## API Client

Gadget auto-generates a typed API client. Keep the existing `web/api.ts`:
```ts
import { Client } from "@gadget-client/your-app-slug";
export const api = new Client();
```

## Package Dependencies

**Remove** (backend-specific, replaced by Gadget):
- Backend SDK packages (e.g., `@supabase/supabase-js`, `firebase`, etc.)
- Data fetching libraries **if only used for backend queries** (e.g., `@tanstack/react-query`) — audit usage first; if the app also uses them for third-party API calls, keep them

**Add** (if not already present):
- `@gadgetinc/react` — Gadget's React hooks

**Keep** from Lovable:
- `react`, `react-dom`
- `react-router-dom`
- All UI packages (Radix, shadcn, Tailwind, etc.)
- Utility libraries (`date-fns`, `lucide-react`, `zod`, `sonner`, etc.)

## Public Assets

If the Lovable app has files in `public/` (favicons, images, etc.), keep them in `public/` at the project root — Vite serves these as static assets regardless of the `web/` directory structure.

## Cleanup: Old Backend Files

After copying and before translating, delete the Lovable app's backend-specific files that were copied into `web/`:
- Backend client initialization (e.g., `web/integrations/supabase/`, `web/lib/supabase.ts`)
- Auto-generated backend types (e.g., `web/integrations/supabase/types.ts`)
- Auth context providers that wrap the old backend (e.g., `web/contexts/AuthContext.tsx`) — these are replaced by `GadgetProvider` and `useUser()`

## See Also

- [auth-translation.md](auth-translation.md) - Replacing auth calls
- [data-translation.md](data-translation.md) - Replacing data queries and mutations
