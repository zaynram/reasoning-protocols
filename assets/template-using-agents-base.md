---
name: using-agents
description: Guide for running reasoning protocol agents in environments without native agent support. Provides session isolation, tiered handoff (memory-keeper or paste-based), and composition chaining for cross-session protocol execution.
---

# Using Agents - Reasoning Protocols

This skill provides guidance for executing reasoning protocol agents in
environments that do not support Claude Code's native agent auto-discovery
(e.g., claude.ai, API-based integrations).

## Available Agents

{AGENTS_LIST}

## Session Isolation

Each protocol execution MUST run in a separate conversation session.
Do not run multiple protocols in the same conversation - output contract
compliance depends on session isolation.

## State Management

### Tier 1: memory-keeper available

If the `memory-keeper` MCP server is available, use context keys for handoffs:

- **Handoff in:** `context_save(channel="protocols", key="protocol:{name}:handoff", value=<inputs>)`
- **Output:** `context_save(channel="protocols", key="protocol:{name}:output", value=<structured_output>)`
- **Composition chains:** AC output key → DF input reads from AC output key

### Tier 2: paste-based fallback

If memory-keeper is unavailable, use structured paste handoffs:

1. Prepare the handoff using the handoff-in template
2. Start a new conversation and paste the handoff
3. When the protocol completes, extract output using the handoff-out template
4. Validate using the validate-refine template

## Self-Containment Test

Before delegating to a protocol session, verify:

- [ ] All required input contract fields are populated
- [ ] Scope boundary is explicitly defined
- [ ] The session can execute without needing to ask the caller questions

If any check fails, resolve it in the current conversation before delegating.

{EXTENSION}

## Handoff Templates

For paste-based handoffs, read these templates on demand:

- `assets/template-handoff-in.md` - structured setup for the protocol session
- `assets/template-handoff-out.md` - what to extract from the completed session
- `assets/template-validate-refine.md` - per-criterion validation loop
