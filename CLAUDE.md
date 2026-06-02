# reasoning-protocols

Five structured reasoning protocols: adversarial stress-testing, impact forecasting, tradeoff analysis, assumption surfacing, and plan review.

## Features

| Component                      | Description                                                                             |
| ------------------------------ | --------------------------------------------------------------------------------------- |
| `/reason` command              | Protocol selection and composition - invokes one or more protocols for a given decision |
| `adversarial-cycle` skill      | Claim stress-testing via conclusion → counter-claim → response cycles                   |
| `downstream-forecasting` skill | Cascade impact tracing with severity-gated termination                                  |
| `tradeoff-analysis` skill      | Scored alternative comparison with sensitivity analysis                                 |
| `assumption-surfacing` skill   | Foundation audit with fragility rating and epistemic status tagging                     |
| `plan-review` skill            | Adversarial plan critique with assumption surfacing and risk identification             |
| `sequential-thinking` MCP      | Required for Full-tier protocol execution; enforces phase discipline                    |

## Composition

Protocols compose at two levels above the primitives:

- **L1 Patterns** - Validate-and-Forecast (AC → DF), Assumption-and-Forecast (AS → DF)
- **L2 Pipelines** - A-V-F, T-V, T-V-F; use adapters for inter-stage orchestration

Without the sequential-thinking MCP server, protocols degrade to Fallback tier: same output format, phase discipline not enforced.

## Operator Runbook

### SubagentStop hook cache-lag after a hook-matcher change

**Context:** Chore 43 (commit `d1c4771`) tightened the `SubagentStop` hook matcher from a wildcard (`*`) to the explicit protocol-agent name list now in `hooks/hooks.json`. The source fix is correct and shipped. However, Claude Code caches plugin hooks for the lifetime of the in-process session. Any session that was already running when the fix was pulled will continue to use the old matcher until the session is restarted.

**Symptom:** In-session non-protocol subagents (e.g. `chore-executor`, `plan-executor`) still route through the hook validation prompt and may lose output to meta-commentary, even after `d1c4771` is in the working tree.

**Runbook step:** After pulling any change to `hooks/hooks.json` in this plugin (including the chore-43 matcher fix), restart the Claude Code session before spawning subagents. A session reload forces hook re-registration from the updated source.

**Scope note:** Plugin-cache invalidation semantics are controlled by the Claude Code substrate and are not configurable from within the plugin. The restart requirement cannot be automated away without substrate-level support.
