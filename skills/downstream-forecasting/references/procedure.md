# Downstream Forecasting

Trace the cascade effects of a decision through its downstream dependents, with severity-gated termination and amplification detection. Produces a structured forecast showing where the decision propagates, at what severity, and where cascades terminate.

## Input Contract

- `decision` (required): The decision being forecasted, stated specifically and falsifiably. Vague decisions produce incoherent impact boundaries.
- `scope_boundary` (required): The set of systems, components, or domains in play. Dependents outside this boundary are not traced. Boundary-exceeded cascades escalate to caller rather than silently truncating.
- `decision_context` (optional): Broader context from which the decision emerged. Used when composed with Adversarial Cycle (Validate-and-Forecast pattern) - the cycle transcript provides framing the forecasting phase can use.
- `severity_threshold` (optional): Minimum severity to include in cascade tracing. Default behavior traces all non-None impacts. Caller-supplied threshold enables scoped forecasting for high-volume dependency graphs.

**Scope coherence validation (Phase 1, Step 1):** Verify that `scope_boundary` is coherent with `decision` - the boundary must encompass the decision's primary impact surface. If the boundary is coherent but narrow (under-scoped), surface a warning and proceed. If the boundary excludes the decision's own domain, flag as incoherent and request clarification before proceeding. Under-scoping risk: a coherent but narrow boundary may miss consequential cascades; the pattern registry in SKILL.md provides guidance on appropriate scope for composition contexts.

## Severity Model

Four levels, ordered from least to most severe:

| Level               | Meaning                                                                            |
| ------------------- | ---------------------------------------------------------------------------------- |
| None                | Decision has no detectable impact on this dependent                                |
| Absorbed            | Dependent adapts internally; no external behavior change                           |
| Requires Adaptation | Dependent must change, but can continue operating during transition                |
| Breaks Constraint   | Dependent's constraints are violated; cannot continue operating without resolution |

**Default termination:** Trace terminates when a cascade reaches `Absorbed` or `None` severity (monotonic decay assumption). **Amplification override:** If a terminating dependent is flagged as amplification-capable (Phase 1), trace one additional recursive hop even at `Absorbed` - this catches cases where an `Absorbed` impact at an amplifying node re-escalates at its dependents. The override is recursive: if the additional hop also terminates at `Absorbed` on an amplification-flagged node, trace one more hop. Ultimate termination is guaranteed by the scope boundary.

## Phase 1 - Impact Surface Enumeration

*(Sequential-thinking required)*

**Step 1 - Scope coherence check.** Verify `scope_boundary` coherence with `decision` per the Input Contract validation rule. Surface warnings or request clarification before proceeding.

**Step 2 - Enumerate direct dependents.** Identify all systems, components, or domains within `scope_boundary` that have a direct dependency on the decision's domain. For each:
- Dependency mechanism: how does this dependent rely on the decision's domain? (interface, data, timing, resource, behavioral contract)
- Initial severity estimate: first-pass assessment of direct impact severity using the four-level model

**Step 3 - Amplification flagging.** Per enumerated dependent: does this dependent have the structural property to amplify a moderate impact into a severe one? Amplification characteristics: fan-out (one input triggers many outputs), multiplier effects (small input change → large output change), or threshold behavior (impact appears absorbed until a tipping point). Flag each dependent: `amplification-capable: yes/no` with brief rationale.

Phase 1 output: Enumerated direct dependents with dependency mechanisms, initial severity estimates, and amplification flags.

## Phase 2 - Cascade Tracing

*(Sequential-thinking required)*

For each direct dependent from Phase 1, trace second-order and deeper effects.

**Step 1 - Per-dependent cascade trace.** Starting from the direct dependent's impact, follow the dependency chain:
- At each hop: identify the next-level dependent, assess impact severity given the incoming effect, and record the cascade path.
- Apply severity-gated termination: stop when a hop reaches `Absorbed` or `None` - unless the terminating node is amplification-flagged.
- Apply amplification override: if terminating at `Absorbed` on an amplification-flagged node, trace one additional hop. If that hop also terminates at `Absorbed` on an amplification-flagged node, trace one more. Continue until: (a) a non-amplification-flagged node absorbs the impact, or (b) a `None` severity hop is reached, or (c) the scope boundary is hit.

**Step 2 - Boundary escalation.** If a cascade path reaches the scope boundary before terminating (severity is still `Requires Adaptation` or `Breaks Constraint` at boundary edge), do not silently truncate. Record the cascade path to boundary and flag as `boundary-exceeded: yes`. Escalate to caller after Phase 5 with the specific cascade path and the severity at boundary edge.

