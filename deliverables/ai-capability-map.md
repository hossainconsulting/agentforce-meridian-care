# The AI capability map — ServiceNow's twelve, answered in Salesforce

**Date:** 07/09/2026 · **Status:** design document. **Nothing below is verified against
`devorg` yet** — the Verify column says how to confirm each row, and the build log gets
the answer.

## Why this document exists

A widely-circulated diagram lists twelve AI capabilities ServiceNow puts in front of
enterprise buyers, and the argument around it is that the platform has stopped being a
ticketing system and become "an AI operating layer." The claim is fair. It is also the
claim Salesforce makes, and Meridian has Agentforce licensed and switched off, so the
comparison is not academic here — it is the Week 6 debrief question in advance.

This maps each of the twelve to what actually delivers it on this platform, what it is
configured with, and how to prove it exists in this org. It is deliberately written as
one document rather than a section repeated across the Sales Cloud, Service Cloud and
Data Cloud engagements: eleven of the twelve rows are platform capabilities, not
track-specific ones, and copying them four times creates four things to correct when a
product name changes.

**Read the caveat before the table.** Salesforce renames and repackages AI features
faster than any other part of the platform — Einstein Copilot became Agentforce inside a
year. Treat every product name below as *the thing to go and look for in Setup*, not as
a confirmed SKU. Licensing especially: several rows need consumption credits or a Data
Cloud entitlement that a Developer Edition org may not carry.

---

## The map

| # | ServiceNow capability | Salesforce equivalent | Configured in | Verify in `devorg` |
|---|---|---|---|---|
| 1 | **Now Assist** — in-workflow assistant | **Agentforce** (employee-facing), grounded prompt templates surfaced on record pages | Agent Builder; Prompt Builder | Setup → Einstein Setup / Agentforce Studio. Confirm it is *enabled*, not merely licensed |
| 2 | **AI Agents** — reason, plan, use tools | **Agentforce** agents: Topics scope the job, Actions do the work | Agent Builder → Topics → Actions (Flow, Apex, Prompt Template, API) | Build a throwaway agent with one Flow action and run it in the preview panel |
| 3 | **GenAI Search** — cited answers over knowledge | Retrieval grounding over **Knowledge** and unstructured data in **Data Cloud** (vector search + retrievers) | Data Cloud; retriever config referenced by an agent action | `SELECT Id FROM DataspaceScope` — if it errors, Data Cloud is not provisioned and this row is blocked |
| 4 | **Summarization** — cases, articles, conversations | Prompt templates of type Record Summary / Field Generation; Einstein work and case summaries for Service | Prompt Builder | Create a Record Summary template on Case, ground it in the fields, run it on a seeded case |
| 5 | **Virtual Agent** — deflect common, escalate complex | **Agentforce Service Agent** for the customer-facing channel; **Einstein Bots** is the older path | Agent Builder + channel (Messaging/Web); Omni-Channel for handoff | Confirm which is available before designing. Do not build both |
| 6 | **Predictive Intelligence** — patterns, risk, outcomes | Einstein Prediction Builder, Einstein Case Classification; Einstein Studio / Model Builder on Data Cloud for custom models | Setup → Einstein; Data Cloud | Check available Einstein features against this org's edition — several are not in Developer Edition |
| 7 | **Workflow Automation** — trigger, decide, act | **Flow**. Unchanged, and still the backbone | Flow Builder | Already in scope for Week 2. Nothing new to enable |
| 8 | **Knowledge Generation** — draft and maintain articles | Prompt-template-generated draft articles, grounded in resolved cases, **published only after human review** | Prompt Builder + Knowledge; approval before publish | Knowledge must be enabled first. Check Setup → Knowledge Settings |
| 9 | **AI Governance** — policy, audit, transparency | **Einstein Trust Layer** — masking, zero data retention, toxicity scoring, audit trail. Plus Shield (Event Monitoring, Field Audit Trail) where licensed | Not a build. It is in the request path | See the section below — this row is structurally different and matters most |
| 10 | **Analytics** — workflow data to insight | Reports and dashboards; CRM Analytics / Tableau where licensed | Report builder | Reports are enough for the deflection metric. Do not reach for CRM Analytics to draw four numbers |
| 11 | **Integrations** — apps, data, APIs | MuleSoft; External Services and Salesforce Connect; any REST endpoint exposed to an agent as an Action | Agent Builder actions; External Services | Out of scope for Meridian unless the retailer feeds land in Week 5 |
| 12 | **Human Oversight** — review, approve, edit, feedback | Approvals, Omni-Channel escalation with transcript, agent action design (read vs. write), Agentforce Testing Center | Agent Builder; Approval Processes; Omni-Channel | The escalation path is already a Week 3 non-negotiable. This row is design, not licensing |

