# mcp-builder

A Claude Code skill for building TypeScript **MCP (Model Context Protocol)**
servers. Not generic best practice — it encodes the exact conventions and
war stories from building and debugging four real servers (`infra`,
`calendar`, `books`, `weather`) in a shared multi-server repo, including a
real correctness bug (a silently-broken conflict gate) that was only caught
because the skill's own verification step wasn't skipped.

## Usage

Just describe the server you want in normal language — the skill triggers on
requests like:

```
add an MCP server for <thing>
build an MCP tool that does <X>
create an MCP server exposing <API/service>
```

If your request is ambiguous on scope (read-only vs mutating, what backs it,
whether the upstream API actually covers everything you asked for), the
skill will ask one tight clarifying question rather than guess and build the
wrong shape.

## How it works

The skill walks through a fixed sequence and won't skip the last step even
if everything looks fine:

1. **Find or scaffold the repo** — detects the shared-repo layout (one root
   `package.json`/`tsconfig.json`, servers under `src/<name>/`, one build) or
   creates it if this is the first server.
2. **Clarify scope** — read-only vs mutating, what backs it (keyless public
   API / API-key / OAuth), and whether the request implies capability the
   backend doesn't actually have (the "search books/weather/countries" via a
   books-only API case this was built from).
3. **Design the tool surface** — one responsibility per tool (composable,
   chainable), a prompt to chain them when there's a common workflow, trimmed
   API responses, passthrough pagination.
4. **Implement** — the McpServer/zod/stdio boilerplate, structured
   `isError` responses instead of throws, `AbortController` timeouts on
   external calls, the conflict-detection + idempotency pattern for
   mutating tools (with the exact bug to avoid, verbatim), and the
   pluggable-backend pattern for servers that might later point at a real
   integration.
5. **Wire up scripts and docs** — `package.json` bin/scripts, per-server
   README template, root README table + layout update.
6. **Build and verify — never skipped** — `npm run build`, a raw-JSON-RPC
   smoke test, explicit edge-case tests (especially the optional-field-
   omitted case, where the real bug hid), and the MCP Inspector for
   interactive checks. Only reports done after this passes.
7. **Optional OAuth setup** — for servers that need to authenticate against
   a real user account (e.g. a real calendar), a one-time local
   refresh-token helper script, not interactive auth inside the server
   itself.

## Requirements

- Node.js (used with v22 in the source repo; anything with native `fetch`
  and top-level ESM works).
- `@modelcontextprotocol/sdk`, `zod`, and (dev) `tsx`/`typescript` — the
  skill scaffolds these into `package.json` if the repo doesn't exist yet.
- The MCP Inspector (`@modelcontextprotocol/inspector`) is used via `npx`,
  no separate install needed.
- Scope: **TypeScript only.** Python MCP SDK (FastMCP) conventions are not
  covered — this skill only encodes patterns that were actually built and
  tested.

## Installation

```bash
cp -r skills/mcp-builder ~/.claude/skills/
# or symlink:
ln -s $(pwd)/skills/mcp-builder ~/.claude/skills/mcp-builder
```

Restart Claude Code after installation.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The step-by-step build instructions, code templates, and the concrete bug this skill was written to prevent repeating |
| `README.md` | This file |

## Source

Derived from a live session building and hardening the `mcp-servers` repo:
`infra` (read-only starter), `calendar` (mutating, conflict-gated,
idempotent, with an optional Google Calendar backend), `books` (OpenLibrary
search, correctly scoped after an initial over-broad request), and `weather`
(Open-Meteo, geocode-then-forecast).
