# Agent Control Plane

Security controls for AI agents in federal-contractor environments, on emulated AWS (Floci), mapped to
NIST SP 800-53. It rebuilds Roblox's broker and gateway pattern and adds the Agent Gate and MCP Vet.
It is a reference implementation and demo, not an authorized system.

## Read before working
1. `docs/PROJECTS.md`: project list, order, acceptance checks. The first unchecked project in **Sequence** is current.
2. `docs/DECISIONS.md`: decision log. Rows marked **Proposed** need the owner's confirmation.
3. `docs/references/README.md`: notes on the Roblox paper and talk (prior art).

## Workflow
- Work one project at a time. Start in plan mode, propose the plan, wait for approval.
- Done means every acceptance check passes, with evidence shown.
- Each project updates `docs/THREAT_MODEL.md` and the control matrix.
- Commit at the end of each project and tick its boxes. Log deviations in `docs/DECISIONS.md`.
- Verify tool, SDK, and emulator behavior (Floci, MCP SDK, TypeSafe) against live docs or a cheap test.
  Say which you did. Never assume Floci enforces a control without testing it.

## Non-negotiables
- **Synthetic data only.** No real credentials, CUI, PII, or customer data, anywhere.
- **No secrets in git, logs, or agent processes.** Secrets live in the emulated Secrets Manager or in local
  `.env` files that are gitignored.
- **Fail closed.** If the Gate, judge, broker, or registry is unavailable, the call is denied.
- **Code owns policy.** Judges (Jev) return scores; thresholds live in one config file a human reviews.
- **Questions to Jev are drafted by Claude and edited by the owner.** Keep them in one file.
- **Pin the Jev model** and log the model ID returned. SDK log level is `warn` or `error`, never `info`/`debug`.
- **Never claim compliance the code doesn't show.** Control matrix rows link to evidence; gaps go in the POA&M.
- **Claude drafts control statements and threat-model entries; the owner reviews them.**
