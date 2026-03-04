---
name: lovable-to-gadget
description: Migrate a Lovable-generated React SPA to the Gadget platform, replacing the existing backend (Supabase, Firebase, etc.) with Gadget models, actions, and @gadgetinc/react hooks
---

# Lovable to Gadget Migration

Gadget can deploy any Vite-based frontend. The goal of this migration is to move a Lovable app's frontend into a Gadget app with **as few frontend stack changes as possible**, while replacing the backend integration layer (database queries, auth, file storage) with Gadget equivalents.

**How to use:** Read individual reference files below for detailed patterns and examples. Start with the overview, then work through backend setup and frontend translation.

## Overview & Planning

- [references/overview.md](references/overview.md) - Migration approach, phases, and analysis checklist

## Backend Setup

- [references/models-and-actions.md](references/models-and-actions.md) - Creating Gadget models, actions, and field type mapping
- [references/access-control.md](references/access-control.md) - Permissions, Gelly filters, and row-level security

## Frontend Migration

- [references/frontend-setup.md](references/frontend-setup.md) - Moving the Lovable frontend into Gadget's `web/` directory
- [references/auth-translation.md](references/auth-translation.md) - Translating auth (sign in/up/out, OAuth, guards, sessions)
- [references/data-translation.md](references/data-translation.md) - Translating queries, mutations, file uploads, and relationships

## Finishing Up

- [references/typescript-fixes.md](references/typescript-fixes.md) - Common TypeScript issues with Gadget's generated types
- [references/pitfalls.md](references/pitfalls.md) - Quick-reference table of common mistakes and their fixes
- [references/checklist.md](references/checklist.md) - Complete migration checklist
