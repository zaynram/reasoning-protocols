# Tradeoff Analysis

Pre-decision structured comparison. Produces a scored comparison of alternatives with sensitivity bounds. The caller decides - this is not a recommendation engine.

## Input Contract

- `decision` (required): The question being decided.
- `alternatives` (required): Options under consideration (≥2).
- `seed_criteria` (required, ≥1): Initial evaluation dimensions, each with a `label` and `operational_definition`. Prompt the caller for missing definitions before proceeding.
- `scope_boundary` (optional): Constraints on consideration.
- `weighting_context` (optional): Caller context for relative importance. On re-invocation, may carry prior remediation hints as refinement priorities.

## Phase 1 - Criteria Enumeration

*(Sequential-thinking required)*

**Step 1 - Receive seed criteria.** Each must include label + operational definition. Prompt caller for missing definitions before proceeding.

**Step 2 - Generate candidate criteria (seq-thinking).** Three derivation sources:
- (a) *Alternative-implied*: dimensions on which the alternatives structurally differ from each other
- (b) *Context-implied*: dimensions the decision context introduces (constraints, stakeholders, timelines)
- (c) *Gap-fill*: standard evaluation dimensions the seed set missed

Each candidate: label + definition, tagged `protocol-suggested`. Seeds tagged `caller-supplied`.

**Step 3 - Caller confirmation.** Present the combined set. Caller confirms, rejects, or modifies each criterion. Guardrails:
- Fewer than 3 confirmed → warning: sensitivity analysis has limited discriminative value; one criterion flip can reverse the result
- More than 8 confirmed → warning: likely marginal or correlated criteria; combinatorial sensitivity load increases

Warnings are informational - proceed if caller confirms.

**Step 4 - Completeness check (seq-thinking).** After confirmation: are obvious evaluation dimensions conspicuously absent given the decision and alternatives? Surface gaps for caller awareness. Informational, not blocking.

**Step 5 - Correlation flagging (seq-thinking).** Per pair of confirmed criteria: if an alternative's ranking on criterion A changed, would B's ranking likely change in the same direction? Output per pair:
- `correlated`: yes/no
- `direction` (if yes): positive (both shift same way) or negative (both shift opposite ways)

No strength qualifier - stays ordinal-consistent. **Cluster concentration warning:** If the majority of criteria form a single correlated cluster, flag that the evaluation may be capturing one underlying dimension from multiple angles.

Phase 1 output: Confirmed criteria set (label, definition, sourcing tag) + completeness notes + correlation pairs with directions.

## Phase 2 - Weighting Justification

**Step 1 - Present confirmed criteria set** from Phase 1.

**Step 2 - Assign weights (seq-thinking).** Per criterion: a percentage weight (all sum to 100%). Each justification must:
- (a) Reference a specific property of the decision context
- (b) Explicitly compare importance to ≥1 other criterion with cross-reference pointers

**Step 3 - Normalize.** Verify weights sum to 100%. If rounding drift, normalize and note the adjustment.

**Step 4 - Coherence check.** Verify weight ordering (highest to lowest) is consistent with justification narratives. If criterion A is weighted higher than B, no justification should rely on B being more important than A. Internal contradictions are flagged for caller resolution before proceeding.

Phase 2 output per criterion: Label, weight (%), justification text, referenced criteria list.

## Phase 3 - Per-Alternative Scoring

*(Sequential-thinking required)*

**Step 1 - Construct matrix scaffold.** Alternatives as rows, criteria as columns.

**Step 2 - Score criterion-by-criterion (seq-thinking).** Process one criterion at a time across all alternatives simultaneously - not alternative-by-alternative. Ranking uses **dense ranks**: 1 = best, ties share rank, next rank increments by 1 (e.g., 1, 1, 2 - not 1, 1, 3).

Per cell: rank position + mechanistic argument that:
- (a) References specific alternative properties
- (b) Is grounded in the Phase 1 criterion definition
- (c) Is sufficient to reconstruct the ranking from the arguments alone

**Step 3 - Cross-check.** Scan for structural anomalies:
- *Dominant alternative*: ranks 1st on all criteria → criteria may not be independent
- *Dominated alternative*: ranks last on all criteria → may not belong in the set, or criteria may be biased
- *Non-discriminating criterion*: all alternatives tied → zero discriminative power, weight is wasted
- *Correlation contradiction*: rank patterns contradict Phase 1 correlation flags

