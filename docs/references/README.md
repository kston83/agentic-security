# References

## Roblox: How Roblox Gives AI Agents Access Without Giving Them Credentials
Judge, Datta, Alomari, Grinstead (Roblox). Local copy: `roblox-agent-access-without-credentials.pdf`.
Source: https://about.roblox.com/publications/how-roblox-gives-ai-agents-access-without-giving-them-credentials

**Design**
- **Agent identity (§2.3):** a JWT from an OIDC token exchange that names the human the agent works for.
  Delivered by sandbox bootstrap, or exchanged from a developer's GitHub session in Coder workspaces.
  Internal calls get a per-request token with about a 15-minute lifetime.
- **Credential Broker (§2.4):** stores vendor OAuth tokens in Vault, keyed by (human, service).
  The OAuth seed uses PKCE in a browser, once per pair. After that, agents exchange their identity
  for a short-lived token with no human interaction. Tokens are cached briefly; refresh tokens are rotated.
- **MCP Gateway (§2.5):** acts as an MCP server to the agent and an MCP client to the upstream server.
  It accepts only stateless Streamable-HTTP backends. It also handles:
  - credential injection through an endpoint only the gateway can call
  - task "profiles" as a least-privilege control
  - soft and hard response-size caps
  - content policy: blocks confidential material and redacts PII and privileged text
  - acting as the single egress and revocation point
  - per-request audit of human, agent, tool, and upstream
- **Brokering vs. injection (§2.6):** injection is the default. Brokering is used only when the agent must
  speak the vendor protocol itself, and only with short-lived, scoped tokens behind an egress allowlist.

**Threat model (§2.1):** prompt injection; unvetted MCP servers; compromised agents; laundering actions
through shared identity; reaching further than the task requires.

**Gaps this project targets**
- "[Injection] does not stop a compromised agent from using a tool it legitimately has." → **Agent Gate**
- "Every MCP server and tool passes security checks before the gateway exposes it." The checks are not
  described. → **MCP Vet**

**Their future work (our stretch items):** extend to non-MCP HTTP traffic, move to SPIFFE/SPIRE
workload identity, and deploy the broker alongside each sandbox.

## Roblox talk: "Prompt to Prod" (Andrew Swerdlow, QCon AI)
Notes only; see InfoQ for the recording.
- Security was the largest workstream. It covered sandboxes, policy gateways, just-in-time least privilege,
  no long-lived secrets, and agent identities that are auditable separately from humans.
- Incident example: an agent asked teammates over Slack, as the engineer, to merge its PR and skip checks.
  → Agent Gate demo scenario (impersonation and social engineering).
- An LLM assesses whether a PR's blast radius is low enough to auto-merge. → stretch: change-risk gate.
- Evals take 30–40% of the effort. OpenTelemetry tracing for agents is recommended. → P9, P10.
- Removed mandatory human code review and deployment freezes for some workflows. → write-up: what
  transfers to federal systems under CM-3, CM-4, and AC-5, and what doesn't.
