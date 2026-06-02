# Adversarial Cycle

Validate a claim through structured conclusion → counter-claim → response cycles with defined sufficiency requirements. Produces a determination of whether the claim survives scrutiny, and if so, in what form.

## Input Contract

- `claim` (required): The decision or position being stress-tested, stated as a falsifiable claim.
- `supporting_argument` (required): The reasoning supporting the claim. Provides the mechanistic base the counter-claim must attack.
- `decision_context` (optional): Broader context framing why this claim matters. Used to scope counter-claims and prevent premise rejection attacks.
- `mode` (optional): `individual` (default) or `aggregated`. See Scope Modes below.

## Output Contract

- `validation_status`: `validated` | `modified` | `refuted` | `early-exit`
- `final_claim`: The surviving claim after the cycle. Present when `validated` or `modified`. When `modified`, this is the revised form - callers must use this, not the original. Absent when `refuted` or `early-exit`.
- `cycle_transcript`: Formatted summary block(s) per the Output Format section.
- `validation_tier`: `Full` | `Fallback` (see Fallback Tier below)

## Purpose

Structured stress-testing for claims and decisions. Each claim passes through a three-phase cycle (conclusion → counter-claim → response) that must meet explicit sufficiency criteria before the claim is considered validated. The goal is not to reverse decisions but to ensure they survive scrutiny - and to surface modifications that strengthen them.

## Scope Modes

### Individual Mode
Test a single claim in isolation. Use when evaluating a specific design choice, architecture decision, or protocol rule.

### Aggregated Mode
Test a set of resolved decisions as a composed system. Use after multiple individual decisions have been validated to check for emergent interaction failures. Counter-claims in aggregated mode must target **interactions between decisions**, not re-litigate individual decisions already validated.

## Sequential Thinking Requirements

The adversarial cycle requires sequential-thinking tool use at two specific points to enforce phase discipline and prevent shortcutting. These are internal reasoning steps - their output feeds the visible cycle phases but is not reproduced directly in the response.

### Counter-Claim Setup (required)
Before writing the counter-claim, use sequential thinking to:
1. Enumerate at least two candidate attack vectors against the conclusion.
2. For each candidate, assess: (a) does it target the core mechanism or a peripheral aspect? (b) what is the severity of the failure scenario? (c) is it a strawman or premise rejection?
3. **Early exit evaluation:** If all candidates fail the non-strawman, independence, or specificity filters - no candidate targets the core mechanism with a plausible failure scenario - classify the conclusion as **unfalsifiable within scope** and exit the cycle. Document the candidates evaluated and the rejection reason for each.
4. Select the strongest candidate - the one that targets the core mechanism with the highest-severity failure scenario.
5. Verify the selected attack is not a strawman by restating the conclusion and confirming the attack addresses it as stated.

Minimum: 3 thoughts. The first enumerates candidates, the second evaluates and applies the early exit check, the third selects and verifies. If the early exit triggers, the second thought documents rejection reasons and the third confirms the exit classification.

### Response Setup (required)
Before writing the response, use sequential thinking to:
1. Enumerate at least two candidate resolutions to the counter-claim's failure scenario.
2. For each candidate, assess: (a) does it directly address the specific failure scenario, or does it sidestep to a related concern? (b) does it operate through a concrete mechanism or through assertion? (c) does it introduce a new failure mode?
3. Select the resolution that most directly addresses the failure scenario without introducing new vulnerabilities.
4. If the selected resolution modifies the original conclusion, state the modification explicitly.
5. Verify no residual vulnerability remains by re-reading the counter-claim's failure scenario and confirming it is either refuted or accommodated.
6. **Classify the resolution type:** If the failure scenario was shown to not apply under the conclusion's stated mechanism → **Refuted**. If the conclusion required only an implicit constraint or scope clarification without changing its mechanism → **Refuted with clarification**. If the conclusion's mechanism or scope was modified to eliminate or accommodate the failure scenario → **Accepted with modification**. If none of these apply, this is non-convergence.

Minimum: 4 thoughts. The first enumerates candidates, the second evaluates them, the third selects and verifies, the fourth classifies the resolution type.

