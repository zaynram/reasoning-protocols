---
name: downstream-forecasting
description: Trace cascade effects of a decision through downstream dependents with severity-gated termination and amplification detection. Use when you need to forecast impact, trace consequences, or assess what breaks downstream. Trigger phrases - "what are the downstream effects", "impact analysis", "what breaks if we do this", "cascade effects".
license: MIT
compatibility: Executed as a custom agent with sequential-thinking MCP for phase enforcement.
    Degrades to Fallback tier if sequential-thinking unavailable.
metadata:
    author: zaynram
    version: "1.0.0"
---

# Downstream Forecasting

Trace the cascade effects of a decision through its downstream dependents, with severity-gated termination and amplification detection. Produces a structured forecast showing where the decision propagates, at what severity, and where cascades terminate. Five phases: impact surface enumeration, cascade tracing, constraint check, interaction detection, forecast summary.

## When to Use

- A decision has been made and you need to understand its downstream impact
- Tracing second-order effects that first-order thinking might miss
- Assessing severity of change propagation across systems or components
- After validating a claim via adversarial-cycle (Validate-and-Forecast pattern)
- After surfacing fragile assumptions (Assumption-and-Forecast pattern)

## When NOT to Use

- The decision hasn't been validated yet (use adversarial-cycle first)
- You need to compare alternatives (use tradeoff-analysis)
- You need to surface hidden assumptions (use assumption-surfacing)
- The scope boundary is undefined - DF requires explicit scope to terminate

## Execution

This protocol runs as the `downstream-forecasting` agent. The agent follows
the full phase procedure autonomously, using the `sequentialthinking` MCP
tool for phases marked _(Sequential-thinking required)_.

- Protocol selection and composition: use `/reason`
- Domain research delegation: agent spawns `domain-researcher` as needed
- Phase validation: enforced by SubagentStop hook
- Without sequential-thinking: agent tags output `validation_tier: Fallback`

## Input

- `decision` (required): The decision being forecasted, stated specifically and falsifiably
- `scope_boundary` (required): Systems, components, or domains in play; dependents outside are not traced
- `decision_context` (optional): Broader context from which the decision emerged
- `severity_threshold` (optional): Minimum severity to include in cascade tracing

## Output

- `impact_surface`: Direct dependents with dependency mechanisms and amplification flags
- `cascade_map`: Per-dependent cascade paths with severity at each hop
- `constraint_violations`: Per-dependent constraint violations
- `convergence_points`: Components with converged impacts and re-evaluated severities
- `boundary_exceeded`: Cascade paths that reached the scope boundary
- `forecast_summary`: Human-readable per-dependent narrative
- `validation_tier`: Full | Fallback

## Composition

- **Validate-and-Forecast (V-F):** Receives `final_claim` from Adversarial Cycle as `decision` input. Modified claims use the revised form.
- **Assumption-and-Forecast (A-F):** Receives `with-basis` assumptions from Assumption Surfacing via routing gate. `without-basis` assumptions route to coverage gap passthrough.
- **T-V-F pipeline:** Receives validated claim from the T-V stage.
- Use `/reason` for full composition guidance, pattern selection, and pipeline specifications.
