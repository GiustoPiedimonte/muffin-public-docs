# Documentation authority and status

This directory describes the current public shape of Muffin without duplicating
private implementation detail.

## Which document answers which question?

| Question | Public authority |
|---|---|
| Why does Muffin exist and where is it going? | [Vision](VISION.md) |
| What can the runtime do today? | [Current product](CURRENT.md) |
| How is the current system shaped? | [Architecture](ARCHITECTURE.md) |
| How can an authorised evaluator install and run it? | [Install and use](INSTALL.md) |
| What comes next? | [Roadmap](ROADMAP.md) |
| What privacy and security boundaries are claimed? | [Security](SECURITY.md) |
| What do common terms and status labels mean? | [FAQ](FAQ.md) |

## Authority rule

Executable code and real integration evidence own mechanical facts. These public
documents own stable explanations, product boundaries and maturity labels.

The following do **not** prove a current feature exists:

- a vision statement;
- an architecture proposal;
- an experiment from an earlier generation;
- a passing isolated unit test;
- an unmerged development branch;
- a roadmap item.

When a public claim cannot be grounded without exposing private operational or
security-sensitive detail, this repository states the boundary at a higher
level or omits the claim.

## Status vocabulary

- **CURRENT** — present in the active runtime.
- **DOGFOOD** — working but still being hardened in real owner use.
- **EXPERIMENTAL** — implemented or explored without a durable guarantee.
- **PLANNED** — intended direction, not current runtime behaviour.
- **HISTORICAL** — preserved earlier architecture or product thinking.

The exact engineering Gate changes too quickly to mirror here safely. Public
status stays deliberately coarse and is revised when the product claim changes.

## Historical material

The numbered Italian chapters at the repository root are the H1 2026 public
corpus. They remain available for research and citation, but their architecture
and status claims are historical. See [the history index](../HISTORY.md).
