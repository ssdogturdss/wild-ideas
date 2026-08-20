---
name: Replit proxy routing
description: How the Replit port-80 proxy forwards requests to artifact services — prefix is NOT stripped
---

# Replit Proxy Routing

## Rule
The Replit proxy routes `/api/*` → port 8080 WITHOUT stripping the `/api` prefix. Express receives the full path including `/api`.

**How to apply:** Always mount the API router at `app.use("/api", router)`. Routes in the router are written without the prefix (e.g. `router.post("/auth/login")`).

**Why:** Changing to `app.use(router)` breaks all routes because the router's paths don't include `/api`, so `/api/auth/login` doesn't match `/auth/login`.

## Verified
- `curl http://localhost:80/api/auth/login` → 200 when `app.use("/api", router)` ✅
- `curl http://localhost:80/api/auth/login` → 404 when `app.use(router)` ❌
