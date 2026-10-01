# Projects

Work one project at a time, in the order under **Sequence**. A project is done only when its acceptance
checks pass, with evidence. Each project also adds:
- its section of `docs/THREAT_MODEL.md` (STRIDE plus MITRE ATLAS and OWASP LLM/agentic references)
- its rows in the control matrix (P11)
- tests

800-53 controls are listed by intent. Final mappings are reviewed in P11.

## Sequence
1. **Slice 1, publishable on its own:** P0 → thin P1 → P4 (static injected secret) → P6 → P9 (basic log)
   → P10 (small set) → P11 (matrix for these) → P12 (write-up 1 and video).
2. **Roblox rebuild:** P2 → P3 → P5 → P7.
3. **Supply chain:** P8 → P12 (write-up 3).
4. **Stretch:** S1–S3.

---

## Foundation

### P0 — Repo and supply-chain baseline
- [ ] CI: lint, typecheck, test, IaC scan (Checkov and Conftest), on every push and PR
- [ ] CodeQL, secret scanning, Dependabot; dependencies pinned with lockfiles
- [ ] CycloneDX SBOM generated per release; releases signed with build provenance
- [ ] Apache-2.0 license; `SECURITY.md`; `docs/THREAT_MODEL.md` skeleton

**Acceptance:** CI green on main; a tagged release has an SBOM and a verifiable provenance attestation.
**800-53:** SA-11, SA-15, SR-3, SR-4, CM-2, CM-14

### P1 — The lab ("range")
- [ ] **Floci capability report first:** which AWS services we need work, and whether IAM policy
      evaluation, STS (`AssumeRoleWithWebIdentity`, session tags), KMS, Secrets Manager, CloudWatch Logs,
      and S3 Object Lock are *enforced* or merely accepted. Record the results in DECISIONS.md.
- [ ] Docker Compose: Floci, OpenTelemetry collector, mock services
- [ ] OpenTofu baseline: KMS keys, Secrets Manager, log groups, log archive bucket (Object Lock), audit tables
- [ ] Synthetic mock MCP servers: Slack, Jira, wiki, git, document store. Seed data includes poisoned
      documents (indirect injection) and fake CUI-marked files.
- [ ] Agent harness: a scripted agent (repeatable) plus a Claude Code configuration that uses the gateway

**Acceptance:** `docker compose up` plus `tofu apply` bring the lab up from clean in under 5 minutes;
the capability report is committed; Checkov passes or has documented exceptions.
**800-53:** CM-2, CM-6, CM-7, SC-12, SC-13, SC-28

---

## Rebuilding Roblox's design

### P2 — Agent identity (paper §2.3)
- [ ] Local OIDC issuer mints short-lived agent JWTs naming the human principal and the agent
- [ ] Every component validates the JWT against the issuer's key set (JWKS); expired or unsigned tokens are rejected
- [ ] If Floci supports it: STS exchange for AWS credentials tagged with the human principal

**Acceptance:** tests reject expired, wrongly signed, and wrong-audience tokens; every audit event carries
both the human and the agent identity.
**800-53:** IA-2, IA-4, IA-5, IA-9, AC-2, AU-3, AU-10

### P3 — Credential broker (§2.4)
- [ ] One-time OAuth seed (PKCE) against a mock vendor OAuth server, per (human, service)
- [ ] Tokens stored in Secrets Manager, encrypted with KMS
- [ ] Token exchange (RFC 8693 style): agent JWT plus service name returns a short-lived scoped token,
      or a connection URL if no seed exists
- [ ] Short cache; refresh with rotation; revoke per human, per service, or both
- [ ] A gateway-only endpoint for injection

**Acceptance:** no agent process environment or log contains a vendor credential (automated check);
revoking a human's tokens cuts off all of that human's agents within one cache TTL.
**800-53:** IA-5, SC-12, SC-28, AC-2(3), AC-6

### P4 — MCP gateway (§2.5–2.6)
- [ ] Streamable-HTTP MCP proxy: a server to the agent, a client to the upstream; one logical server per backend
- [ ] Credential injection by default; brokering supported, with the residual risk documented
- [ ] Profiles (named tool sets) enforced as least privilege
- [ ] Soft and hard response-size caps with actionable errors
- [ ] Pipeline hooks for Gate (P6), inspection (P7), registry (P8), and audit (P9)
- Slice 1 uses a static secret held by the gateway. P3 replaces it.

**Acceptance:** an agent can't call a tool outside its profile; injected credentials never appear in agent
traffic (packet or log check); caps trip in tests.
**800-53:** AC-3, AC-4, AC-6, SC-7, SC-8, SI-10