### When Sequential Thinking Reveals Non-Convergence
If response setup step 5 cannot confirm the failure scenario is resolved - all candidate resolutions sidestep, introduce new failures, or rely on assertion - do not force a response. Proceed to the Non-Convergence Protocol.

## The Cycle

### Phase 1: Conclusion

State the decision as a falsifiable claim. The conclusion is the position being defended.

**Sufficiency requirements:**
1. **Specificity** - The conclusion must name the concrete mechanism, not just the intent.
2. **Falsifiability** - It must be possible to construct a scenario where the conclusion fails. If no failure scenario exists, the conclusion is either trivially true or too vague to be wrong.
3. **Scope boundary** - The conclusion must state what it covers and implicitly what it does not.

**Failure mode - Vague Conclusion:** The conclusion uses hedging language ("generally," "should usually," "where possible") that prevents meaningful counter-claims. Fix: restate with commitment.

### Phase 2: Counter-Claim

*After completing the sequential-thinking counter-claim setup (including early exit evaluation):*

Identify the weakest point in the conclusion and construct the strongest plausible attack against it.

**Sufficiency requirements:**
1. **Specificity** - The counter-claim must identify a concrete failure mode with a describable scenario.
2. **Strongest-attack principle** - The counter-claim must target the weakest point of the conclusion. The sequential-thinking setup enforces this by requiring enumeration and evaluation before selection.
3. **Non-strawman** - The counter-claim must attack the conclusion as stated, not a distorted version. Check: if the conclusion's author would respond "that's not what I said," the counter-claim is a strawman.
4. **Independence** - The counter-claim must not depend on rejecting the conclusion's premises. It must accept the conclusion's framing and find a failure within it.

**Failure modes:**
- **Strawman** - Attacks a distorted version. Symptom: the response is trivially easy, or a Refuted classification requires fewer than 3 sentences of mechanism.
- **Peripheral attack** - Targets a minor aspect while the core mechanism goes untested.
- **Premise rejection** - Challenges the problem, not the solution.

### Phase 3: Response

*After completing the sequential-thinking response setup:*

Address the counter-claim directly - either refute it or modify the conclusion to accommodate it.

**Sufficiency requirements:**
1. **Direct address** - The response must engage the counter-claim's specific failure scenario. Check: re-read the failure scenario after writing the response. Does it resolve that exact scenario?
2. **Mechanism, not assertion** - The response must explain *why* through a concrete mechanism, constraint, or design element.
3. **Type-specific rigor:**
   - If **Refuted**: Identify the specific mechanism, constraint, or precondition that prevents the failure scenario. If refutation requires fewer than 3 sentences of mechanism, re-evaluate the counter-claim for strawman characteristics.
   - If **Refuted with clarification**: Identify the implicit constraint that was unstated in the original conclusion. The clarification must not change the core mechanism or scope - only make explicit what was implicit.
   - If **Accepted with modification**: State the modification explicitly and separately from argumentative prose. The modification must be a concrete change to the conclusion's mechanism, scope, or constraints.
4. **No residual vulnerability** - After the response, the failure scenario must be either refuted or accommodated.

**Failure modes:**
- **Sidestep** - Responds to a different concern than the one raised.
- **Assertion-only** - Claims resolution without mechanism.
- **False resolution** - Appears to resolve but introduces a new failure mode of equal or greater severity.
- **Misclassification** - Resolution type does not match the response content.

## Evaluation Criteria

A cycle **passes** when all of the following are true:
1. The conclusion meets all three sufficiency requirements (specific, falsifiable, scoped).
2. The counter-claim meets all four sufficiency requirements (specific, strongest-attack, non-strawman, independent).
3. The response meets all four sufficiency requirements (direct, mechanistic, type-specific rigor, no residual vulnerability).
4. The resolution type classification is consistent with the response content.
5. The final position (original or modified) is strictly stronger than the input - same scope, fewer unaddressed failure modes.

