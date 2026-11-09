# @sudobility/webgraph_client

TypeScript client SDK for [`webgraph_api`](https://github.com/johnqh/webgraph_api) —
the knowledge-graph service for web applications. It records what a web app's
views look like and how they connect, then plans routes and next steps between
them.

A thin, typed wrapper over the service's HTTP endpoints. The network client is
dependency-injected (no direct `fetch` call baked in), so it is fully testable
without network access.

## Install

Published to GitHub Packages under the `@sudobility` scope. `.npmrc` already
points that scope at `https://npm.pkg.github.com`; authenticating to GitHub
Packages may require a token with `read:packages`.

```sh
bun add @sudobility/webgraph_client
# or: npm install @sudobility/webgraph_client
```

## Usage

```ts
import { WebgraphClient } from '@sudobility/webgraph_client';

const client = new WebgraphClient({
  baseUrl: 'https://your-webgraph-api',
  apiKey: process.env.WEBGRAPH_API_KEY!,
  // fetchImpl?: optional fetch override, injected for testing
});

// Register (or resolve) an app. Use the RETURNED app.key for later calls —
// it is canonical and may differ from the key you sent.
const { app } = await client.createApp({ key: 'myapp', host: 'shop.com' });

// Report a view the caller is looking at.
await client.observe(app.key, { urlPath: '/', regions: [] });

// Ask for a route, the single next hop, or a goal-driven plan.
await client.route(app.key, { to: { urlPath: '/checkout' } });
await client.nextStep(app.key, { to: { urlPath: '/checkout' } });
await client.plan(app.key, { goal: 'add to cart' });
```

### Methods

- `createApp(body)` — register/resolve an app; always send `host` (identity, not
  decoration — and no API traffic is captured without it). Use the returned
  `app.key` afterwards.
- `observe(appKey, body)` — report a view: URL path, region tree, visible links,
  how the caller got there, and optionally `networkRequests` (the service
  redacts values at ingest, storing only shapes).
- `route(appKey, { from?, to, maxDepth? })` — full ordered route between views.
- `nextStep(appKey, { from?, to })` — the single next hop, for callers that
  re-plan as they go (loop: observe → nextStep → execute → observe).
- `graph(appKey)` — views and transitions, for visualization.
- `plan(appKey, body)` — goal-driven plan; may return `source: 'graph-fallback'`
  (treat as a weaker plan, not an error).
- `replan(appKey, body)` — re-plan after a failure, reporting what failed.
- `proxyGet(appKey, endpoint, search)` — forward a raw query string to a read
  endpoint (`graph` | `route` | `next-step`) unchanged, for API gateways.

See [`src/types.ts`](src/types.ts) for the full request/response types.

## Server-side only

The SDK takes an API key in its constructor, so it is meant to run on a server.
**Do not use it in a browser.** The key grants full read and write access to an
app's graph; shipping it to a browser hands that to every visitor. Frontends
should call their own backend, which holds the key and proxies to `webgraph_api`.

## Development

Requires [Bun](https://bun.sh). TypeScript (strict). Tests run on Vitest.

```sh
bun install          # install dependencies
bun run typecheck    # tsc --noEmit
bun run lint         # eslint src/
bun run test:unit    # vitest run
bun run build        # emit dist/ via tsconfig.build.json
bun run verify       # typecheck + lint + test + build (the full gate)
```

## License

BUSL-1.1
