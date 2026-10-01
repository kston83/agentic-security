# Decision log

| Date | Decision | Reason | Status |
| - | - | - | - |
| 2026-10-01 | Separate project from CUI Navigator | Different product; CUI Navigator stays as-is | Decided |
| 2026-10-01 | Emulate AWS with Floci, not LocalStack | Free and open source; LocalStack community edition sunset in March 2026 | Decided |
| 2026-10-01 | Build on Roblox's published broker + gateway design; add Agent Gate and MCP Vet | Credible prior art; their paper names the two gaps | Decided |
| 2026-10-01 | Synthetic data only | Demo runs outside any authorized boundary; Jev provider FedRAMP status unverified | Decided |
| 2026-10-01 | TypeScript for all services | Matches `@typesafe-ai/sdk` and the MCP TypeScript SDK | Proposed |
| 2026-10-01 | OpenTofu for IaC | Open license; Terraform-compatible; works with Floci | Proposed |
| 2026-10-01 | Scripted agent for evals; Claude Code as the agent in demos | Repeatable evals and a realistic demo | Proposed |
| 2026-10-01 | Apache-2.0 license | Common, permissive, federal-friendly | Proposed |
| 2026-10-01 | Project name `agent-control-plane` | Working name; easy to change before first push | Proposed |
