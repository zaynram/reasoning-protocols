# Assumption Surfacing

Audit the assumption foundation underlying a decision. Surfaces all implicit assumptions, assesses their fragility with explicit quality gating, builds an epistemic dependency graph, and produces a unified inventory tagged by epistemic status. Designed for composition with Downstream Forecasting via the Assumption-and-Forecast pattern.

## Input Contract

- `decision_context` (required): The decision whose assumption foundation is being audited. Should be specific enough to bound the assumption space.
- `domain_grounding_source` (required): The source from which Phase 1 derives covered domains and dependency topology - a document, specification, known body of knowledge, or explicit domain description. The quality of Phase 1 determines the quality of the quality floor check in Phase 3.
- `scope_boundary` (optional): Explicit constraints on which assumptions to surface. When composed with Downstream Forecasting (Assumption-and-Forecast pattern), the scope boundary is shared between this protocol and DF.
- `acceptance_threshold` (optional): Caller-supplied confidence threshold for fragility ratings. No protocol default. This is distinct from the protocol-owned quality floor - the acceptance threshold reflects the caller's risk tolerance; the quality floor reflects whether the protocol can produce a non-arbitrary rating at all.

## Epistemic Status

Every assumption in the unified output carries an `epistemic_status` tag:

- `with-basis`: Protocol constructed a mechanistic argument directly addressing whether the assumption is likely to hold or break. The fragility rating reflects analytical grounding.
- `without-basis`: Protocol could not construct a mechanistic argument grounded in Phase 1's covered domains. Fragility characterized through structural inference - position in the epistemic dependency graph (structural importance). The rating answers "how structurally important is this assumption?" not "how likely is it to break?" Protocol never claims domain knowledge it does not have.

**Consumer contract:** `epistemic_status` is non-optional. Composition interfaces (Routing Gate in Assumption-and-Forecast) must enforce the distinction. Feeding `without-basis` structural importance scores into severity propagation is a category error - unknown risk is not the same as low risk.

## Phase 1 - Domain Topology Enumeration

*(Sequential-thinking required)*

Establish the covered domain landscape from which Phase 3's quality floor draws its grounding check.

**Step 1 - Scope binding.** If `scope_boundary` is provided, confirm it is coherent with `decision_context`. Surface a warning if the boundary appears under-scoped (coherent but narrow). Proceed after confirmation.

**Step 2 - Domain enumeration (seq-thinking).** From `domain_grounding_source`, enumerate the covered domains relevant to the decision:
- What subject matter areas does the source cover?
- What is explicitly in scope vs out of scope?
- What dependencies exist between covered domains (which domains presuppose knowledge of others)?

Per domain: label + coverage description + dependency links to other domains.

**Step 3 - Topology construction.** Arrange the covered domains into a dependency topology: which domains depend on which others? This topology is the structural map Phase 3's quality floor uses to determine whether an assumption's subject matter falls within covered territory.

**Step 4 - Coverage gap characterization.** Identify domains that are plausibly relevant to the decision but are absent from `domain_grounding_source`. These are anticipated grounding gaps - not the same as assumption gaps, but signals for where Phase 3 floor checks are likely to fail.

Phase 1 output: Covered domain inventory (label, coverage description), domain dependency topology, anticipated coverage gaps.

## Phase 2 - Assumption Enumeration

*(Sequential-thinking required)*

Surface all assumptions implicit in the decision.

**Step 1 - Direct assumption extraction (seq-thinking).** What must be true for the decision to be valid? For each:
- Assumption label + full statement
- Which aspect of the decision does this assumption support?

**Step 2 - Dependency-implied assumptions (seq-thinking).** For each domain in Phase 1's topology: what does operating in this domain presuppose? Are those presuppositions visible as explicit assumptions, or are they embedded? Surface embedded presuppositions as candidate assumptions.

**Step 3 - Completeness check.** Are any obvious assumption categories missing given the decision? Common categories: temporal (assumptions about timing or sequence), resource (assumptions about availability), behavioral (assumptions about stakeholder or system behavior), causal (assumptions about mechanism). Surface gaps informally - not blocking.

**Step 4 - Deduplication.** Merge assumptions that are equivalent under different phrasings. Note the merge and retain the clearest statement.

Phase 2 output: Assumption inventory (label, full statement, supporting relationship to decision).

## Phase 3 - Fragility Rating with Quality Floor

*(Sequential-thinking required)*

Rate each assumption's fragility and determine whether the protocol can produce a grounded rating.

### Two-Step Quality Floor Check

