# Frontend Setup

Gadget can deploy any Vite app. The goal is to move the Lovable frontend into Gadget's `web/` directory with minimal changes.

## Replace Gadget's Default Frontend

Delete Gadget's existing frontend files — everything that will be replaced by the Lovable app. The specifics depend on what the Gadget app was scaffolded with, but typically includes existing routes, components, stylesheets, and any SSR configuration files.

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

## HTML Entry Point

Update `index.html` to point to the new location:
```html
<script type="module" src="/web/main.tsx"></script>
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
- Data fetching libraries that Gadget hooks replace (e.g., `@tanstack/react-query`)

**Add** (if not already present):
- `@gadgetinc/react` — Gadget's React hooks

**Keep** from Lovable:
- `react`, `react-dom`
- `react-router-dom`
- All UI packages (Radix, shadcn, Tailwind, etc.)
- Utility libraries (`date-fns`, `lucide-react`, `zod`, `sonner`, etc.)

## See Also

- [auth-translation.md](auth-translation.md) - Replacing auth calls
- [data-translation.md](data-translation.md) - Replacing data queries and mutations
