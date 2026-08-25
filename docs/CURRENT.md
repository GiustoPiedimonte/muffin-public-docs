# Current product

> **Status snapshot: 2026-08-25 · DEVELOPER PREVIEW · PRE-DAY-1**

This page describes the active `muffin-agent` runtime at a public, deliberately
coarse level. It does not mirror volatile private Gate counts or operational
details.

## Working today

### Runtime and interaction — CURRENT

- Owner-run TypeScript runtime on Node.js 22 or newer.
- Interactive CLI and one-shot headless execution.
- Scriptable exit codes and operator diagnostics.
- Anthropic and OpenAI-compatible provider adapters.
- Configurable model profiles and streaming where the surface supports it.

### Continuity and memory — CURRENT · DOGFOOD

- Durable episodes and beliefs with provenance.
- Correction and supersession without silently rewriting history.
- Time-aware memory and hybrid recall with reranking.
- A local document vault with indexing and consistency checks.
- Owner inspection commands for search, provenance, review and health.

### Work and time — CURRENT · DOGFOOD

- Persistent turns and work that can wait or resume across process boundaries.
- Scheduled jobs with timezone-aware cron handling.
- A resident gateway for work that must outlive an open terminal.
- A conservative proactivity gate: the scheduler may produce a signal without
  earning the right to send a message.

### Authority and tools — CURRENT · DOGFOOD

- A policy decision boundary between model intent and tool execution.
- Filesystem, process, HTTP/search and local execution capabilities with scoped
  controls.
- Root-of-trust checks for constitutional configuration.
- Secret input and storage paths designed to keep known credentials out of
  normal model and transcript traffic.
- MCP client support for explicitly configured servers.

### Surfaces — CURRENT · DOGFOOD

- CLI and headless operation.
- Telegram connector with a durable inbox, turn handling and media ingestion;
  multi-event intent composition remains an active hardening area.
- Shared continuity across surfaces is a product goal; cross-surface behaviour
  is still being hardened and should not be treated as general-release quality.

### Inspection — CURRENT

- `doctor` checks configuration and runtime dependencies.
- `trace` exposes redacted operational traces.
- `prompt show` lets the owner inspect the effective prompt without calling a
  model.
- Memory, vault, surface, job, gateway and MCP operator commands.

## Not a current claim

- General public availability or consumer-grade onboarding.
- A public package release or stable compatibility promise.
- Fully local inference as the only supported mode.
- Remote Nodes or a multi-device execution protocol.
- Phone, desktop, voice or wearable clients.
- A public extension marketplace or safe arbitrary third-party code sandbox.
- Automatic intentional belief-writing by the agent as a finished capability.
- Universal undo for real-world effects.
- A hosted multi-tenant Muffin service.

## Maturity boundary

The runtime is real; the product is still being hardened for safe daily owner
use. Active development is closing recovery, migration, inbound composition and
real-process acceptance gaps before the 14-day dogfood Gate can begin.

Public documentation therefore uses **CURRENT · DOGFOOD** instead of implying
production readiness.