The quality floor is protocol-owned: it determines whether the protocol can produce a non-arbitrary fragility rating. It is distinct from the caller-owned `acceptance_threshold`.

**Step 1 (Mechanical) - Domain coverage check:** Does the assumption's subject matter fall within Phase 1's reported covered domains? If the subject matter is in an explicit coverage gap, or absent from the domain topology → **floor fails**. Gap characterization derived from Phase 1's topology.

**Step 2 (Constrained judgment - fires only if Step 1 passes) - Argument constructibility check:** Can I construct an argument whose conclusion directly addresses whether this specific assumption is likely to hold or break? The argument must be mechanistic, not assertive - it must explain *why* the assumption is fragile or stable through a concrete mechanism or constraint. If no such argument can be constructed → **floor fails**. Gap characterization derived from the failed argument attempt (Phase 1 overclaimed coverage).

Floor output per assumption: `(floor_status: passed | failed, gap_characterization)`. Dual output enables the refinement loop (Phase 5) to issue targeted narrowing briefs.

### Fragility Assessment

**For floor-passed assumptions (with-basis path):**
Assess fragility: how likely is this assumption to break, and how impactful is a break?
- `fragility_rating`: Low / Medium / High
- `rating_argument`: mechanistic argument directly addressing whether the assumption is likely to hold or break
- `epistemic_status: with-basis`

**For floor-failed assumptions (without-basis path, pre-refinement):**
Rating derived from structural position in the Phase 2 dependency graph:
- `structural_importance`: computed in Phase 4 (how many other assumptions directly or transitively depend on this one)
- `epistemic_status: without-basis`
- Do not assign a fragility rating before Phase 4 completes - structural importance requires the dependency graph.

Phase 3 output: Per assumption - floor_status, gap_characterization (if failed), fragility_rating + rating_argument (if passed), epistemic_status tag.

## Phase 4 - Dependency Graph Tracing

*(Sequential-thinking required)*

Build the epistemic dependency graph: which assumptions depend on which others?

**Step 1 - Dependency identification (seq-thinking).** For each assumption pair: does validating assumption A require assumption B to hold? If yes, A depends on B. Record the dependency mechanism (not just the directional link).

**Step 2 - Graph construction.** Arrange dependencies as a directed graph: edges point from dependent to dependency (A → B if A depends on B). Cycles indicate logical circularity - flag as inconsistencies for caller awareness.

**Step 3 - Structural importance scoring.** For each assumption: count the number of other assumptions that transitively depend on it (transitive fan-in). Higher count = higher structural importance. This score is the basis for `without-basis` characterization - it answers how structurally central the assumption is, not how fragile.

**Step 4 - Annotation for grounded dependencies on ungrounded assumptions.** For each `with-basis` assumption: does its `rating_argument` reference the domain of any `without-basis` assumption? If yes, annotate with `ungrounded-basis` tag. This annotation is metadata - it does not change the fragility rating. It signals to consumers that the grounded rating has a dependency on an uncharacterized domain.

**Step 5 - Assign structural importance to without-basis assumptions.** Using the scores from Step 3, assign `structural_importance` scores to all `without-basis` assumptions. These scores drive coverage gap ordering in the Assumption-and-Forecast composition pattern.

Phase 4 output: Epistemic dependency graph (directed), structural importance scores per assumption, `ungrounded-basis` annotations.

## Phase 5 - Refinement Loop

*(+1 convergence instantiation - one retry)*

For each `without-basis` assumption from Phase 3: attempt one targeted re-invocation of Phase 1 using domain-researcher agent delegation to narrow the domain grounding source.

**Step 1 - Issue targeted briefs.** Per `without-basis` assumption: construct a narrow brief for targeted domain-researcher agent delegation. The brief specifies:
- The assumption's label and full statement
- The domain gap identified in Phase 3 Step 1 or 2 (gap_characterization)
- The specific domain knowledge needed to construct a grounding argument
- A narrowed `domain_grounding_source` directive (subset of the original, or alternative source)

**Step 2 - Re-invoke Phase 1 via domain-researcher agent delegation.** Delegate each brief to a domain-researcher agent. Each domain-researcher agent runs Phase 1 for its targeted assumption domain. Domain-researcher agents run in parallel when possible (independent domains).

**Step 3 - Re-enter Phase 3.** For each assumption where Phase 1 re-invocation produced new covered domain data: re-apply the quality floor check with the expanded domain topology. If the floor now passes → run the full fragility rating and argument construction for that assumption → update `epistemic_status` to `with-basis`.

