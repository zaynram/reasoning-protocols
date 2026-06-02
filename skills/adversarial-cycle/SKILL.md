---
name: adversarial-cycle
description: Stress-test claims via structured conclusion, counter-claim, response cycles. Use when you need to validate a decision, challenge an argument, or verify that a conclusion survives scrutiny. Trigger phrases - "stress-test this", "challenge this claim", "validate this decision", "adversarial review".
license: MIT
compatibility: Executed as a custom agent with sequential-thinking MCP for phase enforcement.
    Degrades to Fallback tier if sequential-thinking unavailable.
metadata:
    author: zaynram
    version: "1.0.0"
---

# Adversarial Cycle

Validate a claim through structured conclusion, counter-claim, response cycles with defined sufficiency requirements. Each claim passes through a three-phase cycle that must meet explicit sufficiency criteria before the claim is considered validated. Produces a determination of whether the claim survives scrutiny and in what form -- validated, modified, refuted, or early-exit (unfalsifiable within scope).

## When to Use

- A specific claim or decision needs stress-testing before commitment
- You have a falsifiable claim with a supporting argument to challenge
- Evaluating a design choice, architecture decision, or protocol rule
- Testing a set of resolved decisions as a composed system (aggregated mode)
- Pre-flight check before investing in downstream impact analysis

## When NOT to Use

- The question is about assumptions underlying a decision (use assumption-surfacing)
- You need to compare alternatives against criteria (use tradeoff-analysis)
- You need to trace downstream impact without first validating a claim (use downstream-forecasting directly)
- The claim is too vague to be falsifiable -- restate with commitment first

## Execution

This protocol runs as the `adversarial-cycle` agent. The agent follows
the full phase procedure autonomously, using the `sequentialthinking` MCP
tool for phases marked _(Sequential-thinking required)_.

- Protocol selection and composition: use `/reason`
- Domain research delegation: agent spawns `domain-researcher` as needed
- Phase validation: enforced by SubagentStop hook
- Without sequential-thinking: agent tags output `validation_tier: Fallback`

## Input

- `claim` (required): The decision or position being stress-tested, stated as a falsifiable claim
- `supporting_argument` (required): The reasoning supporting the claim; provides the mechanistic base the counter-claim must attack
- `decision_context` (optional): Broader context framing why this claim matters; scopes counter-claims
- `mode` (optional): `individual` (default) or `aggregated` for testing composed decision sets

## Output

- `validation_status`: validated | modified | refuted | early-exit
- `final_claim`: The surviving claim (present when validated or modified; absent when refuted or early-exit)
- `cycle_transcript`: Formatted summary block per cycle
- `validation_tier`: Full | Fallback

## Composition

- **Validate-and-Forecast (V-F):** AC output feeds Downstream Forecasting via a validation gate. Refuted claims terminate the pattern; validated/modified claims proceed to impact forecasting.
- **T-V pipeline:** Receives the winning alternative from Tradeoff Analysis as a synthesized claim.
- **T-V-F pipeline:** Same as T-V, then forwards the validated claim to Downstream Forecasting.
- **A-V-F pipeline:** Receives bifurcated input from Assumption Surfacing adapter.
- Use `/reason` for full composition guidance, pattern selection, and pipeline specifications.