---

## Where the mapping is not one-to-one — read this before quoting the table

**Rows 2 and 5 are one product, not two.** ServiceNow lists AI Agents and Virtual Agent
separately because they grew separately. On Salesforce both land on Agentforce; the
difference is who the agent is pointed at and which channel it is published to, not the
build. Treating them as two projects is the duplicate work this document exists to
prevent. Meridian needs *one* agent design with a customer-facing channel and a human
handoff — not a bot project and an agent project.

**Row 9 is not a row.** This is the substantive difference between the two platforms'
stories, and it is the one worth being able to explain. In the ServiceNow diagram, AI
Governance is one tile of twelve — a thing you also do. On Salesforce, the Einstein Trust
Layer sits in the path of every model call: masking on the way out, grounding, toxicity
scoring and audit on the way back. You do not switch it on beside the other eleven; it is
what the other eleven run through.

That has a practical consequence for Daniel's security review. The answer to *"how do you
stop the AI inventing coverage?"* is not "the Trust Layer" — the Trust Layer governs the
call, it does not know Meridian's business rules. The answer is the design decision
already made in the Week 2 brief: `Is_Currently_Covered__c` is a **formula field the agent
retrieves**, not a judgement it generates. Governance is the platform's job; correctness
is the data model's. Conflating them is how people end up trusting a masking layer to
enforce a warranty.

**Row 6 is where Developer Edition will bite.** Several Einstein predictive features are
edition- and licence-gated in ways that do not show up until you try to enable them.
Check before scoping any of Week 5 around them.

**Row 3 is gated on Data Cloud.** Cited answers over unstructured knowledge need
retrieval, and retrieval needs Data Cloud provisioned. If `DataspaceScope` does not
resolve in this org, row 3 is not "hard", it is *unavailable*, and the honest move is to
scope Meridian's agent to grounding in Knowledge and CRM records instead of promising
enterprise-wide cited search. Say that in the debrief rather than discovering it live.

---

## What this changes for Meridian, and what it does not

**Does not change:** the Week 2 build. The data model, record types, queues, entitlement
processes and the coverage formula all stand. Nothing here is a reason to rebuild them,
and the ERD decisions in `week2-build-brief.md` are still the ones to defend.

**Does change:** how Week 3's agent gets described. The engagement already carries three
non-negotiables — coverage from a retrieved action, graceful degradation on lookup
failure, and a human escalation path that carries the transcript. This map shows those
are not Meridian quirks; they are rows 2, 9 and 12 of the same list an enterprise buyer
is being shown. That is the frame for the debrief: not "we built a bot", but "we built
the four rows that matter for a warranty administrator and can say why the other eight
were out of scope."

**The one row to build a metric for:** row 10. Deflection has to be countable *before*
go-live — define what a deflected conversation is, build the report, then launch. A
dashboard built afterwards measures nothing, because nobody agreed what it was counting.

---

## Verification plan

Nothing above enters the build log as fact until it is checked. In order, because each
step gates the next:

1. **Org and edition** — `sf org display --target-org devorg`. Record the edition; it
   determines rows 6 and 10.
2. **Agentforce actually enabled** — Setup, not the licence page. A licensed-but-off org
   is the premise of this whole engagement; confirm the current state rather than assuming
   it is still off.
3. **Data Cloud** — `sf data query --target-org devorg --query "SELECT Id FROM DataspaceScope"`.
   An `sObject type not supported` error means not provisioned, and row 3 is blocked.
4. **Knowledge** — Setup → Knowledge Settings. Gates row 8.
5. **Einstein features present** — walk Setup → Einstein and write down what is actually
   listed. Do not infer from documentation.

Each answer goes in `build-log.md` with its date and what was run to get it. Rows that
come back unavailable stay in this document with the finding recorded — an accurate map
with four gaps is worth more than a complete one that is partly invented.

---

## Accepted risk

This document is written from product knowledge, not from the org, and Salesforce's AI
naming moves quickly. Two failure modes follow: a capability may exist under a name not
used here, and a name used here may have been superseded. The verification plan is the
control. Until it has been run, this is a map of where to look — not a statement of what
Meridian has.
