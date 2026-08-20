---
name: pnpm onlyBuiltDependencies
description: How to allow native build scripts for npm packages in this workspace
---

## Problem

pnpm v10 blocks all package build scripts by default for security. Native packages (node-pty, canvas, etc.) need to run `node-gyp rebuild` during install. Without explicit allowlisting they silently skip the build, then fail at runtime.

`pnpm approve-builds` is interactive (terminal UI) and cannot be scripted.

## Solution

Add the package name to `onlyBuiltDependencies` in `pnpm-workspace.yaml`:

```yaml
onlyBuiltDependencies:
  - '@swc/core'
  - esbuild
  - node-pty   # ← add native packages here
  - unrs-resolver
```

Then run `pnpm install` to trigger the build.

**Why:** This is the pnpm v10 documented approach for workspace-wide build script allowlisting.

**How to apply:** Whenever installing a native npm package (anything with a `binding.gyp` or `install` script), add it here before installing.

## Note on Replit environment

Even with onlyBuiltDependencies set, node-pty v1.1.0 fails on Replit because `python3-dev` and `g++` header files are not available in the NixOS sandbox. Use child_process.spawn as a fallback (see ide-terminal.md).