### P5 — Sandbox and egress control (talk: sandboxes)
- [ ] The agent container's only network routes are to the gateway and an egress proxy with an allowlist
- [ ] IaC shows the AWS equivalent: VPC, security groups, VPC endpoints, Network Firewall rules
      (scanned, even where Floci can't enforce them)

**Acceptance:** from inside the sandbox, a direct call to a non-allowlisted host fails (test); brokered
traffic is limited to allowlisted hosts.
**800-53:** SC-7, SC-7(5), AC-4, CM-7

---

## Additions (the gaps the paper names)

### P6 — Agent Gate (headline)
- [ ] Policy pipeline per tool call: deterministic rules (allow/deny lists, destinations, data patterns),
      then Jev judgments, then code applies thresholds to reach **allow / deny / escalate**
- [ ] Initial judgments (question wording reviewed by the owner):
  - Was the action prompted by instructions in content the agent read, rather than by the user's task?
  - Does it move data to a destination outside the approved boundary?
  - Is the agent presenting itself as a human, or asking a human to bypass a control? (from the talk's Slack incident)
  - Is the action outside the stated task's scope?
- [ ] Human escalation path (approve/deny, with the decision recorded)
- [ ] Fail closed on judge error or timeout; judge behind an interface (Jev first; a second judge is optional)
- [ ] Jev model pinned; no request content logged at info/debug level

**Acceptance:** P10 scenarios meet the targets set in P10; fail-closed behavior is tested; every decision is
auditable with its rule or judgment scores.
**800-53:** AC-3, AC-4, AC-5, SI-10, SI-4

### P7 — Response inspection (§2.5 content controls)
- [ ] Inspect tool results before they reach the agent: redact PII, block CUI-marked content
      (banner and portion-marking patterns), flag injection payloads in returned data
- [ ] Optional Jev judgment on whether a result contains instructions aimed at the agent

**Acceptance:** seeded CUI-marked and PII documents never reach the agent unredacted; injection-payload
detection rate is reported in P10.
**800-53:** AC-4, SI-4, SI-15, PT-2, MP-3

### P8 — MCP Vet and server registry (threat named in §2.1)
- [ ] CLI: connect to an MCP server, list its tools, and score:
  - poisoned or hidden instructions in tool descriptions
  - excessive capability
  - unpinned or unsigned packages
  - provenance
- [ ] Signed registry of approved servers and pinned description hashes; the gateway exposes only registry entries
- [ ] Rug-pull detection: alert or quarantine when an approved server's tool descriptions change
- [ ] Labeled corpus (benign and malicious) plus a research scan of public servers

**Acceptance:** catches every malicious fixture in the labeled corpus at the documented threshold; a
description change triggers quarantine in an end-to-end test.
**800-53:** SR-3, SR-5, SR-6, SR-11, CM-7, CM-8, CM-11, SA-9, RA-5

---

## Cross-cutting

### P9 — Monitoring, detection, kill switch
- [ ] One OpenTelemetry audit event per call: human, agent, profile, tool, upstream, gate decision and
      scores, inspection actions, latency
- [ ] Shipped to CloudWatch Logs and archived to an S3 bucket with Object Lock
- [ ] Detections: denial spikes, tool-sprawl per agent, repeated escalations, description-hash changes
- [ ] Kill switch: one command revokes a human's or an agent's access everywhere

**Acceptance:** the audit trail answers "who did this" for every scenario in P10; the kill switch stops a
running agent within the documented time.
**800-53:** AU-2, AU-3, AU-6, AU-9, AU-11, AU-12, SI-4, IR-4, IR-5

### P10 — Red-team and eval harness (talk: evals are 30–40% of the work)
- [ ] Scenarios: direct and indirect injection, poisoned docs, impersonation and social engineering,
      credential theft attempts, exfiltration, malicious MCP server, rug pull
- [ ] Each scenario records which layer stopped it: gateway, Gate, inspection, egress, or none
- [ ] Runs in CI with cached judge responses; Gate thresholds are tuned on a dev split and reported on a held-out split

**Acceptance:** report committed; known misses written up, not hidden.
**800-53:** CA-8, RA-3, SI-4, SA-11

### P11 — Compliance package
- [ ] Control matrix: control → implementation statement (SSP style) → code location → test/evidence → status
- [ ] OSCAL component definition per component
- [ ] POA&M for known gaps (for example, the Jev provider's FedRAMP status, and controls Floci can't enforce)
- [ ] Evidence gathered automatically in CI
- [ ] Checkov NIST 800-53 framework plus Conftest policies on all IaC

**Acceptance:** OSCAL validates against the NIST schema; every "implemented" row links to passing evidence.
**800-53:** CA-2, CA-5, CA-7, PL-2

### P12 — Publishing
- [ ] Three-minute demo video; scenario viewer and audit-log dashboard
- [ ] Write-ups:
  1. Agent control plane mapped to 800-53
  2. Red-team results
  3. MCP Vet public-server findings
  4. Prompt-to-prod through an 800-53 lens

**Acceptance:** each write-up cites the evidence it relies on.

---

## Stretch
- **S1** SPIFFE/SPIRE workload identity (the paper's future work)
- **S2** Brokering and injection for non-MCP HTTP traffic (the paper's future work)
- **S3** Change-risk gate for agent PRs, framed against CM-3, CM-4, and AC-5 (from the talk)
