---
name: IDE Terminal Implementation
description: How the browser terminal works in the IDE without node-pty; WS auth pattern
---

## Terminal without node-pty

node-pty requires native compilation (python3-dev, g++, node-gyp). These are not available on Replit's NixOS sandbox — the build fails with "gyp ERR! not ok".

**Solution:** Use `child_process.spawn('/bin/bash', ['-c', command])` with stdio piped. Implement a REPL loop in the WebSocket handler:
- User input characters are buffered line-by-line with local echo sent back over WS
- On Enter: run the accumulated line as a bash command
- Track `cwd` across commands; handle `cd` natively before spawning
- Handle `clear`, `exit`, backspace (0x7f), Ctrl+C (SIGINT to running process)

**Why:** This avoids any native build while providing a functional command-execution terminal. Interactive TUI programs (vim, htop) won't work, but all standard shell commands do.

**How to apply:** Any time a PTY-based terminal is needed in Node.js and node-pty compilation fails, fall back to this REPL pattern. The key file is `artifacts/api-server/src/terminal-ws.ts`.

## WebSocket terminal auth

Browser WebSocket connections do carry session cookies (same-origin), but the `http.Server` upgrade event fires before Express session middleware runs, so `req.session` is not populated.

**Solution:** Issue a one-time token via an authenticated REST endpoint:
1. Client: `POST /api/ide/ws-token` (session-authenticated) → `{ token: "uuid" }`
2. Client: `new WebSocket('/api/ide/terminal?token=<uuid>')`
3. Server: validate token from URL query string on upgrade; delete token immediately (one-use); tokens expire in 60 seconds

**Why:** Avoids the complexity of running session middleware on raw upgrade events, while still preventing unauthenticated terminal access.

**How to apply:** Use this pattern for any WebSocket endpoint that requires auth in this Express + ws setup.
