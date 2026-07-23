---
name: mcp-builder
description: Scaffolds and implements new Model Context Protocol (MCP) servers in TypeScript, following the conventions validated in the mcp-servers repo (infra, calendar, books, weather) — shared build, per-server tools/resources/prompts, structured errors, pluggable backends, idempotency/conflict gating, and mandatory build+smoke-test verification before declaring done. Use this skill when the user asks to "add an MCP server", "build an MCP tool for X", "create an MCP server that does Y", or wants to expose an API/service/data source to an LLM agent via MCP.
---

# MCP Builder

Builds TypeScript MCP servers the way this workspace has already validated them —
not generic best practice, but the exact shape that was designed, built, and
debugged end-to-end in the `mcp-servers` repo. Follow these steps in order;
don't skip the verification step even under time pressure — it is what caught
a real correctness bug (see "War story" below) that would otherwise have
shipped silently.

## Delegating to the SDLC personas

This skill owns the MCP-specific know-how — repo layout, tool-surface design,
conventions, docs, OAuth. It borrows three things from the `sdlc` skill via
the **Agent tool**/`AskUserQuestion` instead of reinventing them: the
up-front requirements-questioning discipline (Step 1), implementation
(Step 3), and review (Step 5) — so scope gets the same "confirm before you
build" gate, and code gets the same discipline and checklists, as any other
sdlc-built project. This is a lightweight borrow of the personas' habits and
tools, not a full sdlc run — no `docs/sdlc/` artifact trail or `STATE.md` is
created.

- **Gather requirements → `subagent_type: sdlc-product-manager` (optional,
  larger builds only).** For a single well-scoped server, do Step 1 inline —
  it's fast and the persona would add nothing. Delegate instead when the
  request spans multiple servers, a non-obvious domain, or several
  stakeholders' worth of "what should this actually do" — the PM persona's
  whole job is scope + acceptance criteria; reuse it rather than
  re-improvising. Give it the raw user request and get back scope +
  acceptance criteria to feed Step 1's confirmation gate.
- **Implement → `subagent_type: sdlc-developer`.** Give it: the target repo
  path, the concrete tool surface decided in Steps 1–2 (names, schemas,
  read-only vs mutating), and a pointer to **this file** (`skills/mcp-builder/
  SKILL.md`) as the spec to follow for server shape, error handling, timeouts,
  and the conflict/idempotency pattern in Step 3. Do not hand it vague intent —
  hand it the resolved design.
- **Verify → `sdlc-reviewer` + `sdlc-secops`, in parallel.** After the
  Developer reports back, spawn both in one message: Reviewer checks
  correctness against Step 5's test list (required-field-omitted, conflict
  path with the optional key genuinely omitted, force/override, idempotent
  retry); SecOps checks secrets/env-var handling, injection via tool inputs,
  and the timeout/DoS surface on external calls. Route findings back to
  `sdlc-developer` to fix; bound this to 3 rounds same as sdlc, then escalate
  to the user with the open findings.
- Steps 0–2, 4, and 6 (scaffolding, scope, wiring, OAuth) stay owned by this
  skill and run inline — they're MCP-domain judgment calls, not implementation
  or QA work.
- If the `sdlc-developer`/`sdlc-reviewer`/`sdlc-secops` subagents aren't
  installed (see `skills/sdlc/README.md#install`), fall back to doing Steps 3
  and 5 inline as before — don't block on their availability.

## Step 0 — Find or create the shared repo

This skill assumes a **shared multi-server repo**: one root `package.json` +
`tsconfig.json`, each server self-contained under `src/<name>/server.ts` +
`src/<name>/README.md`, all compiling to `dist/<name>/` via one `tsc` build.

1. Look for this layout in the current working directory (root `package.json`
   with `@modelcontextprotocol/sdk` as a dependency, a `src/` with sibling
   server folders). If found, **match its existing conventions exactly**
   (indentation, error-handling style, whether it uses `registerTool` vs the
   older `tool()` API, etc.) rather than the template below verbatim.
2. If nothing exists yet, scaffold the shared skeleton first:
   - `package.json`: `"type": "module"`, deps `@modelcontextprotocol/sdk` +
     `zod`, devDeps `@types/node` + `tsx` + `typescript`.
   - `tsconfig.json`:
     ```json
     {
       "compilerOptions": {
         "target": "ES2022",
         "module": "Node16",
         "moduleResolution": "Node16",
         "outDir": "./dist",
         "rootDir": "./src",
         "strict": true,
         "esModuleInterop": true,
         "skipLibCheck": true,
         "declaration": true,
         "sourceMap": true
       },
       "include": ["src/**/*"]
     }
     ```
   - Root `README.md` with a table of servers (path, one-line description)
     and a `## Layout` tree — every new server gets a row and a tree entry.

