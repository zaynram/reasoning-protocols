# Plan Review - Protocol Procedure

## Setup

Read the full plan artifact before beginning. Do not start the review from
memory or summary.

## Phase 1: Assumption Inventory *(Sequential-thinking required - minimum 3 thoughts)*

Before critiquing, enumerate the plan's load-bearing assumptions - things
the plan takes as given that could be wrong. For each:
- State the assumption explicitly
- Rate confidence it is correct (High / Medium / Low)
- Identify what breaks downstream if it is wrong

Flag any Low-confidence assumptions for priority investigation in Phase 2.

## Phase 2: Adversarial Review *(Sequential-thinking required - minimum 4 thoughts)*

For each of the following categories, identify the strongest specific finding
(if any). A category with no confident finding is noted "None identified" -
do not manufacture findings to fill categories.

**Invalid assumptions** - Which of the Phase 1 assumptions is most likely
wrong? What is the concrete failure scenario?

**Missing edge cases** - What input, state, or timing condition does the plan
not address? What happens when it occurs?

**Underspecified integration** - Which integration point (API, service,
dependency) is assumed to behave in a way that is not guaranteed? What happens
when it doesn't?

**Over-engineering** - What complexity exists in the plan that does not earn
its weight given the stated goals?

**Security gap** - What system boundary accepts external input without
validation or trust-level enforcement?

**Goal-approach inconsistency** - Where does the proposed approach fail to
deliver the stated goal, or deliver something adjacent to it?

## Phase 3: Severity Assignment

For each finding from Phase 2:
- **Critical** - Would cause implementation failure or a production incident.
  Must be resolved before proceeding.
- **Important** - Meaningfully weakens the approach. Should be addressed;
  acceptable to defer with documented plan.
- **Minor** - Low-risk. Addressable opportunistically.

Findings with no confident severity assignment are dropped.

## Phase 4: Integration Notes

After presenting findings and receiving user direction, append a "## Review
Notes" section to the plan file:

```
## Review Notes

**Reviewed at:** {timestamp}
**Reviewer:** plan-review agent

### Integrated
- {finding}: {change made and rationale}

### Deferred
- {finding}: {when it will be addressed}

### Dismissed
- {finding}: {rationale for dismissal}
```
