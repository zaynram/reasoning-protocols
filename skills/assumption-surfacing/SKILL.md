---
name: assumption-surfacing
description: Audit the assumption foundation underlying a decision. Surfaces hidden assumptions, assesses fragility with quality gating, builds an epistemic dependency graph, and produces a unified inventory tagged by epistemic status. Trigger phrases - "what assumptions are we making", "assumption audit", "what could break", "hidden dependencies", "fragility assessment".
license: MIT
compatibility: Executed as a custom agent with sequential-thinking MCP for phase enforcement.
    Phase 5 refinement loop delegates to domain-researcher agent.
    Degrades to Fallback tier if sequential-thinking unavailable.
metadata:
    author: zaynram
    version: "1.0.0"
---

# Assumption Surfacing

Audit the assumption foundation underlying a decision. Six phases: domain topology enumeration, assumption enumeration, fragility rating with quality floor, dependency graph tracing, refinement loop (one retry via domain-researcher agent), output synthesis. Produces a unified inventory with epistemic status tags (`with-basis`, `without-basis`, `ungrounded`) and structural importance scores.

## When to Use

- A decision depends on unstated beliefs that need exposure
- You need to know which assumptions are fragile before committing
- Pre-flight for impact analysis on fragile assumptions (A-F pattern)
- Auditing the foundation of a system design or strategy
- When "what could go wrong?" needs a structured answer beyond gut-feel

## When NOT to Use

- You have a specific claim to stress-test (use adversarial-cycle)
- You need to compare alternatives (use tradeoff-analysis)
- You already know the assumptions and need downstream impact (use downstream-forecasting directly)

## Execution

This protocol runs as the `assumption-surfacing` agent. The agent follows
the full phase procedure autonomously, using the `sequentialthinking` MCP
tool for phases marked _(Sequential-thinking required)_.

- Protocol selection and composition: use `/reason`
- Domain research delegation: agent spawns `domain-researcher` as needed
- Phase validation: enforced by SubagentStop hook
- Without sequential-thinking: agent tags output `validation_tier: Fallback`

## Input

- `decision_context` (required): The decision whose assumption foundation is being audited
- `domain_grounding_source` (required): Source for domain knowledge - document, specification, or domain description
- `scope_boundary` (optional): Constraints on which assumptions to surface
- `acceptance_threshold` (optional): Caller-supplied confidence threshold for fragility ratings

## Output

- `unified_inventory`: All assumptions with epistemic status, fragility ratings (where applicable), structural importance scores (where applicable), ungrounded-basis annotations, and dependency positions
- `dependency_graph`: Directed epistemic dependency graph
- `unanalyzed_risks`: Filtered view of `without-basis` and `ungrounded` assumptions ordered by structural importance
- `summary`: grounded_count, without_basis_count, ungrounded_count, validation_tier

## Composition

- **Assumption-and-Forecast (A-F):** AS output → routing gate → `with-basis` to DF severity propagation, `without-basis`/`ungrounded` to coverage gap passthrough. Structural importance is prioritization metadata, NOT fragility input.
- **A-V-F pipeline:** AS → bifurcating adapter → AC + DF. Surface assumptions, stress-test decision, forecast fragile assumption impacts.
- Use `/reason` for full composition guidance, pattern selection, and pipeline specifications.
