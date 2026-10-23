<div align="center">

<img src="https://raw.githubusercontent.com/NEURAL-Y/Byte-Plus/main/public/logo.png" alt="DualBridge" width="220"/>

# DualBridge

**Connect once. Build everywhere.**

![Status](https://img.shields.io/badge/status-under%20development-yellow)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

[Why](#why) • [Features](#features) • [Quick Start](#quick-start) • [Architecture](#architecture) • [Roadmap](#roadmap) • [Contributing](#contributing)

</div>

---

DualBridge is an open-source communication framework that automatically bridges your backend with applications, AI agents, frontend frameworks, and services — without writing repetitive integration code. Instead of hand-rolling API wrappers, API key management, WebSocket setup, and auth middleware, DualBridge handles the communication layer for you.

Whether you're on **FastAPI**, **Express.js**, an **MCP Server**, or anything speaking HTTP/WebSockets, DualBridge gives you one secure, unified way to connect it all.

<a id="why"></a>
<details open>
<summary><b>Why?</b></summary>

<br>

Every backend project eventually needs the same infrastructure: REST clients, WebSocket connections, auth, API key management, secure communication, rate limiting, multi-language SDKs, config management — hours of plumbing before you can build the actual feature. DualBridge eliminates that repetitive work.

</details>

<a id="features"></a>
<details>
<summary><b>Features</b></summary>

<br>

| | |
|---|---|
| 🌐 **Multi-Framework** | FastAPI and Express.js today, more via plugins |
| 🔄 **Built-in WebSockets** | Connection lifecycle, reconnection, event handling, message routing — handled automatically |
| 🔑 **API Key Management** | Multiple keys, rotation, expiration, permission levels |
| 🔒 **Optional Encryption** | Encrypt API keys, credentials, and config — or disable for local dev |
| 🛡 **Password Protection** | An extra password layer before any client can connect |
| 🚦 **Rate Limiting** | Requests/min, concurrent connections, WebSocket limits, custom policies |
| 🧩 **Automatic SDK Generation** | Python and JavaScript client libraries generated for you, more languages planned |

</details>

<a id="quick-start"></a>
<details>
<summary><b>Quick Start</b></summary>

<br>

Define your project once:

```yaml
backend: fastapi
endpoint: http://localhost:8000

sdk:
  - python
  - javascript

websocket: true

security:
  encryption: true
  password: my-password

rate_limit:
  requests_per_minute: 500
```

Then run:

```bash
dualbridge connect
```

DualBridge automatically connects to your backend, generates SDKs, configures WebSockets, sets up authentication, manages API keys, and applies your security settings.

</details>

<a id="architecture"></a>
<details>
<summary><b>Architecture</b></summary>

<br>

```text
                    Backend
          (FastAPI / Express.js)
                     │
                     ▼
              ┌───────────────┐
              │  DualBridge   │
              └───────────────┘
          ┌────────┼─────────┐
          ▼        ▼         ▼
     Python SDK  JS SDK   WebSocket
                     │
                     ▼
 AI Agents • MCP Servers • Web Apps
 Desktop Apps • Mobile Apps • CLI
```

</details>

<details>
<summary><b>Example Use Cases</b></summary>

<br>

AI applications • MCP servers • internal APIs • SaaS platforms • React / Next.js apps • desktop & mobile apps • Python automation • CLI tools

</details>

<details>
<summary><b>Philosophy</b></summary>

<br>

Infrastructure shouldn't be rewritten for every project. DualBridge lets developers focus on business logic instead of repeatedly implementing communication layers, auth systems, SDKs, and networking code.

**Build your backend. Configure DualBridge. Connect everything.**

</details>

<a id="roadmap"></a>
<details>
<summary><b>Roadmap</b></summary>

<br>

<details>
<summary><b>v0.1</b></summary>

<br>

FastAPI support · Express.js support · configuration parser · CLI · Python SDK generator · JavaScript SDK generator

</details>

<details>
<summary><b>v0.2</b></summary>

<br>

WebSocket manager · API key manager · password authentication · encryption

</details>

<details>
<summary><b>v0.3</b></summary>

<br>

Rate limiting · plugin system · Docker support · MCP integration · additional framework support

</details>

</details>

<a id="contributing"></a>
<details>
<summary><b>Contributing</b></summary>

<br>

Contributions of every size are welcome — bug fixes, docs, new framework support, or feature proposals.

1. Fork the repo and branch from `main`.
2. Make focused commits; add tests where relevant.
3. Open a PR describing the change and motivation.

</details>

<details>
<summary><b>License</b></summary>

<br>

Released under the [MIT License](LICENSE).

</details>

---

<div align="center">

**Build your backend once. Connect it everywhere.**

</div>
