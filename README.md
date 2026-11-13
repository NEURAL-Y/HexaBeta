<div align="center">

<img src="https://raw.githubusercontent.com/NEURAL-Y/Byte-Plus/main/public/logo.png" alt="DualBridge" width="220"/>

# DualBridge

**Define once. Connect everywhere.**

A unified TypeScript framework for backend connectivity: one definition, every protocol, one policy.

![Status](https://img.shields.io/badge/status-under%20development-yellow)
![Language](https://img.shields.io/badge/TypeScript-first-3178c6)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

[Why](#why) • [Core Idea](#core-idea) • [Quick Start](#quick-start) • [Features](#features) • [Architecture](#architecture) • [Roadmap](#roadmap) • [Contributing](#contributing)

</div>

---

> **Status:** early design and development. The API below is the design target and may change. Nothing here is production-ready yet.

DualBridge is an open-source framework that fixes the connectivity layer of a backend. You write a capability once, and DualBridge exposes it over **HTTP, WebSocket and MCP** (gRPC and GraphQL later) with shared identity, shared policy, typed clients and reliable realtime built in, instead of stitching a separate library onto each protocol.

<a id="why"></a>
<details open>
<summary><b>Why?</b></summary>

<br>

Backends rarely speak one protocol. The same logic ends up re-exposed as REST, WebSocket events, MCP tools and RPC, and each exposure brings its own library, its own auth, its own limits and its own error format. Over time they drift apart.

Common pains DualBridge targets:

- **Same logic, many entry points.** Re-implementing handlers per protocol.
- **Contract drift.** Backend changes silently break frontends, SDKs and agents.
- **Realtime plumbing.** Reconnect, resume, dedupe and backpressure rebuilt in every project.
- **Policy drift.** Auth and rate limits differ between REST, WS and agent endpoints.
- **Fragmented observability.** No single trace across protocols.

DualBridge's answer: **one framework, one handler definition, one policy, every protocol.**

</details>

<a id="core-idea"></a>
<details open>
<summary><b>Core Idea</b></summary>

<br>

Every protocol is an **adapter** that normalizes traffic into one common context (`Ctx`). Everything after that is protocol-agnostic.

```text
HTTP ──┐
WS   ──┤
MCP  ──┼─► adapter ─► Ctx { client, proto, op, input, meta } ─► policy ─► handler
gRPC ──┤
GraphQL┘
```

What is unified:

| Unified | Meaning |
|---|---|
| **Identity** | One `client` object whichever protocol it came from |
| **Handlers** | Write the logic once, expose it on N protocols |
| **Policy** | Guard, limits and rules defined once |
| **Errors** | One error model mapped to HTTP status, WS frames, MCP codes |
| **Observability** | One audit and trace format, with `proto` as a label |

What is **not** reimplemented: wire protocols. Adapters wrap maintained libraries (`ws`, the MCP SDK, and so on). DualBridge owns the layer above them.

</details>

<a id="quick-start"></a>
<details open>
<summary><b>Quick Start</b></summary>

<br>

> Package not yet published. Usage shown is the planned API.

```bash
npm install dualbridge
```

```ts
import { bridge } from "dualbridge";

const app = bridge({
  protocols: ["http", "ws", "mcp"],
  cors: { origins: ["https://myapp.com"] },     // required, no open default
  guard: { default: "deny", trustProxy: ["10.0.0.5"] },
});

app.define("getOrder", {
  expose: {
    http: "GET /orders/:id",
    ws:   "order.get",
    mcp:  "get_order",
  },
  policy: { scope: "orders:read" },
  handler: async (ctx) => db.order(ctx.params.id),
});

app.start();
```

One definition gives you a REST route, a WebSocket event and an MCP tool, with the same guard, the same errors and one audit log.

Generate a typed client from the same definitions:

```bash
npx dualbridge client --out ./sdk
```

</details>

<a id="features"></a>
<details>
<summary><b>Features</b></summary>

<br>

| | |
|---|---|
| 🌉 **Multi-protocol from one definition** | HTTP, WebSocket and MCP first; gRPC and GraphQL via the same adapter contract |
| 🧾 **Typed clients** | Client SDKs generated from your definitions, so the contract is the source of truth |
| 🔄 **Reliable realtime** | Reconnect, resume after drop, dedupe and backpressure handled for you |
| 🔗 **Service-to-service client** | Timeouts, retries with idempotency, and trace propagation across hops |
| 🛡 **Private-by-default shield** | Internal routes are not exposed unless marked `public`; upstream URLs and keys stay hidden behind a proxy layer |
| 🚧 **IP guard + CORS** | Server-enforced IP and CIDR allowlist with default-deny, plus always-on CORS with explicit origins |
| 🧬 **YARA-style rules** | Developer-written rules (`strings`, `condition`, `meta`) inspecting request and response content, with report mode and log replay |
| ⚡ **Adaptive execution** | Cheap handlers run inline, heavy ones are offloaded to workers, overload is shed, chosen at runtime (the "semisynchronous" model) |
| 🧩 **Adapter contract** | Add a protocol by implementing a small interface |
| ⚙️ **Self-assigned middleware** | Secure defaults on out of the box, each individually switchable off in code |

</details>

<details>
<summary><b>Policy Layer</b></summary>

<br>

Policy is a plugin on top of the connectivity core and runs for every protocol in the same order:

```text
ipGuard → cors/preflight → rules → auth → handler
```

Example rule:

```text
rule sqli_basic {
  meta:      severity = "high", action = "block"
  scope:     route "/api/*"
  strings:
    $a = /union\s+select/i
    $b = "' or 1=1" nocase
  fields:    query, body
  condition: any of ($a, $b)
}
```

Design notes:

- Default-deny: no matching `allow` means blocked.
- Modes: `block | report | tag`, global or per rule.
- Inputs are normalized (URL decode, unicode, case) before matching.
- Pattern matching uses a ReDoS-safe engine with a bounded inspection size.
- Signature rules are defense in depth. Parameterized queries and real authentication remain the actual protection.
- The IP guard honors `trustProxy` so spoofed `X-Forwarded-For` headers cannot bypass it.
- Route and origin hiding reduces exposure but is not a substitute for auth. Browser DevTools can always see requests the browser makes.

</details>

<a id="architecture"></a>
<details>
<summary><b>Architecture</b></summary>

<br>

```text
                   app.define(...)
                        │
        ┌───────────────┴───────────────┐
        │        @dualbridge/core       │
        │  Ctx • pipeline • errors      │
        │  policy • audit • typed gen   │
        └───────────────┬───────────────┘
      ┌─────────┬───────┴───┬───────────┬──────────┐
      ▼         ▼           ▼           ▼          ▼
   HTTP       WebSocket    MCP        gRPC      GraphQL
  adapter     adapter     adapter    (later)    (later)
      │         │           │
      ▼         ▼           ▼
  Web apps • Mobile • AI agents • CLIs • Other services
```

Packages:

```text
@dualbridge/core     Ctx, pipeline, errors, audit
@dualbridge/http     HTTP adapter
@dualbridge/ws       WebSocket adapter
@dualbridge/mcp      MCP adapter
@dualbridge/client   typed client generator
dualbridge           meta-package: core + http + ws + mcp
```

Adapter contract:

```ts
interface Adapter {
  name: string;
  start(pipeline: Pipeline): Promise<void>;
  normalize(raw: unknown): Ctx;
  reply(ctx: Ctx, res: Result): void;
  stop(): Promise<void>;
}
```

Adapters load lazily, so only the protocols you configure are imported.

</details>

<details>
<summary><b>Philosophy</b></summary>

<br>

Connectivity should be one framework, not a pile of per-protocol libraries. Write your backend logic once, describe how it is exposed, and let the framework handle identity, policy, typing and reliability across every transport.

**Define once. Connect everywhere.**

</details>

<a id="roadmap"></a>
<details>
<summary><b>Roadmap</b></summary>

<br>

<details>
<summary><b>v0.1 — Core</b></summary>

<br>

`Ctx` and pipeline · error model · HTTP adapter · `app.define()` · config validation (fail at startup on unsafe config)

</details>

<details>
<summary><b>v0.2 — Realtime and agents</b></summary>

<br>

WebSocket adapter (reconnect, resume, dedupe, backpressure) · MCP adapter · shared policy proven across HTTP, WS and MCP · typed client generation

</details>

<details>
<summary><b>v0.3 — Policy layer</b></summary>

<br>

IP guard with `trustProxy` · strict CORS · private-by-default shield · rule engine with report mode · `rules test --replay` · audit log

</details>

<details>
<summary><b>v0.4 — Services</b></summary>

<br>

Service-to-service client (timeouts, retries, idempotency) · trace propagation across protocols · Docker support

</details>

<details>
<summary><b>Later</b></summary>

<br>

gRPC and GraphQL adapters · adaptive execution scheduler · plugin system · Fastify/Hono integration

</details>

</details>

<a id="contributing"></a>
<details>
<summary><b>Contributing</b></summary>

<br>

Contributions of every size are welcome: bug fixes, docs, new protocol adapters, bypass tests, or feature proposals.

1. Fork the repo and branch from `main`.
2. Make focused commits and add tests where relevant.
3. Open a PR describing the change and the motivation.

Security reports: please open a private advisory instead of a public issue.

</details>

<details>
<summary><b>License</b></summary>

<br>

Released under the [MIT License](LICENSE).

</details>

---

<div align="center">

**Define once. Connect everywhere.**

</div>
