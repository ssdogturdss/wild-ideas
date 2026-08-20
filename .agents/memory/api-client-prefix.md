---
name: API client URL prefix
description: Orval-generated API client URLs already include the /api prefix from the OpenAPI spec server URL
---

# API Client URL Prefix

## Rule
The orval-generated client in `lib/api-client-react/src/generated/api.ts` already includes the `/api` prefix in every URL (e.g. `/api/auth/login`, `/api/auth/me`). Do NOT call `setBaseUrl("/api")` — this would produce `/api/api/auth/login` (double prefix → 404).

**How to apply:** Remove any `setBaseUrl("/api")` call from `App.tsx`. The generated URLs work as-is for browser-side fetches since the proxy routes `/api/*` correctly.

**Why:** The OpenAPI spec has `servers: [{url: /api}]` which orval prepends to all generated path URLs.
