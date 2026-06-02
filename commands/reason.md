---
description: Select and compose structured reasoning protocols (adversarial cycle, downstream forecasting, tradeoff analysis, assumption surfacing, plan review)
---

# Reasoning Protocols - Protocol Selection & Composition

A stateless protocol library for structured analytical reasoning. Five protocols at L0 (primitive), two composition patterns at L1, and three validated pipelines at L2.

## Scope

**Provides:** Five analytical protocols (Adversarial Cycle, Downstream Forecasting, Tradeoff Analysis, Assumption Surfacing, Plan Review), two composition patterns (Validate-and-Forecast, Assumption-and-Forecast), three multi-protocol pipelines (A-V-F, T-V, T-V-F).

**Does not provide:** Persistent state, lifecycle machinery, or enforcement logic. Callers own all of these. Protocols are side-effect-free - callers spawn agents and receive structured output.

## Core Principles

**Stateless protocol library.** No persistent state ownership. Callers supply inputs, receive structured outputs, and own all enforcement and state. Mid-pipeline failure has no checkpoint/resume - protocols are side-effect-free, so re-invocation from scratch is always safe.

**Convergence-seeking (+1).** A protocol terminates when an additional pass cannot improve the output - not when a predetermined number of passes is reached. Each protocol instantiates convergence differently:

| Protocol               | Convergence Instantiation        | Convergence Signal                                                                                                       |
| ---------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Adversarial Cycle      | Native termination semantics     | No surviving counter-claim after a full cycle                                                                            |
| Downstream Forecasting | Amplification override (Phase 2) | Severity propagation stabilizes                                                                                          |
| Assumption Surfacing   | Refinement loop (Phase 5)        | All assumptions grounded or explicitly ungrounded after retry                                                            |
| Tradeoff Analysis      | Phase 4 as convergence check     | No flip condition targets a weakly-argued cell AND no flip condition is precision-dependent on a loosely-anchored weight |

**Full and Fallback tiers.** Sequential-thinking is required for Full-tier execution and enforces phase discipline. Without sequential-thinking, protocols degrade to Fallback tier: same output format contract, same sufficiency criteria, sequential-thinking enforcement omitted, `validation_tier: Fallback` declared in all output fields.

## Protocol Registry (L0)

Five primitive protocols. Each has a dedicated agent in `agents/` and a `references/procedure.md` containing the complete execution specification. Invoke a protocol by spawning its agent via the Task tool with the required inputs.

| Protocol               | Abbreviation | Agent                  | Purpose                                                                     |
| ---------------------- | ------------ | ---------------------- | --------------------------------------------------------------------------- |
| Adversarial Cycle      | AC           | adversarial-cycle      | Claim stress-testing via conclusion → counter-claim → response cycles       |
| Downstream Forecasting | DF           | downstream-forecasting | Cascade impact tracing with severity-gated termination                      |
| Tradeoff Analysis      | TA           | tradeoff-analysis      | Scored alternative comparison with sensitivity analysis                     |
| Assumption Surfacing   | AS           | assumption-surfacing   | Foundation audit with fragility rating and epistemic status tagging         |
| Plan Review            | PR           | plan-review            | Adversarial plan critique with assumption surfacing and risk identification |

A sixth agent, `domain-researcher`, handles targeted knowledge acquisition - spawned by protocol agents (especially AS Phase 5) when domain gaps are identified.

## Composition Patterns (L1)

Two documented patterns for direct inter-protocol sequencing. Every inter-stage decision is fully determined by data flowing through the pipeline - no contextual evaluation or protocol re-invocation mid-flight. If any stage requires re-evaluation or re-invocation, it is a pipeline with adapters (L2), not a pattern.

**Orchestration:** Spawn the first agent via the Task tool, read its structured output, apply the gate logic, then spawn the second agent with the piped output as input.

### Validate-and-Forecast

**Purpose:** Stress-test a claim via adversarial challenge, then forecast downstream impact of the surviving claim. Prevents investment in impact analysis of flawed conclusions.

**Shape:** Gated pipeline - Adversarial Cycle → Validation Gate → Downstream Forecasting

**Stage sequence:**

- S1: Adversarial Cycle (full protocol) → `validation_status`, `final_claim`, `cycle_transcript`
- S2: Validation Gate - routes on `validation_status`. Validated or Modified → S3. Refuted → pattern terminates with refutation result (gate is logical entailment: forecasting a false claim produces nonsense). Early-exit (claim unfalsifiable within scope) → proceed to S3 with the original claim unchanged.
- S3: Downstream Forecasting (full protocol) - receives `final_claim` (original if validated, REVISED if modified). Output must surface any modification visibly so caller can verify.

**Output mapping:** Adversarial Cycle `final_claim` → Downstream Forecasting `decision` input. When modified, DF operates on the modified claim, not the original. For early-exit, `final_claim` is absent - the Validation Gate passes the original pattern `claim` input to DF unchanged.

**Caller params:** Shared: `decision_context` (AC framing + DF scope). Stage-specific: `claim` + `supporting_argument` (AC), `scope_boundary` (DF, termination per severity-gated design). Optional: `severity_threshold` (DF termination).

**Completion paths:**

- Path A (Validated/Modified/Early-exit): claim + cycle transcript + severity propagation results. For early-exit, the original claim is used unchanged.
- Path B (Refuted): refutation result + cycle transcript. No severity propagation - this is a high-value outcome: prevented analysis of a flawed conclusion.

**Anti-patterns:** Forecasting a refuted claim (gate prevents by logical necessity); ignoring modification (forecast applies to revised claim); using this pattern when assumptions need testing (use Assumption-and-Forecast instead).

### Assumption-and-Forecast

**Purpose:** Surface assumptions underlying a decision, assess their fragility, and forecast the downstream impact of fragile assumptions - cleanly separating grounded analysis from coverage gaps.

