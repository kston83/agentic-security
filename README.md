# Agent Control Plane

A reference implementation of security controls for AI agents in federal-contractor environments,
built on emulated AWS ([Floci](https://github.com/kurtmc/floci)) and mapped to NIST SP 800-53.

It rebuilds the agent-access pattern Roblox published (human-attributed agent identity, a credential
broker, and an MCP gateway that injects credentials so agents never hold them) and adds the two
pieces that pattern leaves open:

- **Agent Gate**: judges each tool call an agent makes with access it legitimately has
  (deterministic rules first, then TypeSafe Jev judgments; code owns policy; fails closed).
- **MCP Vet**: screens MCP servers and tools before they are admitted, and detects changes after approval.

Everything runs locally. Data is synthetic. The goal is three things together: working code, threat models,
and control documentation a federal assessor would recognize.

> **Status:** planning. See [`docs/PROJECTS.md`](docs/PROJECTS.md).

## Docs
- [`docs/PROJECTS.md`](docs/PROJECTS.md): project list, order, and acceptance checks
- [`docs/DECISIONS.md`](docs/DECISIONS.md): decision log
- [`docs/references/`](docs/references/): prior art and notes
