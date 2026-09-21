# Engineering guidance — agentforce-meridian-care

Agentforce simulation for fictional Meridian Appliance Care.

## Project context

Audit baseline recorded 2026-09-22; reverify before relying on it.

- Read the existing CLAUDE.md for detailed scenario and design constraints.
- Intended alias devorg is documented but was not authenticated in the audited
  WSL environment. Verify its identity before any org operation.
- Hemayet builds Setup configuration manually unless he explicitly requests
  implementation through the Metadata API.
- Preserve the shared-org demo records identified in CLAUDE.md.
- Keep commercial Warranty_Entitlement__c separate from standard service
  entitlements and milestones.
- Coverage answers must use action results; preserve human escalation and
  define measurable deflection before launch.
- Scenario metrics are fictional inputs, not measured delivery outcomes.
- No tracked files under force-app/main/default were found during the audit.

## Working approach

- Read README.md, existing CLAUDE.md, relevant deliverables and any applicable
  nested instructions before editing.
- Inspect Git status and preserve unrelated or untracked work.
- Distinguish documented scenario, repository evidence and verified org state.
- State unknowns explicitly. Never invent completed tests, credentials,
  implementation outcomes or architecture decisions.

## Repository and metadata

- Read packageDirectories and sourceApiVersion from sfdx-project.json.
- Preserve the current package layout and API version unless a change is needed
  and its compatibility has been checked.
- A clean Git tree does not establish synchronization with an org.
- Before retrieval, identify the target org and exact metadata scope, preserve
  local changes and inspect the resulting diff.
- Do not retrieve all metadata automatically before every edit.
- Verify commands and flags with the installed sf CLI help before unfamiliar
  operations. Never suppress errors to make a check appear successful.
- Keep .sf/, .sfdx/, authentication files, tokens and private data out of Git.

## Org operations and authorization

- Use an explicit --target-org for commands that support it.
- Verify the alias maps to the intended org; never substitute sunrise for an
  unavailable project-specific org.
- Read-only inspection and local documentation work may proceed within scope.
- Before deployment, data mutation, destructive scripts or Flow activation,
  present the exact target, scope, validation and recovery approach to Hemayet.
  Obtain authorization unless the current request already explicitly covers
  that operation and target.
- Git commits and pushes do not constitute Salesforce deployment approval.
- Do not expose access tokens or authentication URLs in logs or documentation.

## Implementation and review

- Keep changes small and traceable to a requirement or documented issue.
- For Apex, review bulk behavior, governor limits, sharing, object and field
  access, input validation and failure handling.
- For automation, review entry conditions, repeated execution, fault paths,
  permissions and effects on existing records.
- Use synthetic test data and meaningful assertions for new behavior.
- Never execute seed or purge scripts merely to inspect them.

## Verification and evidence

- For documentation-only changes, review content, links and git diff --check.
- For metadata or code changes, select validation appropriate to the affected
  behavior and confirm current Salesforce requirements for the target.
- Record the test target, command or procedure, result and limitations.
- Treat coverage as one check, not proof of correctness.
- Do not claim Apex tests ran locally when they ran in a Salesforce org.
- Preserve existing evidence; label simulations and proposed work clearly.

## Git and decisions

- Review staged diffs and stage only files belonging to the task.
- Prefer a focused branch and pull request for implementation changes.
- Never force-push, reset or clean away unrelated work.
- Record significant decisions with context, alternatives and consequences.
  Mark proposals as proposed; do not invent accepted decisions or approvers.
