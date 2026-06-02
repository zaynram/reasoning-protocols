---
name: tradeoff-analysis
description: Score alternatives against weighted criteria with sensitivity testing. Use when comparing options, evaluating tradeoffs, or making a structured decision between alternatives. Trigger phrases - "compare alternatives", "which option is better", "tradeoff analysis", "evaluate options", "decision matrix".
license: MIT
compatibility: Executed as a custom agent with sequential-thinking MCP for phase enforcement.
    Degrades to Fallback tier if sequential-thinking unavailable.
metadata:
    author: zaynram
    version: "1.0.0"
---

# Tradeoff Analysis

Pre-decision structured comparison. Scores alternatives against weighted criteria with sensitivity analysis and convergence checking. Four phases: criteria enumeration, weighting justification, per-alternative scoring (dense ranks with mechanistic arguments), sensitivity analysis with flip condition detection. The caller decides - this is not a recommendation engine.

## When to Use

- Choosing between two or more concrete alternatives
- Need auditable, reproducible comparison rather than gut-feel
- Want to understand which criteria or weights could flip the result
- Pre-flight before stress-testing the selected option (T-V pipeline)

## When NOT to Use

- Only one alternative exists (nothing to compare)
- The decision is about validating a specific claim (use adversarial-cycle)
- You need to trace downstream effects (use downstream-forecasting)
- You need to surface assumptions (use assumption-surfacing)

## Execution

This protocol runs as the `tradeoff-analysis` agent. The agent follows
the full phase procedure autonomously, using the `sequentialthinking` MCP
tool for phases marked _(Sequential-thinking required)_.

- Protocol selection and composition: use `/reason`
- Domain research delegation: agent spawns `domain-researcher` as needed
- Phase validation: enforced by SubagentStop hook
- Without sequential-thinking: agent tags output `validation_tier: Fallback`

## Input

- `decision` (required): The question being decided
- `alternatives` (required): Options under consideration (≥2)
- `seed_criteria` (required, ≥1): Initial evaluation dimensions, each with `label` and `operational_definition`
- `scope_boundary` (optional): Constraints on consideration
- `weighting_context` (optional): Caller context for relative importance; on re-invocation, may carry prior remediation hints

## Output

- `criteria_assessment`: Confirmed criteria with completeness notes and correlation pairs
- `weighted_criteria`: Criteria with percentage weights, justifications, cross-references
- `score_matrix`: Alternative × criterion matrix with dense ranks and per-cell mechanistic arguments
- `sensitivity_report`: Flip conditions (independent + clustered), remediation hints, convergence_status
- `validation_tier`: Full | Fallback

## Composition

- **T-V pipeline:** TA output → synthesizing adapter → Adversarial Cycle. The winning alternative becomes a claim for stress-testing.
- **T-V-F pipeline:** TA → AC → DF. Full decision pipeline: compare, stress-test, forecast.
- Use `/reason` for full composition guidance, pattern selection, and pipeline specifications.