Phase 2 output: Per-dependent cascade paths with severity at each hop, amplification override applications, and boundary-exceeded flags.

## Phase 3 - Constraint Check

*(Mechanical - no sequential thinking required)*

For each dependent in the cascade graph: does the decision, or any cascade effect, violate a known constraint of that dependent system?

Constraint types to check: interface contracts (does the decision change an interface the dependent expects?), resource constraints (does the cascade produce resource demand the dependent cannot meet?), timing constraints (does the cascade introduce latency or ordering violations?), behavioral contracts (does the cascade produce outputs that violate the dependent's behavioral guarantees?).

Per violation: identify the specific constraint, the cascade path that produces the violation, and whether the violation is direct (Phase 1 dependent) or cascade-induced.

Phase 3 output: Constraint violations per dependent, with constraint type, cascade path, and directness.

## Phase 4 - Interaction Detection

*(Mechanical - no sequential thinking required)*

Identify convergent impacts: cases where multiple independent cascade paths from Phase 2 reach the same component.

**Step 1 - Convergence identification.** Scan all Phase 2 cascade paths. Identify any component that appears in two or more distinct paths (reached through different first-hop dependents from Phase 1).

**Step 2 - Re-evaluate severity at convergence.** For each convergent component: the combined severity from multiple arriving impacts may be higher than any individual path's severity. Re-evaluate using the severity model - if two `Requires Adaptation` impacts converge, the combined effect may `Break Constraint`.

**Step 3 - Re-trace escalated branches.** If re-evaluation at a convergence point escalates severity beyond what the original single-path assessment showed, re-trace any branches from that convergence point that had terminated at `Absorbed` or `None`. The escalated severity at the convergence point may re-activate cascade paths that were previously terminated.

Phase 4 output: Convergent components, re-evaluated severities, and any additional cascade paths revealed by re-tracing.

## Phase 5 - Forecast Summary

Produce a structured output consolidating all phases.

Per direct dependent entry:
- Dependent name and dependency mechanism
- Direct impact severity (Phase 1)
- Cascade path summary (Phase 2): hops, severity at each, termination reason
- Amplification override applications (if any)
- Constraint violations (Phase 3, if any)
- Convergent impact flag (Phase 4, if applicable) with re-evaluated severity

Summary-level fields:
- `boundary_exceeded`: list of cascade paths that hit the scope boundary with severity at edge
- `convergence_points`: components with re-evaluated severity post-convergence
- `validation_tier`: `Full` | `Fallback`

## Output Contract

- `impact_surface`: Enumerated direct dependents with dependency mechanisms and amplification flags
- `cascade_map`: Per-dependent cascade paths with severity at each hop
- `constraint_violations`: Per-dependent constraint violations (empty list if none)
- `convergence_points`: Components with converged impacts and re-evaluated severities (empty list if none)
- `boundary_exceeded`: Cascade paths that reached the scope boundary (empty list if none)
- `forecast_summary`: Human-readable per-dependent narrative consolidating all phases
- `validation_tier`: `Full` | `Fallback`

## Edge Cases

- **Single dependent in scope:** Simplest case - no convergence possible, cascade trace is linear. Amplification override still applies.
- **Empty dependency graph:** No direct dependents within scope boundary. `impact_surface` is empty. Output is a well-formed empty forecast - not an error.
- **All cascades absorb at first hop:** No second-order effects. Phase 4 has no convergence candidates. Output reflects this cleanly.
- **Decision affects a scope boundary node directly:** Flag the node as a direct dependent with `Breaks Constraint` severity and simultaneously as a boundary-exceeded cascade - the impact exists at the boundary, not within it.
- **Cascades from prior Adversarial Cycle (Validate-and-Forecast):** When composed, `decision` receives `final_claim` from the cycle - or the original pattern `claim` if the cycle produced an early-exit (unfalsifiable conclusion). If the claim was modified during the cycle, the forecast applies to the modified claim; the modification must be visible in the forecast output - do not silently elide the modification.

## Fallback Tier

When sequential-thinking is unavailable:
- Phase 1 and Phase 2 sequential-thinking requirements are omitted - perform enumeration and cascade tracing as careful in-line deliberation
- All output format contracts remain identical
- `validation_tier: Fallback` appears in output
- Amplification override logic still applies; document each override application explicitly in the cascade map even without sequential-thinking enforcement
