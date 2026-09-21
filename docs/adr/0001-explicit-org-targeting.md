# ADR-0001: Explicit org targeting and scoped operations

- Status: Accepted policy, recorded retrospectively
- Recorded: 2026-09-22
- Acceptance evidence: committed engineering guidance, linked below

## Context

This simulation is maintained alongside other Salesforce projects.
An available authenticated org is not necessarily this project's target.
Repository contents and a clean Git tree do not prove current org state.

## Decision

Record the existing AGENTS.md policy:

- Use an explicit --target-org for commands that support it.
- Verify that the alias identifies the intended org.
- Do not substitute another project's org when the intended org is unavailable.
- Scope retrieval to the required metadata, preserve local work and review diffs.
- Follow AGENTS.md authorization requirements for deployment and data changes.
- Treat Git publication separately from Salesforce deployment authorization.

## Alternatives and trade-offs

Implicit defaults are shorter but make the target less visible.
Automatic full retrieval can collect a broad snapshot but may introduce
unrelated changes and overwrite local work.

These comparisons explain the documented policy; they do not claim that a
historical options review or architecture meeting occurred.

## Consequences

Commands require explicit target and scope checks. Work that depends on an
unverified org must wait for its identity and access to be established.
Local documentation work can proceed within the authorized scope.

## Verification and limitations

The policy exists in the repository. This ADR does not demonstrate enforcement,
org connectivity, deployment success or completed implementation.
Review actual commands and results when org work occurs.

## References

- [Current engineering guidance](../../AGENTS.md)
- [Project context](../../CLAUDE.md)
- [Committed policy](https://github.com/hossainconsulting/agentforce-meridian-care/blob/9a532b6/AGENTS.md)
