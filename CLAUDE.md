# CLAUDE.md

Typed TypeScript client SDK for `webgraph_api` (a knowledge-graph service for web
apps). Published to GitHub Packages as `@sudobility/webgraph_client`.

## Stack

- TypeScript (strict, all `strict*` + `noUnused*` flags on), ESM only
  (`"type": "module"`).
- Bun is the package manager and script runner; Vitest for tests; ESLint
  (flat config) + Prettier.
- Zero runtime dependencies — the SDK is a thin wrapper over `fetch`.

## Commands (verified)

```sh
bun install
bun run typecheck    # bunx tsc --noEmit
bun run lint         # bunx eslint src/
bun run test:unit    # bunx vitest run --passWithNoTests
bun run build        # bunx tsc -p tsconfig.build.json  → dist/
bun run verify       # typecheck + lint + test + build, in that order
```

Run `bun install` before lint/test — ESLint fails with `ERR_MODULE_NOT_FOUND`
if `node_modules` is missing. `verify` is the full pre-publish gate.

## Architecture

- `src/WebgraphClient.ts` — the `WebgraphClient` class. One private `request<T>()`
  helper does all HTTP: prepends `baseUrl` (trailing slash stripped in the
  constructor), sets `Content-Type: application/json` and `X-Api-Key: <apiKey>`.
  Public methods (`createApp`, `observe`, `route`, `nextStep`, `graph`, `plan`,
  `replan`, `proxyGet`) each just build a path/body and call `request`.
- `src/types.ts` — all request/response interfaces. Rich doc comments here encode
  server semantics (redaction, view identity, plan `source` modes); keep them in
  sync with the service.
- `src/index.ts` — public barrel; re-exports the class and types only. Keep the
  public surface defined here.
- `src/WebgraphClient.test.ts` — Vitest. Tests inject a mock via `fetchImpl` and
  assert on URL/headers/body; no network.

## Conventions

- Imports use explicit `.js` extensions (ESM/bundler resolution) even for `.ts`
  source — required, do not drop them.
- `fetchImpl` is dependency-injected so the SDK is testable without network;
  never call global `fetch` directly in the client.
- Prettier: single quotes, semicolons, `printWidth` 80, `arrowParens: avoid`,
  `trailingComma: es5`.
- Server-side only — the constructor holds a full-access API key; never ship to
  a browser.
- `endpoint` args (e.g. in `proxyGet`) are closed string unions, not free
  strings, so callers cannot steer the request path. Preserve that.

## Gotchas

- `createApp` returns a canonical `app.key` that may differ from the key sent;
  always use the returned key for later calls.
- `tsconfig.json` (editor/typecheck) includes tests; `tsconfig.build.json`
  excludes `*.test.ts` / `*.spec.ts` from the emitted `dist/`.
- `prepublishOnly` runs a safety check from a sibling `../workflows/` repo if
  present, then rebuilds. CI publishes via `johnqh/workflows` with
  `publish-target: github`.
