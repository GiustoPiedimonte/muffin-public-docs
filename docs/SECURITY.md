# Security and privacy boundaries

> **Status: CURRENT BOUNDARIES · ACTIVE HARDENING**

This page describes public security semantics, not private configuration,
internal paths or exploit detail.

## Owner control

Muffin is designed as an owner-run agent. Runtime data is kept in the owner's
Muffin home rather than a central Muffin service. Source updates must not own or
rewrite identity and memory.

Owner-run does not mean risk-free. The current runtime can call cloud models,
execute local capabilities and connect to external services when configured.

## Provider egress

The configured model provider receives the context required for a model call.
Muffin does not currently claim that every item has an automatic local-only or
per-provider privacy policy.

Local-capable is a direction; fully local inference is not the only current mode.

## Meaning is not authority

The model may interpret context and propose a tool call. A separate policy
boundary decides whether the declared capability is allowed, requires owner
confirmation or is denied.

Understanding the owner's likely intent never grants additional authority by
itself.

## Provenance and taint

Muffin keeps source and trust information with remembered material and propagates
untrusted-context constraints toward tool and egress decisions. Retrieval or
summarisation must not silently upgrade trust.

## Secrets

Known stored secrets are designed to stay outside normal model prompts,
transcripts and tool output. Secret values should enter through hidden input or
stdin, not command-line arguments.

Redaction is defence in depth. Arbitrary text pasted by an owner may contain a
credential in a form the system cannot recognise structurally.

## Local tools and third-party code

Filesystem, process and network capability is scoped and policy-gated, but local
execution remains security-sensitive.

An MCP tool schema or pinned configuration does not prove that a third-party MCP
server process is contained. Until stronger process isolation is publicly
proven, explicitly configured third-party server code should be treated as part
of the trusted computing base.

## Effects and crashes

Muffin distinguishes intended work, attempted work and observed outcome. If a
process dies after an external action starts, honest uncertainty is safer than
assuming the action never happened and retrying blindly.

Universal rollback or undo is not a current public guarantee.

## Current non-claims

- No hosted multi-tenant security model.
- No remote Node protocol in the current runtime.
- No guarantee that arbitrary third-party extension code is safely contained.
- No claim that all DAY-1 security and recovery evidence is complete.
- No claim of an independent public security audit.

Security mechanisms are considered real only when the production path reaches
them and failure-path evidence supports the claim.