## Step 1 — Gather requirements and confirm scope before writing code

Borrow sdlc's kickoff discipline here: **ask clarifying questions in one
focused batch up front** (via `AskUserQuestion`) rather than guessing and
building the wrong shape, and **don't start Step 2's design until scope is
actually confirmed** — this is the same "gate before you invest" pattern
sdlc uses before Analyze → Plan.

1. **Ask, in one batch, whatever of these is not already clear from the
   request:**
   - **What's the actual goal?** What should the model be able to *do* once
     this server exists — what question should it answer or action should it
     take? If the request names a feature but not the underlying need,
     ask; building the literal ask instead of the actual need is the
     failure mode this step exists to prevent.
   - **Read-only or mutating?** Mutating tools (create/update/delete against
     a real system) need conflict-checking, idempotency, and an explicit
     `force` escape hatch (see Step 3). Read-only tools don't.
   - **What backs it?** A free/keyless public API (build it directly, no
     config needed), an API requiring a key (env var, document it), or a
     stateful system needing OAuth (see Step 3's pluggable-backend pattern
     and Step 6's auth helper)?
   - **Does the upstream API only cover part of what the user described?**
     Don't build tools that imply capability the backend doesn't have.
     (Example from this repo: a user asked for a "search" server covering
     "books, weather, countries & towns" via OpenLibrary — that API is
     books-only. The fix was building a correctly-scoped `books` server and
     flagging the rest needed separate backends, not stretching one tool's
     schema to imply coverage it doesn't have.)
   For a request that spans multiple servers or an unfamiliar domain, delegate
   this gathering to `sdlc-product-manager` instead of improvising (see
   "Delegating to the SDLC personas" above) and fold its scope/acceptance
   criteria into the question batch or the confirmation below.
2. **Naming**: name the package/directory after the **domain**, not a generic
   verb — `books`, not `search`; `weather`, not `data`. Tool names describe
   the **action** and can differ from the package name (`search_books` inside
   the `books` server).
3. **Confirm before proceeding.** Once scope is resolved, state back in 2–3
   sentences what you're about to build (server name, read-only/mutating,
   backend, the tools it will expose) and get explicit confirmation before
   moving to Step 2 — same purpose as sdlc's "user confirms scope" gate.
   Skip this restatement only when the request was already fully
   unambiguous (e.g. "add a read-only tool wrapping the free FooBar API's
   `/status` endpoint").

## Step 2 — Design the tool surface

- **One responsibility per tool.** Don't collapse a workflow into one opaque
  call. The `calendar` server has 5 separate tools
  (`list_events`/`find_free_slots`/`check_conflicts`/`schedule_interview`/`cancel_event`)
  instead of one `book_meeting` that does everything internally — this lets
  the model see intermediate state (what's free, what conflicts) and make
  decisions, and lets you unit-test each step in isolation. Same logic split
  `weather` into `geocode_location` + `get_weather` rather than one
  "weather for a city name" tool, because a place name can resolve to
  multiple real locations and the model should see the candidates.
- **Add a chaining prompt** (`registerPrompt`) when a multi-tool workflow is
  common — it documents the intended sequence and nudges the model toward
  it. See `schedule_interview_flow` and `weather_briefing` for the pattern:
  numbered steps, each naming the tool to call next, plus explicit guardrails
  ("do NOT force without approval").
- **Trim external API responses.** Don't hand the model a raw upstream
  payload. Reshape to only the fields a caller needs (see `books`: OpenLibrary
  docs are large/nested; the server requests `fields=` and reshapes to
  ~9 flat fields) and translate opaque codes to text where cheap (see
  `weather`: WMO weather codes → `"Overcast"` instead of `3`).
- **Pagination**: if the upstream API paginates, expose the same params
  (`limit`/`page`) rather than inventing new semantics — passthrough matches
  upstream `numFound`/counts exactly and avoids off-by-one translation bugs.

## Step 3 — Implement the server

**Delegate this step to `sdlc-developer`** (see "Delegating to the SDLC
personas" above) — give it the resolved tool surface from Steps 1–2 plus this
section as the spec. Fall back to implementing inline only if that subagent
isn't installed.

Core shape, every server:

```ts
#!/usr/bin/env node
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "<name>-mcp-server", version: "1.0.0" });

server.registerTool(
  "tool_name",
  {
    title: "Human title",
    description: "What it does. State read-only vs mutating explicitly.",
    inputSchema: { /* zod fields, each with .describe(...) */ },
  },
  async (input) => {
    // validate business rules tool-schema can't express, then act
    return toolText({ ... });   // success
    // or toolError("message") // structured failure
  }
);

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("<name>-mcp-server running on stdio"); // stderr — see below
}
main().catch((err) => {
  console.error("Fatal error starting <name>-mcp-server:", err);
  process.exit(1);
});
```

Non-negotiable details, each backed by a concrete failure mode seen in this
repo:

- **Logs go to `stderr` (`console.error`), never `stdout`.** stdout is the
  JSON-RPC channel; anything else written there corrupts framing. This is
  the single most common first bug when writing a new server.
- **Validate with zod, not ad-hoc checks**, and `.describe()` every field —
  the schema is both runtime validation *and* the signature the model sees.
  Cross-field rules zod can't express (e.g. `end` after `start`) go in the
  handler body as an explicit `toolError(...)` return.
- **Structured errors, not throws.** `isError: true` in the tool result lets
  the client/model reason about failure and retry or ask for approval;
  a thrown exception just looks like a broken tool.
  ```ts
  function toolError(text: string) {
    return { isError: true, content: [{ type: "text" as const, text }] };
  }
  function toolText(payload: unknown) {
    return { content: [{ type: "text" as const, text: JSON.stringify(payload, null, 2) }] };
  }
  ```
- **Timeout every external HTTP call** with `AbortController` (8s is the
  precedent in this repo) so a hung upstream can't hang the tool call:
  ```ts
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 8_000);
  try {
    const res = await fetch(url, { signal: controller.signal });
    if (!res.ok) throw new Error(`Upstream returned HTTP ${res.status}`);
    return await res.json();
  } finally {
    clearTimeout(timeout);
  }
  ```
- **Defer startup work (auth, backend selection) into `main()`**, not a
  top-level `await`, so a misconfiguration fails through the same
  `main().catch(...)` path as every other startup error — one consistent
  "Fatal error starting..." message, not an unhandled top-level stack trace.

### Mutating tools: conflict + idempotency pattern

For any tool that creates/changes state (see `schedule_interview`):

1. **Check for conflicts first**, and refuse by default; require an explicit
   `force: boolean` (default `false`) to override. Never silently clobber.
2. **Accept an optional `idempotencyKey`** so a retried call returns the
   existing result instead of duplicating. Detect "this is a retry of my own
   prior booking" *before* the conflict check — a retry legitimately
   overlaps itself; that's not a conflict.
3. Put the actual dedup-and-write in one atomic step in the backend if the
   store allows it (in-memory: trivial). If the backend can only do
   check-then-insert (most real APIs), **say so explicitly** in a doc
   comment/README concurrency note — don't imply a guarantee that isn't
   there.

**War story — read before implementing this pattern:** the first cut of the
conflict gate had this exact bug:

```ts
// WRONG: undefined !== undefined evaluates to false
const conflicts = overlapping.filter((ev) => ev.idempotencyKey !== idempotencyKey);
```

When a booking request omits `idempotencyKey` (the common, keyless case) and
the existing conflicting event also has none, `undefined !== undefined` is
`false` — so the real conflict gets filtered *out* of `conflicts`, and the
tool silently double-books. `check_conflicts` (a separate read-only tool)
correctly reported the overlap; `schedule_interview` didn't use the same
logic and got it wrong. This was only caught because Step 5's smoke test
exercised the exact keyless scenario the README's own example used — not
just a scenario with an idempotency key. The fix: only exclude an
overlapping event when *this* request actually carries a key that matches
it:

```ts
const conflicts = idempotencyKey
  ? overlapping.filter((ev) => ev.idempotencyKey !== idempotencyKey)
  : overlapping;
```

**Lesson for this skill: always write a test case with the optional field
*omitted*, not just one with it populated.** Optional-field bugs hide
exactly there.

### Pluggable backend for stateful/real-integration servers

When a server might swap a demo backend for a real one (in-memory ↔ Google
Calendar, mock ↔ real API), extract the seam into its own interface file
(`backend.ts`), not the main server file:

```ts
// src/<name>/backend.ts
export interface XBackend {
  /* only what the tools need, nothing they don't */
}
export class InMemoryXBackend implements XBackend { /* zero-dependency default */ }
```

Then `src/<name>/real-backend.ts` implements the same interface against the
live API, and `server.ts` selects between them at startup via an env var
(`<NAME>_BACKEND=real`), failing fast with a clear message if required
credentials are missing — never falling back silently. See `calendar`'s
`backend.ts` / `google-backend.ts` / the `CALENDAR_BACKEND` switch in
`server.ts` for the full worked example, including a documented, honest
concurrency caveat ("this backend's idempotency check is check-then-insert,
not atomic, because the underlying API has no compare-and-swap").

## Step 4 — Wire up scripts and docs

For a server named `<name>`:

**Root `package.json`** — add to `bin`, and four scripts:
```jsonc
"bin": { "<name>-mcp-server": "dist/<name>/server.js" },
"scripts": {
  "start:<name>":   "node dist/<name>/server.js",
  "dev:<name>":     "tsx watch src/<name>/server.ts",
  "inspect:<name>": "npx @modelcontextprotocol/inspector tsx src/<name>/server.ts"
}
```

**`src/<name>/README.md`** — mirror the structure already used by every
server in this repo: What is it → table of tools/resources/prompts with
one-line purpose each → How to use (build, Inspector, `claude mcp add`
snippet, a raw-JSON-RPC quick sanity check with the *expected* output stated)
→ Design notes (the "why" behind non-obvious choices) → Layout.

**Root `README.md`** — add a row to the servers table and a line to the
`## Layout` tree.

## Step 5 — Build and verify (do not skip this)

This is not optional polish — it is how real bugs get caught before a user
does. **Delegate the review portion to `sdlc-reviewer` + `sdlc-secops` in
parallel** (see "Delegating to the SDLC personas" above) once build + smoke
test pass; route their findings back to `sdlc-developer`. Fall back to doing
this inline only if those subagents aren't installed.

1. `npm run build` from repo root. Must be clean (no tsc errors) before
   anything else.
2. **Raw JSON-RPC smoke test** — this is faster than the Inspector for
   scripted verification and is what caught the idempotency bug above:
   ```bash
   printf '%s\n' \
   '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"c","version":"0"}}}' \
   '{"jsonrpc":"2.0","method":"notifications/initialized"}' \
   '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' \
   '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"<tool>","arguments":{...}}}' \
   | node dist/<name>/server.js
   ```
3. **Test the cases that break things, not just the happy path**:
   - required-field-omitted validation errors,
   - a mutating tool's conflict path *with the optional field genuinely
     omitted*, not just populated,
   - `force`/override paths,
   - idempotent retry (same key called twice → second call reports "already
     done", doesn't duplicate),
   - one real external-API call end-to-end if the server hits the network
     (don't just trust the mock).
4. **MCP Inspector** for interactive/visual verification —
   `npm run inspect:<name>` runs the TypeScript source directly via `tsx`
   (no build needed for this loop). Tabs: Tools / Resources / Prompts / Ping;
   bottom pane shows the stderr log stream. Use **Restart** after code edits;
   in-memory backends reset to their seed data on restart/reconnect — that's
   expected, not a bug.
5. Only after 1–4 pass, report the work as done. If you can't exercise a
   piece (e.g. an OAuth-gated real backend needing the user's own
   credentials), say so explicitly rather than claiming it was tested.

## Step 6 — Optional: real external-service auth (OAuth)

When a server needs to authenticate as the *user* against a real account
(not a service-to-service integration), follow the pattern in
`calendar`/`google-backend.ts` + `calendar/scripts/get-google-refresh-token.ts`:

1. A one-time local script does the installed-app OAuth flow: generate an
   auth URL, open a tiny local HTTP server on a loopback port to catch the
   redirect, exchange the code for tokens, print the long-lived
   `refresh_token` for the user to save as an env var. Never have the
   *server itself* do interactive OAuth — it runs headless under an MCP
   client.
2. Document every required env var in a table in the server's README
   (what it's for, required vs optional, default), plus the exact
   `claude mcp add <name> --scope user -e KEY=value -e KEY2=value -- node ...`
   invocation for registering it with a real value filled in per var.
3. At startup, validate all required env vars are present and fail with one
   clear message naming exactly what's missing and where to get it — this
   is the same "fail fast through `main().catch`" rule from Step 3, applied
   to config instead of business logic.

## Reference: registering a finished server

```bash
claude mcp add <name> --scope user -- node /absolute/path/to/dist/<name>/server.js
# with env vars:
claude mcp add <name> --scope user -e KEY=value -- node /absolute/path/to/dist/<name>/server.js
claude mcp list          # shows it + does a connection check
claude mcp get <name>    # shows resolved config
```
