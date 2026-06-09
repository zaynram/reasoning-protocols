---
name: reasoning-chain
description: This skill should be used when deep analytical reasoning is required, particularaly when there are identifiable factor "groupings" that would benefit from scope isolation (signal clarity) or sequential ordering (dependent analysis).
user-invocable: true
compatibility: Sequential-thinking is required for Full-tier execution and enforces phase discipline. Without sequential-thinking, protocols degrade to [Fallback](#fallback-tier) tier.
---

# Reasoning Chain

This skill defines a "reasoning chain" as the composition of one or more `reasoning protocols` to provide intelligent, targeted analysis (usually on highly complex or interwoven scopes).

There are two types of "reasoning chains": composition `patterns` and registered `pipelines`.

- Composition `patterns` are _simple_ chains; they are purely sequential invocations of the baseline `reasoning protocols`.
- Registered `pipelines`; however, are _complex_ chains; they use adapters to handle inter-stage orchestration where a pattern's pure data-flow constraint would be violated.

## Domain Research

Grounding is a necessary aspect of the "reasoning chain" as it is the landscape of which analysis takes place upon.
Poorly or sparsely grounded contexts may compose a perfectly valid "reasoning chain" with no logical flaws and still be inaccurate or misguided due to a flawed context.

This effect is to be mitigated through generous usage of the  [`domain-researcher`](../../agents/domain-researcher.md) subagent, which is an exploration agent geared towards **targeted knowledge acquisition and context hardening**.
This subagent is not strictly required; however, there are few cases where it will not benefit the overall "reasoning-chain" and should be used by default unless explicitly directed to the contrary.

The proper workflow for using the `domain-researcher` subagent mandates the first invocation must prepend the entire "reasoning chain" and 
it should be _reused_ via the `SendMessage` tool should there arise a need for extended research, clarification, or scope refinement. 

## Protocols

Each has a dedicated agent in `agents/` and a `references/procedure.md` containing the complete execution specification.
Protocols are invoked by spawning the respective agent via the Task tool with the required inputs.

Individually, they are side-effect-free; dispatchers spawn agents and receive structured output. This requirement is supported by design with respect to these principles:

- Persistent state, lifecycle machinery, and enforcement logic are the responsibility of the dispatcher (you).
- Mid-pipeline failure has no checkpoint/resume; thus, protocols are side-effect-free and re-invocation from scratch is always safe.
- There is no predetermined number of passes defined for completion criteria; instead, a protocol terminates when an additional pass _cannot further improve the output_.

The four analytical protocols that may be used in the composition of a "reasoning chain", each have a **convergence instantiation** (minimum condition) and a **convergence signal** (sufficient condition).

### [Adversarial Cycle](../adversarial-cycle/SKILL.md)

**Abbreviation:** AC
**Purpose:** Claim stress-testing via conclusion → counter-claim → response cycles
**Convergence Instantiation:** Native termination semantics
**Convergence Signal:** No surviving counter-claim after a full cycle

### [Downstream Forecasing](../downstream-forecasting/SKILL.md)

**Abbreviation:** DF
**Purpose:** Cascade impact tracing with severity-gated termination
**Convergence Instantiation:** Amplification override (Phase 2)
**Convergence Signal:** Severity propagation stabilizes

### [Tradeoff Analysis](../tradeoff-analysis/SKILL.md)

**Abbreviation:** TA
**Purpose:** Scored alternative comparison with sensitivity analysis
**Convergence Instantiation:** Adaptive convergence check (Phase 4)
**Convergence Signal:** No flip condition targets a weakly-argued cell AND no flip condition is precision-dependent on a loosely-anchored weight

### [Assumption Surfacing](../assumption-surfacing/SKILL.md)

**Abbreviation:** AS
**Purpose:** Foundation audit with fragility rating and epistemic status tagging
**Convergence Instantiation:** Refinement loop (Phase 5)
**Convergence Signal:** All assumptions grounded or explicitly ungrounded after retry

## Composition Patterns

Every inter-stage decision is fully determined by data flowing through the pipeline; no contextual evaluation or protocol re-invocation mid-flight.
There are two prescribed composition `patterns` (validate-and-forecast, assumption-and-forecast); however, patterns may be constructed through any reasonable sequencing of multiple `reasoning protocols`.
If any stage requires re-evaluation or re-invocation, it is a `pipeline` with adapters, not a `pattern`.

**Orchestration:** Spawn the first agent via the Task tool, read its structured output, apply the gate logic, then spawn the second agent with the piped output as input.

### Pattern 1: Validate-and-Forecast

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

### Pattern 2: Assumption-and-Forecast

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

### Prescribed Pattern Selection Guide

| Dimension           | Validate-and-Forecast                                   | Assumption-and-Forecast                                            |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------------------------ |
| Question answered   | Does this conclusion hold, and what happens if it does? | What are we assuming, how fragile, and what breaks?                |
| Entry point         | Specific claim with supporting argument                 | Decision context (claim may not exist yet)                         |
| Gate type           | Binary proceed/terminate (validation status)            | Partitioning classifier (epistemic status)                         |
| Gate semantics      | Logical entailment: can't forecast false claims         | Prevents conflation of structural importance with domain fragility |
| DF receives         | Single validated/modified claim                         | Set of fragile assumptions with grounded ratings                   |
| Failure mode output | Refutation result (high-value: don't proceed)           | Coverage gap inventory (high-value: unknown risk surface)          |

**Boundary criterion - pattern vs pipeline:** 
Can every inter-stage decision be fully determined by data flowing through the pipeline? If yes → `pattern`. 
If any decision requires contextual evaluation or protocol re-invocation → `pipeline` with adapters.

## Registered Pipelines

Each stage spawns its protocol agent via the Task tool; adapters run between agent invocations to reshape output into the next agent's input contract.

### Adapters

**Orchestration Constraints:**

- Adapters reshape data between protocol output/input contracts.
- Adapters do not generate analytical conclusions, infer classifications, or produce structured assessments that a protocol is designed to produce.
- If an adapter requires analytical judgment to perform its transformation, the pipeline is missing a protocol invocation.

**Structural Constraints:**

- _Downward-only invocation_: adapters invoke `protocols`, never composition `patterns` or nested `pipelines`. The call graph is a DAG by construction; no valid cycles are possible.
- _Static max-invocation bound_: each adapter spec declares the maximum re-invocations per protocol. Total cost is statically analyzable before execution.
- _Execution context threading_: adapters pass protocol output as scoped input to the next protocol within a single pipeline run. This is ephemeral data flow, not persistent state - it dies with pipeline completion and does not violate the stateless design principle.

**Transformation Types:**

- _Bifurcating_: one protocol output → two different input shapes fed to different protocols simultaneously
- _Synthesizing_: structured ranking output → singular claim formulation fed to the next protocol

### Invalid Pipelines

These pipelines have been considered and rejected on account of their ineffectiveness and should not be executed under any circumstances, unless given explicit direction to do so.

- TA → DF (direct): adapter must synthesize fragility assessments from option scores - fragility assessment is Assumption Surfacing's analytical function. Fix: insert AS between TA and DF.
- AS → TA (direct): adapter must infer evaluation criteria from surfaced assumptions - criteria derivation requires analytical judgment beyond orchestration scope.

### Pipeline Registry

The `pipeline` registry contains three pre-validated entries anticipated to be most common and most generally applicable. 

| Pipeline | Shape        | Adapter Type | Purpose                                                                        |
| -------- | ------------ | ------------ | ------------------------------------------------------------------------------ |
| A-V-F    | AS → AC + DF | Bifurcating  | Surface assumptions, stress-test decision, forecast fragile assumption impacts |
| T-V      | TA → AC      | Synthesizing | Compare alternatives, stress-test the winning selection                        |
| T-V-F    | TA → AC → DF | Synthesizing | Full decision pipeline: compare, stress-test, forecast                         |

The registry is not exhaustive; more specific `pipelines` suiting individual cases may be composed so long as they are not refuted as an [invalid pipeline](#invalid-pipelines).

## Fallback Tier

Consists of the same output format contract and same sufficiency criteria, but with sequential-thinking enforcement omitted and `validation_tier: Fallback` declared in all output fields.
