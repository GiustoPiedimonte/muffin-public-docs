# Public architecture

> **Status: CURRENT SHAPE** — stable boundaries, not private implementation detail.

Muffin is one owner-run agent with replaceable model providers, surfaces and
capabilities around a durable continuity substrate.

## Five semantic planes

```text
Evidence · Beliefs · Work · Effects · Authority
```

- **Evidence** records what entered the system or was observed, with source and
  time when known.
- **Beliefs** represent what Muffin currently thinks, including uncertainty,
  contradiction and correction history.
- **Work** represents what remains owed, waiting, resumable or due.
- **Effects** distinguish intended action, attempted action, known outcome and
  honest uncertainty after a failure boundary.
- **Authority** defines which transitions Muffin may perform for which owner,
  surface and capability.

These planes should not collapse into one conversation transcript or one opaque
profile. Meaning does not imply permission, and a requested effect is not proof
that it happened.

## Runtime flow

```text
CLI / Telegram / scheduled occurrence
                  │
                  ▼
        durable ingress and work
                  │
                  ▼
     context + memory + agent loop
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
 model provider       policy-gated tools
        │                   │
        └─────────┬─────────┘
                  ▼
        durable outcome / delivery
```

Current durable paths establish identity before work crosses a crash boundary.
Inbound event composition is still being hardened. A model response, a tool
effect and delivery to a surface are distinct outcomes.

## Main boundaries

### Core

Owns configuration, policy, root of trust, tracing, memory, work, scheduling and
the stable contracts used by the rest of the runtime.

### Agent

Assembles context, calls the configured provider, interprets tool requests and
coordinates a turn. The model proposes; policy and runtime wiring decide what is
allowed to execute.

### Surfaces

CLI and Telegram connect people and events to the same logical agent. A surface
is not a separate Muffin and should not own a separate identity or policy.

### Providers

Model providers are replaceable compute boundaries. Current adapters cover
Anthropic and OpenAI-compatible APIs. Provider choice does not own memory or
identity.

### Capabilities and MCP

Built-in tools and configured MCP servers extend what Muffin can read or do.
Capability declarations and owner policy remain distinct from installation or
discovery.

## Data location

Runtime data lives under the owner's Muffin home rather than inside the source
checkout. Updating code must not overwrite identity, memory or owner data.

The current physical representation is an implementation detail. Long-term
portability requires preserving semantic continuity, not treating today's
database or embedding index as the identity of Muffin.

## Planned topology

A future Home/Node model may let one canonical Muffin use capabilities on
multiple owner-controlled devices. That protocol and those clients are
**PLANNED**, not present in the current runtime.
