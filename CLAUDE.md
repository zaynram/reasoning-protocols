# reasoning-protocols

Five structured reasoning protocols: adversarial stress-testing, impact forecasting, tradeoff analysis, assumption surfacing, and plan review.

## Features

| Component                      | Description                                                                             |
| ------------------------------ | --------------------------------------------------------------------------------------- |
| `reasoning-chain` skill        | Protocol selection and composition - invokes one or more protocols for a given decision |
| `adversarial-cycle` skill      | Claim stress-testing via conclusion → counter-claim → response cycles                   |
| `downstream-forecasting` skill | Cascade impact tracing with severity-gated termination                                  |
| `tradeoff-analysis` skill      | Scored alternative comparison with sensitivity analysis                                 |
| `assumption-surfacing` skill   | Foundation audit with fragility rating and epistemic status tagging                     |
| `plan-review` skill            | Adversarial plan critique with assumption surfacing and risk identification             |
| `sequential-thinking` MCP      | Required for Full-tier protocol execution; enforces phase discipline                    |