A cycle **exits early** when counter-claim setup step 3 determines the conclusion is unfalsifiable within scope. All candidates must be documented with rejection reasons. `validation_status: early-exit`. No `final_claim` modification - original conclusion holds.

A cycle **fails** when any of the following are true:
1. Any phase does not meet its sufficiency requirements.
2. The response introduces a failure mode of equal or greater severity.
3. The resolution type is inconsistent with the response content.
4. The final position is not strictly stronger than the input.

A failed cycle should be re-attempted with the specific failure mode identified and corrected. If a cycle fails twice on the same phase, escalate to the user with the specific insufficiency documented.

## Non-Convergence Protocol

If the response cannot resolve the counter-claim (the failure scenario remains plausible and no modification eliminates it), the cycle has not converged. This is a signal, not a failure of process.

1. **Document the residual vulnerability** explicitly, including the specific failure scenario and why it resists resolution.
2. **Assess severity** - Is the failure scenario likely (occurs under normal operation) or edge-case (requires unusual conditions)?
3. **Escalate to user** - Present the residual vulnerability with severity assessment. The user decides whether to accept the risk, redesign the approach, or investigate further.

Do not silently accept a non-converging cycle. A decision with a documented, accepted residual vulnerability is stronger than a decision with an undiscovered one.

## Aggregated Mode: Additional Requirements

When testing decisions as a composed system:
1. **Interaction matrix** - Before generating counter-claims, enumerate the pairwise interactions between decisions that could produce emergent effects.
2. **Cascade analysis** - Counter-claims must describe the cascade path: which decision fails first, what failure output it produces, and how that becomes a failure input to the second decision.
3. **Shared-dependency detection** - Identify assumptions or mechanisms that multiple decisions depend on. Shared single points of failure are higher-priority targets.

## Output Format

Each completed cycle produces a summary block:

```md
### {Decision Label}

**Conclusion:** {One-sentence statement of the decision}
**Counter-claim:** {One-sentence description of the failure scenario}
**Response:** {One-sentence resolution or modification}
**Resolution type:** Refuted | Refuted with clarification | Accepted with modification
**Position modification:** {Explicit change, or "None - original conclusion holds"}
**Residual vulnerability:** {None, or description with severity}
**Cycle result:** Pass | Fail:{reason} | Non-convergent:{escalation}
```

For early exit cycles:

```md
### {Decision Label}

**Conclusion:** {One-sentence statement of the decision}
**Attack surface assessment:** {Candidates evaluated with rejection reasons}
**Cycle result:** Early exit - unfalsifiable within scope
```

For aggregated mode, prefix with an interaction matrix and append a revised position table showing all decisions post-cycle. The position table must include a Resolution type column.

## Edge Cases

- **Unfalsifiable conclusion:** Counter-claim setup step 3 finds all attack candidates fail the filters → `validation_status: early-exit`. Document evaluated candidates with rejection reasons. Original conclusion holds unchanged - no modification. In Validate-and-Forecast composition, an early-exit conclusion proceeds to Downstream Forecasting with the original claim.
- **Non-convergence:** Response cannot resolve the counter-claim → proceed to Non-Convergence Protocol. Do not force a resolution.
- **Failed cycle after two attempts on the same phase:** Escalate to the user with the specific insufficiency documented. Do not attempt a third cycle on the same failure.
- **Aggregated mode with no pairwise interactions:** If the interaction matrix reveals no interactions, state this explicitly. Counter-claims shift to individual decision robustness within the system context rather than interaction targeting.
- **Trivially true conclusion (no meaningful failure scenario):** Symptom: all attack candidates are premise rejections or strawmen. Apply early-exit. A trivially true conclusion does not benefit from adversarial stress-testing.

## Fallback Tier

When sequential-thinking is unavailable, the protocol operates in Fallback tier:
- All output format and sufficiency requirements remain identical
- Sequential-thinking enforcement at counter-claim setup and response setup is omitted - rely on careful deliberation in-line
- `validation_tier: Fallback` must appear in all output fields
- Callers using Fallback-tier output in composition pipelines should note the tier visibly in the composed output - downstream consumers need to know the analysis quality level