**Step 4 - Mark residually ungrounded assumptions.** If the floor still fails after one retry: mark assumption as `epistemic_status: ungrounded`. This is distinct from `without-basis` (which has structural importance) - `ungrounded` signals that even a targeted domain expansion could not produce grounding.

**Convergence:** The refinement loop terminates after one retry. Convergence signal: all assumptions are either `with-basis`, `without-basis` (with structural importance scored), or `ungrounded`. An additional pass cannot improve the output - either grounding exists in the targeted domain or it does not.

Phase 5 output: Updated epistemic status per assumption (potentially upgraded from `without-basis` to `with-basis`), `ungrounded` flags for residually ungroundable assumptions.

## Phase 6 - Output Synthesis

Produce the unified inventory and filtered views.

**Unified inventory:** All assumptions from Phase 2, annotated with:
- `epistemic_status`: `with-basis` | `without-basis` | `ungrounded`
- `fragility_rating` + `rating_argument` (for `with-basis` assumptions)
- `structural_importance` score (for `without-basis` and `ungrounded` assumptions)
- `ungrounded-basis` annotation (for `with-basis` assumptions whose arguments reference ungrounded domains)
- Dependency graph position (direct dependents and dependencies)

**Unanalyzed risks view:** A filtered presentation over the unified inventory - the subset of assumptions with `epistemic_status: without-basis` or `ungrounded`, ordered by structural importance (descending). Identical structural importance scores within this view are annotated as a tie group. This view is a presentation layer, not a separate data path - the structural data lives once in the unified inventory.

**Summary fields:**
- `grounded_count`: count of `with-basis` assumptions
- `without_basis_count`: count of `without-basis` assumptions (characterized by structural importance)
- `ungrounded_count`: count of assumptions that resisted grounding after one retry
- `validation_tier`: `Full` | `Fallback`

## Output Contract

- `unified_inventory`: All assumptions with epistemic status, fragility ratings (where applicable), structural importance scores (where applicable), ungrounded-basis annotations, and dependency positions
- `dependency_graph`: Directed epistemic dependency graph
- `unanalyzed_risks`: Filtered view of without-basis and ungrounded assumptions ordered by structural importance
- `summary`: grounded_count, without_basis_count, ungrounded_count, validation_tier

## Composition Interface (Assumption-and-Forecast)

The `unified_inventory` is the output consumed by the Routing Gate in the Assumption-and-Forecast composition pattern:
- `with-basis` assumptions → enter Downstream Forecasting's severity propagation via fragility ratings
- `without-basis` and `ungrounded` assumptions → route to coverage gap handling; structural importance is prioritization metadata, NOT a fragility input. Both carry structural importance scores (computed in Phase 4 before Phase 5 refinement). `ungrounded` additionally signals that a targeted domain expansion was attempted and failed.

This routing is documented in SKILL.md (Composition Patterns). It is not enforced in this protocol - this protocol produces tagged output; the pattern enforces the routing.

## Edge Cases

- **No assumptions found:** "No assumptions identified" signal in unified inventory. Well-formed empty output - not an error.
- **All assumptions with-basis:** No refinement loop needed. `unanalyzed_risks` is empty.
- **All assumptions without-basis:** Phase 1 domain source lacks coverage for the decision domain. Refinement loop runs for all assumptions. Consider whether `domain_grounding_source` was appropriate for this decision.
- **Circular dependency detected:** Flag as logical circularity in Phase 4 output. Escalate to caller - circular dependencies indicate a structural inconsistency in the assumption set itself.
- **Subagent-delegation unavailable:** Skip refinement loop (Phase 5). All `without-basis` assumptions remain as-is. Note in output that refinement loop was skipped. `validation_tier: Fallback` if sequential-thinking also unavailable; otherwise `Full` tier with a noted limitation.
- **Empty `domain_grounding_source`:** Phase 1 produces empty domain topology. Phase 3 floor fails for all assumptions at Step 1. Refinement loop receives briefs with no narrowed source to target - mark all as `ungrounded` after one retry with empty source.

## Fallback Tier

When sequential-thinking is unavailable:
- Sequential-thinking enforcement at Phase 1 Step 2, Phase 2 Steps 1–2, Phase 3 fragility assessment, and Phase 4 Step 1 is omitted - perform as careful in-line deliberation
- All output format contracts remain identical
- Refinement loop (Phase 5) still runs via `domain-researcher` agent delegation
- `validation_tier: Fallback` appears in output