All flags are informational - not blocking.

Phase 3 output: Score matrix (alternative × criterion) with dense ranks + per-cell mechanistic arguments.

## Phase 4 - Sensitivity Analysis

*(Sequential-thinking required)*

Phase 4 serves a dual role: sensitivity analysis AND convergence check.

**Step 1 - Compute weighted rank-sums.** Per alternative: sum of (rank × weight/100) across all criteria. Lower = better. Produce the final ordering.

**Step 2 - Derive independent flip conditions.** For each adjacent pair in the ordering (focus: rank 1 vs rank 2), compute the score gap, then:
- (a) *Weight-sensitivity*: per criterion, what weight shift is required to reverse the ordering?
- (b) *Rank-sensitivity*: per criterion, what rank change is required to reverse the ordering?

**Step 3 - Model correlated clusters.** For each flip condition involving a criterion in a correlated pair or cluster: recompute the threshold modeling the cluster as a unit (rank change on one → simultaneous change on all correlated members in the flagged direction). Compare clustered vs independent threshold. Flag material differences - correlation can amplify or dampen the flip.

**Step 4 - Generate remediation hints.** Per flip condition:
- `upstream_phase`: `2` (weighting) or `3` (scoring)
- `target`: criterion label (Phase 2 target) or (alternative, criterion) cell (Phase 3 target)
- `argument_quality`: structural assessment - `well-argued` or `weakly-argued` - with brief explanation grounded in specific justification properties
- `precision_dependent` (Phase 2 targets only): perturbation test - would the justification hold without modification at ±10% of the assigned weight? If yes → loosely anchored, flip condition is precision-dependent. If no → tightly anchored. Not applicable to Phase 3 targets (ordinal ranks have no precision problem).
- `remediation_action`: what the caller would reconsider

**Step 5 - Convergence assessment.** Signal (analysis is robust): no flip condition (a) targets a weakly-argued cell, OR (b) is precision-dependent on a loosely-anchored weight. If both are clear → `convergence_status: robust`. If not → `convergence_status: refinement-needed`, with remediation hints indicating the refinement focus. Re-invocation is always full protocol (stateless by design - no skip/resume). `weighting_context` can carry prior remediation hints to focus a re-run.

Phase 4 output: Weighted rank-sums + final ordering, flip conditions (independent + clustered), correlated_clusters, remediation hints with argument_quality + precision_dependent, convergence_status.

## Output Contract

- `criteria_assessment`: Confirmed criteria set (label, definition, sourcing tag) + completeness notes + correlation pairs with directions
- `weighted_criteria`: Criteria with percentage weights + justifications + cross-references
- `score_matrix`: Alternative × criterion matrix with dense ranks + per-cell mechanistic arguments
- `sensitivity_report`: Flip conditions (independent + clustered), correlated_clusters, remediation hints (argument_quality + precision_dependent), convergence_status
- `validation_tier`: `Full` | `Fallback`

## Edge Cases

- **Fewer than 3 criteria:** Warning at Phase 1 Step 3. Proceed if caller confirms. Sensitivity analysis has low discriminative value.
- **Exactly 2 alternatives:** Simplest case - no special handling. Dense ranking with 2 items is unambiguous.
- **All alternatives tied on a criterion:** Flagged as non-discriminating in Phase 3 cross-check. Zero discriminative power in Phase 4 - no flip condition targets it.
- **Cluster concentration (majority of criteria in one cluster):** Flagged at Phase 1 Step 5. Evaluation may be measuring one underlying dimension repeatedly.
- **Dominant alternative (ranks 1st on all criteria):** Flagged in Phase 3. Re-examine whether criteria are independent.
- **Dominated alternative (ranks last on all criteria):** Flagged in Phase 3. Consider whether this alternative belongs in the set.
- **Re-invocation:** Always full protocol. No skip/resume. `weighting_context` carries prior remediation hints to guide criteria weighting focus without skipping Phase 1 confirmation.

## Fallback Tier

When sequential-thinking is unavailable:
- Sequential-thinking enforcement at Phase 1 Steps 2, 4, 5, Phase 2 Step 2, Phase 3 Step 2, and Phase 4 is omitted - perform as careful in-line deliberation
- All output format contracts remain identical
- `validation_tier: Fallback` appears in output
- Dense ranking and convergence assessment still apply; document reasoning explicitly in-line