**Shape:** Bifurcating pipeline - Assumption Surfacing → Routing Gate → {Downstream Forecasting ∥ Coverage Gap passthrough}

**Stage sequence:**

- S1: Assumption Surfacing (full protocol, all phases) → unified inventory with `epistemic_status` tags per assumption
- S2: Routing Gate - partitions on `epistemic_status`: `with-basis` → S3, `without-basis` or `ungrounded` → S4 passthrough. Mandatory classifier, not a filter: always runs even if one partition is expected empty.
- S3: Downstream Forecasting (full protocol) - invoked unconditionally regardless of partition size. Empty input → well-formed empty output (protocol owns the empty-result contract; avoids coupling the pattern to the protocol's output schema).
- S4: Output Merge - severity propagation results + coverage gap passthrough

**Critical distinction:** Assumption Surfacing Phase 4 dependency graph is epistemic (which assumptions depend on others). Downstream Forecasting's graph is causal (impact propagation). These are different structures answering different questions. Do not transfer the epistemic graph to DF.

**Without-basis routing:** Structural importance scores from Assumption Surfacing attach to `without-basis` entries as prioritization metadata only - NOT consumed as substantive fragility input in severity propagation. Feeding structural importance scores into severity propagation is a category error.

**Coverage gap ordering:** Single-axis - structural importance score. Identical scores annotated as a tie group with consumer guidance: "equivalent structural importance; selection between these should be driven by domain-specific considerations outside the pattern scope." Prevents silent arbitrary tiebreaking.

**Caller params:** Shared: `decision_context`, `scope_boundary` (AS Phase 1 bounding + DF termination scope). Stage-specific: `domain_grounding_source` (AS Phase 1). Optional: `acceptance_threshold` (AS quality floor), `severity_threshold` (DF termination).

**Degenerate cases:** All with-basis → linear degradation, empty coverage gaps. All without-basis → DF invoked with empty input, full gap inventory. Empty inventory → "no assumptions identified" signal.

**Anti-patterns:** Feeding `without-basis` scores into severity propagation; skipping routing gate when all assumptions appear grounded (gate validates, does not assume); treating coverage gaps as lower-priority than propagated severities (unknown risk ≠ low risk).

### Pattern Selection Guide

| Dimension           | Validate-and-Forecast                                   | Assumption-and-Forecast                                            |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------------------------ |
| Question answered   | Does this conclusion hold, and what happens if it does? | What are we assuming, how fragile, and what breaks?                |
| Entry point         | Specific claim with supporting argument                 | Decision context (claim may not exist yet)                         |
| Gate type           | Binary proceed/terminate (validation status)            | Partitioning classifier (epistemic status)                         |
| Gate semantics      | Logical entailment: can't forecast false claims         | Prevents conflation of structural importance with domain fragility |
| DF receives         | Single validated/modified claim                         | Set of fragile assumptions with grounded ratings                   |
| Failure mode output | Refutation result (high-value: don't proceed)           | Coverage gap inventory (high-value: unknown risk surface)          |

**Boundary criterion - pattern vs pipeline:** Can every inter-stage decision be fully determined by data flowing through the pipeline? If yes → pattern. If any decision requires contextual evaluation or protocol re-invocation → pipeline with adapters (L2).

## Pipeline Registry (L2)

Three validated pipelines. Pipelines use adapters to handle inter-stage orchestration where a pattern's pure data-flow constraint would be violated. Each stage spawns its protocol agent via the Task tool; adapters run between agent invocations to reshape output into the next agent's input contract.

**Adapter constraint - orchestration only.** Adapters reshape data between protocol output/input contracts. Adapters do not generate analytical conclusions, infer classifications, or produce structured assessments that a protocol is designed to produce. Diagnostic: if an adapter requires analytical judgment to perform its transformation, the pipeline is missing a protocol invocation.

**Adapter transformation types:**

- _Bifurcating_: one protocol output → two different input shapes fed to different protocols simultaneously
- _Synthesizing_: structured ranking output → singular claim formulation fed to the next protocol

**Structural constraints on adapters:**

1. Downward-only invocation: adapters invoke L0 protocols, never L1 patterns or L2 pipelines. The call graph is a DAG by construction - no cycles possible.
2. Static max-invocation bound: each adapter spec declares the maximum re-invocations per protocol. Total cost is statically analyzable before execution.
3. Execution context threading: adapters pass protocol output as scoped input to the next protocol within a single pipeline run. This is ephemeral data flow, not persistent state - it dies with pipeline completion and does not violate the stateless design principle.

| Pipeline | Shape        | Adapter Type | Purpose                                                                        |
| -------- | ------------ | ------------ | ------------------------------------------------------------------------------ |
| A-V-F    | AS → AC + DF | Bifurcating  | Surface assumptions, stress-test decision, forecast fragile assumption impacts |
| T-V      | TA → AC      | Synthesizing | Compare alternatives, stress-test the winning selection                        |
| T-V-F    | TA → AC → DF | Synthesizing | Full decision pipeline: compare, stress-test, forecast                         |

**Banned adapter combinations:**

- TA → DF (direct): adapter must synthesize fragility assessments from option scores - fragility assessment is Assumption Surfacing's analytical function. Fix: insert AS between TA and DF.
- AS → TA (direct): adapter must infer evaluation criteria from surfaced assumptions - criteria derivation requires analytical judgment beyond orchestration scope.

## Deferred to v2

- **Consistency Audit** - logical contradiction, redundancy, and coverage-gap detection across a claim set
- **Root Cause Analysis** - backward causal tracing from a failure or unexpected outcome
- Pipelines dependent on these protocols (e.g., AS → Consistency Audit, DF → Root Cause Analysis)
