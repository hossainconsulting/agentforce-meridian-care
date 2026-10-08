# The five layers of AI guardrails, applied to Meridian

**Engagement:** The Meridian Engagement · **Phase:** Week 3 design, drafted ahead of build
**Author:** Hemayet Hossain · **Audience:** Priya Raman (sponsor), Daniel (IT/security), Mel (contact centre)
**Status:** Draft for review. The gap register at the end contains five decisions Meridian must make before go-live.

---

## Why this document exists

Meridian licensed Agentforce after a board directive and nobody switched it on. The
common reading of that is a technical delay. It isn't. The agent could have been
switched on in an afternoon; what's missing is anyone willing to own what it says to a
customer. That is a guardrails problem, and guardrails are not a feature you enable.

The framework below has five layers — **Culture, Governance, Operating Model, Process,
System** — each containing the one inside it. The claim the framework makes, and the
claim this document tests against Meridian, is that a control at any single layer fails
alone. A content filter with no policy behind it is a setting nobody maintains. A
policy with no training behind it is a document nobody reads. The layers hold each
other up.

**The finding, stated up front:** Meridian's agent design rules — coverage from a
retrieved action, graceful degradation, a human escalation path with transcript,
countable deflection — are strong and they are *all* System and Process layer. Every
layer above them is currently unowned. That is the actual risk in this engagement, and
it is not fixed by building better.

### What this document is not

It is not an AI policy. A 35-seat warranty administrator that adopts a
fifteen-page enterprise AI policy will not follow it, and an unfollowed policy is worse
than none — it converts a known gap into a documented failure. Everything below is
sized for a company of Meridian's actual shape: one contact centre, one IT person, one
executive sponsor, and no appetite for a committee.

---

## Layer 1 — System

> *Build technical safeguards into the AI system itself. Prevents unsafe, inaccurate or
> sensitive outputs from reaching users.*

This is the layer the platform gives you and the layer most teams mistake for the whole
job. At Meridian it is also the layer that is furthest along, because the Week 2 data
model was built with it in mind.

| Control | What it is at Meridian | Owner | Status |
|---|---|---|---|
| **Prompt guardrails** | Topic scope and instructions on the Warranty Enquiry topic. The agent may check coverage, log a claim and answer policy questions. It may not quote a repair cost, promise a timeframe outside the SLA table, or discuss another customer's policy. | Hemayet → Mel | Week 3 |
| **Output validation** | Coverage is never generated. `Is_Currently_Covered__c` is a formula field read by a retrieved action; the agent reports the value, it does not compute it. | Built (Week 2) | ✅ |
| **Content filters** | Einstein Trust Layer toxicity scoring, on by default. Not configured further — Meridian's conversation surface is warranty claims, not open chat. | Daniel | Default |
| **PII detection** | Trust Layer masking before the prompt reaches the model. Relevant here: customer names, addresses and phone numbers appear in every case. | Daniel | Verify in Week 3 |
| **Agent identity** | The agent runs as a dedicated user with its own permission set. It gets Read on `Warranty_Entitlement__c`, Create/Edit on Case, and nothing else. No Delete anywhere. No access to Contract or Order (the residual demo objects). | Hemayet | Week 3 |
| **Platform controls** | Validation rules, restricted picklists and the closure rule apply to the agent exactly as to a human. `Closure_Requires_Resolution_Category` will block an agent-closed case with no category, and should. | Built (Week 2) | ✅ |

### What the Trust Layer does not cover

Worth stating plainly, because "we have the Einstein Trust Layer" is offered as a
complete answer more often than it is one. The Trust Layer grounds prompts in data the
running user can see, masks PII, enforces zero retention with the model provider, scores
output for toxicity and writes an audit trail. It does **not** protect against:

- badly written instructions — a topic told to be helpful about refunds will be
- an over-permissioned agent user — if it can edit the field, it will edit the field
- hallucination in anything not grounded in a retrieved record
- a correctly permitted action taken for the wrong business reason

Those four are Process and Governance problems. They are why this document has four more
sections.

