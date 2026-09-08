# Runbook — Claude Code and Salesforce ("Claudeforce" readiness)

**Date:** 08/09/2026 · **Applies to:** every Salesforce engagement repo under
`hossainconsulting/` · **Owner:** Hemayet

This is the workspace's answer to the wave of "prepare for Claudeforce in 30 minutes"
guides that followed the 26 August 2026 Salesforce and Anthropic announcement. It records
what was set up, why it was set up that way, and how to reproduce it on a new machine.
The configuration itself is committed in each repo; this document is the reasoning.

---

## 1. What "Claudeforce" actually is

The announcement bundles several distinct things. Only some of them matter for a
consultant working a Developer Edition org from a terminal.

| Layer | What it is | Who it's for | Used here? |
|---|---|---|---|
| **Salesforce in Claude** | A connector in claude.ai and Claude Desktop that reaches the org through Salesforce **hosted MCP servers** (OAuth via an External Client App). Ships with 37 prebuilt sales skills. | Sellers, service reps, anyone in the Claude app | Optional, section 5 |
| **Claude in Agentforce** | Claude models available as the reasoning engine inside Agentforce and Prompt Builder. | Agent builders | Org-side, nothing to install |
| **`salesforce-development` plugin** | Salesforce's official Claude Code plugin: 38 skills, a production deploy gate, Apex and SOQL language servers, and its own hosted "expert" MCP servers. Detects a DX project by `sfdx-project.json`. | Admins, developers, consultants in Claude Code | **Yes** |
| **Salesforce DX MCP server** (`@salesforce/mcp`) | A local MCP server that runs on the Salesforce CLI's own org authorisations. SOQL, deploy, retrieve, Apex and agent tests. No OAuth app needed. | Anyone with the CLI | **Yes** |

The two rows marked **Yes** are what the repos are configured for. They cost nothing,
need no Setup changes in the org, and work in Developer Edition.

## 2. What was configured, repo by repo

Seven repos received the same three files. Three of them also received the
`sfdx-project.json` they were missing, because the plugin uses that file to recognise a
Salesforce project.

| Repo | Org alias | `sfdx-project.json` |
|---|---|---|
| `agentforce-meridian-care` | `devorg` | already present |
| `meridian-field-services` | `meridian` | already present |
| `sunrise-solar-internship` | `sunrise` | already present |
| `tradelink-group` | `tradelink` | already present |
| `coastline-retail-group` | `coastline` | **added** |
| `ironbark-industrial-supply` | `ironbark` | **added** |
| `kurrajong-energy` | `kurrajong` | **added** |

`home-services-ai` and `portfolio` are not Salesforce projects and were left alone.

### `.mcp.json` — the DX MCP server, pinned to one org

```json
{
  "mcpServers": {
    "salesforce-dx": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@salesforce/mcp",
               "--orgs", "devorg",
               "--toolsets", "data,metadata,testing",
               "--no-telemetry"]
    }
  }
}
```

Three decisions in that file:

- **`--orgs` is the repo's alias, not `DEFAULT_TARGET_ORG`.** The server resolves
  `DEFAULT_TARGET_ORG` on every call, so it follows whatever `sf config set
  target-org` last touched. With seven orgs on one laptop, that is how a verification
  query for Kurrajong ends up running against SunRise. Pinning the alias makes the
  wrong-org failure loud (an auth error) instead of silent (a plausible number).
- **Three toolsets, not `all`.** The server carries more than 60 tools; the docs
  themselves warn that enabling everything swamps the context window. `data` gives
  `run_soql_query`. `metadata` gives `deploy_metadata` and `retrieve_metadata`.
  `testing` gives `run_apex_test` and `run_agent_test`, and the second one is why the
  Meridian care repo cares. The LWC, Aura, mobile and DevOps Center toolsets have no
  place in an admin-track engagement.
- **`--no-telemetry`** because these are client-simulation orgs and there is no reason
  to send usage data anywhere.

### `.claude/settings.json` — what the repo turns on for everyone

```json
{
  "enabledMcpjsonServers": ["salesforce-dx"],
  "enabledPlugins": {
    "salesforce-development@claude-plugins-official": true
  },
  "permissions": {
    "allow": ["Bash(sf org display:*)", "Bash(sf org list:*)", "Bash(sf data query:*)"]
  }
}
```

- `enabledMcpjsonServers` pre-approves the project server so a fresh clone does not
  prompt. Claude Code only honours this once the folder has been trusted, which is the
  right order.
- `enabledPlugins` declares the official plugin at project scope. It does not install
  it; a machine that lacks it is told which command to run. This is also what makes the
  plugin load in cloud sessions.
- The permission allowlist covers the three read-only CLI commands that get run
  constantly. Nothing that writes is pre-approved.

### `CLAUDE.md` — the rules the tooling does not change

Every repo now has one. The Meridian care file already existed and gained a section;
the other six got a short file with the same skeleton: what the simulation is, the org
alias, the division of labour, where things go, and the tooling rules. The rules matter
more than the tooling:

1. Hemayet builds Setup configuration by hand. The plugin can generate objects, fields,
   flows and permission sets, and it must not, unless asked. The certifications test
   Setup navigation and so does the job.
2. Read before write. Queries and retrieves are routine. Deploys, deletes and anything
   that changes the org need an explicit ask in the conversation, every time.
3. Seed and fix-up scripts are anonymous Apex, kept in the repo, and logged in the
   build log like any other change.

## 3. Setting up a machine (the actual 30 minutes)

