---
name: plan-review
description: Critique a plan, spec, or architecture decision before committing to implementation. Spawns an adversarial reviewer agent to surface invalid assumptions, missing edge cases, underspecified integrations, and over-engineering. Trigger phrases - "review this plan", "critique this spec", "challenge this design before I build it", "adversarial plan review", "stress-test this plan".
license: MIT
compatibility: Executed as a custom agent with sequential-thinking MCP for phase enforcement.
    Degrades to Fallback tier if sequential-thinking unavailable.
metadata:
    author: zaynram
    version: "1.1.0"
---

# Plan Review

Adversarial critique of a plan, spec, or architecture decision before
implementation begins. Four phases: assumption inventory, adversarial review
across six categories, severity assignment, and integration notes. Produces
findings grouped by severity with concrete suggested resolutions.

## When to Use

- A plan, spec, or architecture document exists and implementation is imminent
- You want an external perspective before committing resources to a direction
- A design decision has been made and you want to stress-test it
- After running writing-plans or feature-dev architecture phases
- As a pre-flight check before opening a Full-weight work package

## When NOT to Use

- No concrete plan exists yet - use adversarial-cycle on specific claims instead
- You need to compare alternatives - use tradeoff-analysis
- You need to trace downstream impact of a decided plan - use downstream-forecasting
- The plan is a one-liner or trivially obvious - overhead exceeds value

## Execution

This protocol runs as the `plan-review` agent. The agent follows the full
phase procedure autonomously, using the `sequentialthinking` MCP tool for
phases marked _(Sequential-thinking required)_.

- Protocol selection and composition: use `/reason`
- Phase validation: enforced by SubagentStop hook
- Without sequential-thinking: agent tags output `validation_tier: Fallback`

## Input

- `artifact` (required): Path to or content of the plan, spec, or architecture document
- `focus_areas` (optional): Specific areas or concerns to prioritize during review

## Output

- `findings`: Array of {finding, category, severity, resolution}
- `assumption_inventory`: Assumptions with confidence ratings (High / Medium / Low)
- `summary`: critical_count, important_count, minor_count, validation_tier

## Composition

- **Post-planning review:** Natural follow-up to writing-plans or feature-dev architecture output. Review the plan before opening execution brackets.
- **Finding escalation:** Critical findings can be escalated to adversarial-cycle for deeper stress-testing of specific claims.
- **A-PR pipeline:** Assumption-surfacing → plan-review. Surface assumptions first, then review the plan with explicit assumption awareness.
- Use `/reason` for full composition guidance, pattern selection, and pipeline specifications.