**If this layer alone were missing:** the agent invents coverage. A customer is told
their dishwasher is covered, a technician is dispatched, and Meridian eats the call-out
fee on a policy that expired in March.

---

## Layer 2 — Process

> *Embed safeguards into delivery. Ensures AI outputs pass through human accountability
> before deployment.*

The System layer decides what the agent *can* do. This layer decides what happens
around each thing it does.

| Control | What it is at Meridian | Owner | Status |
|---|---|---|---|
| **Human review** | Pre-launch: Mel's team reviews 100% of agent conversations for the first two weeks. Post-launch: a weekly sample of 20 conversations, plus 100% of any conversation that ended in escalation. | Mel | Week 4 |
| **Approval paths** | The agent may log a claim. It may not approve one. Any outcome with money attached — a fee waiver, a goodwill credit, an out-of-policy repair — routes to a human, with no exception path. | Priya | **Gap** |
| **Monitoring** | Deflection rate, escalation rate, containment, and a defect log. Definitions in the measurement section below. | Hemayet → Mel | Week 5 |
| **Responsible use standards** | The agent identifies itself as an assistant in its opening message. It never claims to be a person, and never continues a conversation a customer has asked to leave. | Mel | Week 3 |
| **Shadow AI management** | The live question for Meridian, not a hypothetical: 35 agents under handle-time pressure, and ChatGPT is one tab away. Customer names and case details pasted into a consumer chatbot are a Privacy Act exposure that no Agentforce control touches. | Daniel | **Gap** |

### The escalation path is a process control, not a feature

The design rule says every conversation has a human escalation path and the handoff
carries the transcript. That is a System capability. The Process control is the part
that decides whether it works: **who is on the other end, within what time, and what
happens when nobody is.** An escalation button that queues into an empty queue at 6pm
Saturday is worse than no button, because the customer has now been promised a person.

Meridian's business hours are Mon–Sat 8:00–18:00. The agent must not offer escalation
outside them; it must offer a callback and create the case. That is a Week 3 build item
and it belongs in the topic instructions, not in someone's memory.

**If this layer alone were missing:** the agent is technically well-behaved and nobody
finds out it is wrong. The defect surfaces in month four as a CSAT number, unattributable
to any specific conversation because nobody was reading them.

---

## Layer 3 — Operating Model

> *Translate strategy into daily behaviour. Turns governance principles into repeatable
> organisational behaviour.*

Where the previous two layers are the consultant's to build, this one is Meridian's to
run after the consultant leaves. It is the layer that decides whether the engagement
survives contact with month three.

| Control | What it is at Meridian | Owner | Status |
|---|---|---|---|
| **Staff training** | Not "how to use the agent" — the agents don't use it, customers do. The training is: how to read an agent transcript on handoff, how to log an agent defect, and when to override it. Half a day, delivered by Mel, materials from this engagement. | Mel | Week 4 |
| **Usage policies** | One page, three rules: no customer data leaves Salesforce and approved tools; the agent's coverage answer is authoritative and a human override must be logged with a reason; anything with money attached goes to a human. | Priya | **Gap** |
| **Adoption guidelines** | How the next agent topic gets approved. Meridian will want a second one within a quarter — returns, or parts tracking. Without a gate, topic two ships with none of the scrutiny topic one got. | Priya + Daniel | **Gap** |
| **Compliance reviews** | Australian Privacy Principles apply to every conversation. APP 11 (security) and APP 6 (use and disclosure) are the two that bite. Quarterly check against the conversation log; annual review of what the agent is permitted to read. | Daniel | **Gap** |

### The one-page usage policy is the highest-leverage artefact here

Of the four gaps in this layer, three are documents nobody has written and one is a
recurring meeting nobody has scheduled. Written honestly, the usage policy is one page
and takes ninety minutes. It is proposed as a Week 4 deliverable of this engagement
rather than as homework left with the client, because homework left with a client at
this size does not get done.