Do this once per laptop. Steps 1 and 2 are the only ones with any waiting in them.

**1. Prerequisites.** Node.js LTS, Python 3.8 or later, Salesforce CLI, and Claude Code
2.1.222 or later.

```bash
npm install -g @salesforce/cli
sf --version
claude --version
```

**2. Authorise each org under the alias its repo expects.** The DX MCP server reads the
CLI's auth store, so this is the only login there is. One browser round-trip per org.

```bash
sf org login web --alias devorg
sf org login web --alias meridian
sf org login web --alias sunrise
sf org login web --alias tradelink
sf org login web --alias coastline
sf org login web --alias ironbark
sf org login web --alias kurrajong
sf org list
```

**3. Install the plugin** once, at user scope, from inside any Claude Code session:

```
/plugin install salesforce-development@claude-plugins-official
```

If that reports the marketplace is missing, add it with
`/plugin marketplace add anthropics/claude-plugins-official` and retry. Salesforce's
own marketplace (`/plugin marketplace add forcedotcom/sf-skills`) carries the same
plugin plus specialised ones such as `service-engagement`, `dx-devops` and
`platform-trust-security`. None of those are enabled by default here.

**4. Open a repo and trust it.** `cd` into the repo, run `claude`, accept the workspace
trust dialog. The `.mcp.json` server and the plugin load from the committed settings.

**5. Validate.**

```
/salesforce-development:setup
/mcp
```

The first checks prerequisites and org detection. The second should list
`salesforce-dx` as connected.

**6. First prompt.** Something that can only be answered from the org:

> How many Cases are in the org, grouped by Status? Use the Salesforce DX server.

The answer should come back via `run_soql_query` against the pinned alias. If it comes
back from a `sf data query` shell call instead, that is fine too; if it comes back
without any tool call, something is not connected.

## 4. Verification checklist

Run after setup on a new machine and after any change to the three files.

- [ ] `sf org list` shows all seven aliases as connected.
- [ ] `/mcp` inside each repo lists `salesforce-dx` as connected.
- [ ] A SOQL prompt in `kurrajong-energy` returns Kurrajong data, not another org's.
  This is the pinning test and the one that matters.
- [ ] `/salesforce-development:setup` passes in a repo that has `sfdx-project.json`.
- [ ] Asking Claude to "create a custom object" in any repo produces a refusal that
  cites `CLAUDE.md`, not a deployed object.

## 5. Optional: Salesforce in Claude (hosted MCP, claude.ai)

Not configured, and not needed for the engagements. Recorded here because it is the
part of Claudeforce the guides spend most time on, and because the 30-minute figure in
their titles is the wait for a new External Client App to become usable.

Salesforce side, in Setup: **MCP Servers → Salesforce Servers → Activate** the server,
then **External Client App Manager → New External Client App**, enable OAuth, set the
callback URL (`https://claude.ai/api/mcp/auth_callback` for claude.ai and Claude
Desktop; a localhost callback for Claude Code), grant the API and refresh-token scopes,
require PKCE, and copy the consumer key. Sources differ on whether a dedicated
`mcp_api` scope is required; check the scope list in the org rather than trusting a blog.

Claude side: **Customize → Connectors → Add custom connector**, paste the server URL and
consumer key, authorise. For Claude Code the equivalent is a remote HTTP server:

```bash
claude mcp add --transport http salesforce-hosted <server-url> \
  --client-id <consumer-key> --client-secret --callback-port 38000
```

Hosted servers expose sObjects, Flows and invocable actions and are GA on Enterprise
Edition and above; they also work in Developer Edition. They authenticate as a
Salesforce user through OAuth, so the run-as user's profile and permission sets are the
whole security model. That is the sentence to remember if Daniel ever asks.

## 6. Framing, for the debrief

Two of the frameworks that circulate alongside these guides fit this workspace
uncomfortably well. BCG's 10-20-70 rule puts ten percent of an AI programme's effort in
algorithms, twenty in technology and data, and seventy in people and process. This
runbook is the twenty. The division-of-labour rule in every `CLAUDE.md`, the build log
discipline, and the "deflection must be countable before go-live" rule in the Meridian
engagement are the seventy. Deploy-Reshape-Invent is the other one worth naming:
wiring Claude to the org is Deploy, the Week 3 agent that reads
`Warranty_Entitlement__c` instead of guessing is Reshape, and nothing here is Invent,
which is the honest answer when someone asks.

## Sources

- Salesforce DX MCP Server, [github.com/salesforcecli/mcp](https://github.com/salesforcecli/mcp)
  (flags, toolsets, `--orgs` semantics; package `@salesforce/mcp` 0.30.15 at time of writing)
- Salesforce Skills Library and the `salesforce-development` plugin,
  [github.com/forcedotcom/sf-skills](https://github.com/forcedotcom/sf-skills)
- Claude Code docs: [MCP](https://code.claude.com/docs/en/mcp),
  [plugin marketplaces](https://code.claude.com/docs/en/discover-plugins)
- Salesforce Developers blog, *Connect Claude with Salesforce Hosted MCP Servers* (May 2026)
  and *Headless Development with Skills and a Claude Code Plugin* (August 2026)
- Salesforce press release, *Salesforce and Anthropic Announce Claudeforce* (26/08/2026)
- The LinkedIn guide that prompted this, Kudryk, *Full Guide to Claude in Salesforce
  (Prepare for ClaudeForce, 30 minutes)*. Not reachable from the build environment; the
  steps above were reconstructed from the primary sources it draws on.
