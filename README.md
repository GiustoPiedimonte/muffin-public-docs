# Muffin

### A human-first personal agent built for continuity, useful work and owner control.

> **DEVELOPER PREVIEW · PRE-DAY-1**
>
> Muffin is a working owner-run runtime under active hardening. It is not a
> general-release product, a public package or a promise that every capability
> described in the vision exists today.

Models change. Devices change. Apps change. Muffin should not have to become a
different agent every time they do.

Muffin is built around **quality, ownership and portability of personal
continuity**: preserving not only facts, but their sources, corrections,
unfinished work, possible real-world effects and the authority the agent has.

## Read this repository by status

| Label | Meaning |
|---|---|
| **CURRENT** | Present in the active `muffin-agent` runtime. |
| **DOGFOOD** | Working, but still being hardened through real owner use. |
| **EXPERIMENTAL** | Implemented or explored without a stable product guarantee. |
| **PLANNED** | Product or architecture direction, not current functionality. |
| **HISTORICAL** | A preserved description of an earlier Muffin generation. |

A design document is not proof that the runtime implements a feature. Current
product claims are intentionally narrower than the vision.

## What exists today

| Area | Public status |
|---|---|
| Owner-run TypeScript runtime and CLI | **CURRENT · DOGFOOD** |
| Interactive and one-shot operation | **CURRENT** |
| Anthropic and OpenAI-compatible model providers | **CURRENT** |
| Durable memory with provenance and correction history | **CURRENT · DOGFOOD** |
| Persistent work, waiting, recovery and scheduling | **CURRENT · DOGFOOD** |
| Policy-gated tools, local execution and MCP connections | **CURRENT · DOGFOOD** |
| Telegram as an additional surface | **CURRENT · DOGFOOD** |
| Diagnostics, tracing and owner inspection commands | **CURRENT** |
| Consumer-grade installer and onboarding | **PLANNED** |
| Remote Nodes, phone/wearable bodies and portable capsule format | **PLANNED** |
| Public extension marketplace | **PLANNED** |

The active hardening goal is to return Muffin to daily owner use safely, then
earn a continuous 14-day dogfood run. This is a safety and reliability gate, not
a launch date.

## What Muffin is trying to become

Muffin has three equal jobs:

- **Do** — carry real work through, including work that waits or spans sessions.
- **Understand** — keep evidence, beliefs, uncertainty and provenance distinct.
- **Be present** — preserve obligations over time and know when silence is better.

The target is more capability **without making the owner less agentic**. Better
understanding never grants more authority by itself.

Read the [vision](docs/VISION.md), then the [current product](docs/CURRENT.md) and
[roadmap](docs/ROADMAP.md) to keep destination and implementation separate.

## Developer preview

There is no public package release yet. If you have authorised access to the
`muffin-agent` source checkout, the current path is:

```sh
./install.sh
muffin init
muffin
```

Node.js 22 or newer and a supported model provider are required. The setup flow
collects the provider secret through hidden interactive input; secrets should
not be passed as command-line arguments.

See [install and use](docs/INSTALL.md) for the current operator workflow and its
limits.

## Documentation map

- [Documentation authority and status](docs/README.md)
- [Vision](docs/VISION.md)
- [Current product](docs/CURRENT.md)
- [Public architecture](docs/ARCHITECTURE.md)
- [Install and use](docs/INSTALL.md)
- [Roadmap](docs/ROADMAP.md)
- [Security and privacy boundaries](docs/SECURITY.md)
- [FAQ](docs/FAQ.md)
- [Historical H1 2026 corpus](HISTORY.md)

## Why the older documents remain

The original public corpus records an earlier Muffin generation built around an
awareness loop, Predictor/Scheduler/Decider, a dream cycle, living profiles and a
different product thesis. Those documents are useful research history, but they
do not describe the current runtime.

They remain available with an explicit **HISTORICAL** banner so old links keep
working and changes in the project's thinking stay visible.