**If this layer alone were missing:** the guardrails hold for as long as the consultant
is in the building. Six months on, a second topic has been added by someone who wasn't
here for the first one, the agent has been given Edit on a field to unblock a ticket,
and nobody can say when either happened.

---

## Layer 4 — Governance

> *Set accountability and oversight. Defines acceptable risk and assigns ownership for
> AI decisions.*

| Control | What it is at Meridian | Owner | Status |
|---|---|---|---|
| **Accountability model** | Named single owner for the agent's behaviour, with the authority to switch it off. Proposed: Priya accountable, Mel operationally responsible, Daniel accountable for data and access, Hemayet responsible for build during the engagement only. | Priya | **Gap — the critical one** |
| **Risk framework** | Not a matrix. Four named risks with a tolerance each: wrong coverage answer (zero tolerance — must be impossible by design, and is); PII exposure (zero); unhelpful-but-safe answer (tolerated, tracked); escalation the agent should have handled (tolerated, target under 15%). | Priya + Hemayet | Week 4 |
| **AI ethics committee** | Not appropriate at 35 seats. The independent-oversight function is met instead by Daniel's security review being a genuine gate — he can block go-live, and that authority must be stated rather than assumed. | Priya | **Gap** |
| **Compliance reviews** | See Operating Model. Same person, different cadence. | Daniel | **Gap** |

### The question this layer exists to answer

**Who owns the call when the agent and the record disagree?**

A customer rings. The agent, reading `Is_Currently_Covered__c`, says the policy lapsed
in March. The customer says they renewed in April and has an email. Mel's agent is now
holding a phone, a system that says no, and a customer who says yes.

Today at Meridian there is no answer to that. The Week 2 build made coverage a computed
fact precisely so the *machine* can't be wrong about what the record says — but the
record itself can be wrong, and someone has to be allowed to override it and be
accountable for the override. That authority, its limit, and where the override is
logged are a single decision Priya can make in one meeting, and nothing above the System
layer works until she does.

**If this layer alone were missing:** the first genuine dispute escalates to whoever
happens to be senior that day, is resolved inconsistently, and sets a precedent nobody
chose.

---

## Layer 5 — Culture

> *Embed AI ethics into organisational mindset. 32% of employees using AI at work hide
> it from their employer.*

The outermost layer and the one that cannot be built, only cultivated. It is also the
layer where Meridian's specific circumstances are most dangerous.

| Control | What it is at Meridian | Owner | Status |
|---|---|---|---|
| **AI literacy** | Mel's team needs to know what the agent can and can't do, in concrete terms, or they will either over-trust it or route around it. One session, examples not concepts. | Mel | Week 4 |
| **Psychological safety** | The specific fear: 0% deflection today, 12,400 cases a month, 35 seats. Every person in that contact centre can do the arithmetic on what successful deflection means for headcount. | Priya | **Gap** |
| **Leadership example** | Priya using the agent's own reporting in her board update, and being visible about a defect it produced, does more than any policy. | Priya | Week 6 |
| **Ethical judgment** | The standing rule, stated to the team rather than filed: the agent is never the reason a decision was made. A human made it, using the agent's answer. | Priya | **Gap** |

### The deflection conversation has to happen before go-live, not after

This is the risk most likely to sink the engagement and the least likely to appear in a
status report. If Mel's team believes the agent is a redundancy programme, they will not
report its defects — and the defect log is the entire Process layer's early-warning
system. The framework's own statistic is the point: a third of employees using AI at
work hide it from their employer. The corollary is that a team that fears the tool will
hide what it knows about the tool.

Priya needs to say what deflection means for headcount before Week 4, in whatever terms
are true. "Nobody loses their job over this" if it's true. "This is how we absorb next
year's volume without hiring" if that's the truth. What cannot work is silence, because
silence is read as the worst available answer, and the team's cooperation is a
prerequisite for every control in the Process layer.

**If this layer alone were missing:** every other layer is intact and slowly stops being
told the truth.

---

## Making deflection countable — a design rule with teeth

The engagement's non-negotiable is that deflection must be countable *before* go-live,
not after. That requires a definition that survives being audited, so here it is.

**A conversation counts as deflected when all four are true:**

1. It was initiated by a customer through the self-service channel
2. The agent completed the intent — a coverage answer given, or a claim logged with a case number issued
3. No escalation to a human occurred within the conversation
4. No case from that customer on the same asset was created through any other channel within 48 business hours

Condition 4 is the one that makes the number honest. Without it, a customer who gives up
on the agent and rings the contact centre is counted as a success, and a deflection
metric that counts failures is worse than no metric — it is a metric that will be
believed.

**Measured against a baseline of 0%.** The counting mechanism is a report built in Week
5, on data that exists because the fields were designed in Week 2. If the report cannot
be built before launch, the launch moves.

Secondary measures, so deflection is never read alone:
- **Escalation rate** — target under 15% of agent conversations
- **Repeat contact rate** — currently 28% overall; agent-handled conversations must not exceed it
- **CSAT on agent-handled conversations** — currently 71% overall; must not be below it
- **Defect rate** — logged agent errors per 100 conversations, trending down or the topic is wrong

---

## Traceability — guardrail to artefact

Every control above has to land somewhere a successor can find it. Nothing is a control
until it has a row here.

| Guardrail | Layer | Where it lives | Week |
|---|---|---|---|
| Coverage is retrieved, never generated | System | `Is_Currently_Covered__c` formula field | 2 ✅ |
| Agent cannot close a case sloppily | System | `Closure_Requires_Resolution_Category` validation rule | 2 ✅ |
| Agent cannot exceed its remit | System | Agent user permission set | 3 |
| Failed lookup degrades gracefully | System | Topic instructions + action error path | 3 |
| Escalation carries the transcript | System/Process | Agent handoff configuration | 3 |
| No escalation offered outside business hours | Process | Topic instructions + Meridian Support Hours | 3 |
| Money decisions go to a human | Process | Topic scope + approval path | 3 |
| Conversation review | Process | Mel's weekly sample + review log | 4 |
| Usage policy | Operating Model | One-page policy, `deliverables/` | 4 |
| Risk tolerances | Governance | This document, signed off | 4 |
| Override authority | Governance | Accountability decision + logged reason field | **Gap** |
| Deflection counting | Process | Report built on Week 2 fields | 5 |
| Every config change traceable | All | `deliverables/build-log.md` | Ongoing |

---

## Gap register — five decisions for Priya

Ranked by what blocks go-live. None of these are technical and none of them are the
consultant's to make.

| # | Decision | Blocks | Needed by |
|---|---|---|---|
| 1 | **Who owns the agent's behaviour and who may override a coverage answer, with the override logged where?** | Governance and everything above System | Week 4 |
| 2 | **What does deflection mean for the 35 seats, said out loud to the team?** | Culture; and the defect reporting the Process layer depends on | Before Week 4 training |
| 3 | **Is Daniel's security review a genuine gate that can block go-live?** | Independent oversight; substitutes for an ethics committee at this size | Week 4 |
| 4 | **What is the approval path for anything with money attached?** | Process; currently no defined route | Week 3 build |
| 5 | **How does a second agent topic get approved?** | Operating model durability past this engagement | Week 6 handover |

Decision 1 is the one to take first. It is a single meeting, and four of the five layers
are waiting on it.

---

## Assumptions

Stated so they can be corrected rather than inherited.

- Role titles for Priya, Daniel and Mel are inferred from the engagement brief. Confirm
  before this document is circulated.
- Meridian handles Australian consumer data only; the Privacy Act and the Australian
  Privacy Principles are the applicable regime. If any retailer relationship involves
  New Zealand customers, the review scope in the Operating Model layer changes.
- The agent's first release is a single topic — warranty coverage and claim logging.
  The controls above are sized for that. A second topic re-opens the Operating Model
  layer, which is why decision 5 exists.

---

*Source framework: "5 Layers of AI Guardrails" (Culture / Governance / Operating Model /
Process / System). Applied here to Meridian Appliance Care Pty Ltd, a fictional company;
this is a simulation deliverable, not client work.*
