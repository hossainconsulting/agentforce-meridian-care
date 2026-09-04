# The service-agent pattern library

**What this is:** twenty-five reusable agent patterns for service businesses, with the
integration and compliance cost of each, and an honest map of which ones the Meridian
build already covers.

**Where it came from:** a public marketing infographic — *"200 Business Service Agent
Ideas for Service Businesses"* (AI Matt, @aimattant) — laying out 25 agent ideas across
eight niches: trades, cleaning, landscaping, clinics, salons, repair shops, agencies,
consultants. Read column by column it looks like 200 opportunities. Read row by row it
collapses.

**The finding:** it is roughly **25 patterns re-skinned across 8 verticals**. Missed-call
capture appears in every column. So does booking, reminder, review request, invoice chase,
handoff summary and FAQ. Trades' *"warranty question triage"* is Clinics' *"insurance
document chase-up"* with different nouns. Salons' *"waitlist filler"* is Trades'
*"cancellation soft filler"*.

That is not a criticism of the board — it is the useful part. **The unit of reuse is the
pattern, not the vertical.** Build the triage-and-handoff engine once and the eight
columns become eight configurations, not eight builds. Everything below is written from
that premise.

> Analysed 2026-09-04. Meridian Appliance Care is a fictional company; the compliance
> notes here are general Australian context for a real build, not legal advice, and
> should be confirmed with someone qualified before anything ships to customers.

---

## 1 · The risk tiers

Every pattern in this library gets a tier, and the tier decides the architecture. This is
the single most important table in the document.

| Tier | What a wrong answer costs | Architectural rule |
|---|---|---|
| 🟢 **Green** | A minute of someone's time. A reminder fires twice; an ETA is off. | Generation is fine. Ship it. |
| 🟡 **Amber** | Trust, or a booking. A misrouted enquiry, a bad summary, a review request to an angry customer. | Generation with a retrieved fact as the anchor, and a measured error rate. |
| 🔴 **Red** | Money, safety, or a legal breach. Coverage statements, clinical urgency, pricing, payment claims, anything touching a health record. | **The answer comes from a retrieved action, never from the model.** The model may only phrase what the action returned. |

The red-tier rule is the generalised form of the Meridian design constraint: coverage
answers come from `Warranty_Entitlement__c.Is_Currently_Covered__c`, not from the LLM.
The same logic applies to any statement a customer could reasonably act on or hold you
to. Under the Australian Consumer Law, a statement about repair rights, coverage or
remedy is a **representation** — an agent that improvises one is a compliance problem
wearing a chat interface.

---

## 2 · The twenty-five patterns

Grouped by lifecycle stage. "AI's actual job" is the column that matters: in most of
these, the intelligence is a thin band and everything around it is deterministic code.

### Stage 1 — Capture and qualify

| # | Pattern | Trigger → outcome | AI's actual job | Tier |
|---|---|---|---|---|
| 1 | **Missed-call capture** | Unanswered inbound call → SMS within 60 seconds capturing intent | Parse the reply into an intent and a callback preference | 🟢 |
| 2 | **Enquiry triage** | Inbound form / email / DM → classified and routed | Classify intent, extract entities, pick a queue | 🟡 |
| 3 | **Lead qualification** | New enquiry → qualified or politely declined | Judge fit against service area, job type, budget signal | 🟡 |
| 4 | **Urgency triage** | Any inbound → emergency path or normal queue | Detect the red flag (gas, water, electrical, clinical) | 🔴 |
| 5 | **Quote request intake** | Free-text + photos → structured scope on a record | Extract scope; describe what's in the photo | 🟡 |

Pattern 4 is red-tiered and people miss it. An agent that fails to recognise "I can smell
gas" or a cardiac description is not a bad chatbot, it is an incident. Emergency detection
should be keyword-and-rule first, model second, and it should over-escalate by design.

### Stage 2 — Book and schedule

| # | Pattern | Trigger → outcome | AI's actual job | Tier |
|---|---|---|---|---|
| 6 | **Booking agent** | Qualified enquiry → slot written to the calendar | Turn "sometime Thursday arvo" into a slot query | 🟡 |
| 7 | **Appointment reminder** | T-24h / T-2h → confirmation received | Almost none. This is a cron job with a template. | 🟢 |
| 8 | **Reschedule handler** | Inbound "can we move it?" → calendar updated | Interpret a vague new time; confirm | 🟢 |
| 9 | **Cancellation backfill** | Slot freed → offered to the waitlist in priority order | Draft the offer; rank who to ask first | 🟢 |
| 10 | **No-show recovery** | Missed appointment → rebooked or closed out | Tone. A no-show message that reads as a telling-off loses the customer. | 🟡 |
| 11 | **Route / ETA update** | Technician running late → customer told before they notice | None. Geofence plus template. | 🟢 |
| 12 | **Waitlist manager** | Capacity opens → waitlist worked down | Rank by fit and recency | 🟢 |

Seven of the twelve patterns so far are green, and several need no model at all. **This is
the board's own best advice and it is correct:** start with follow-up, scheduling and
reminders, and add AI only where language, judgment or triage actually appear. Most of the
value in Stage 2 is a working queue, not a clever one.

### Stage 3 — Deliver and hand off

| # | Pattern | Trigger → outcome | AI's actual job | Tier |
|---|---|---|---|---|
| 13 | **Visit note summariser** | Voice memo / scrappy notes → structured record | Genuine summarisation. The strongest fit on the whole board. | 🟡 |
| 14 | **Photo evidence summary** | Before/after images → written condition note | Vision description, tightly scoped | 🟡 |
| 15 | **Crew / clinician handoff** | Shift or ownership change → receiving party briefed | Compress a history into what the next person needs | 🔴 in clinical settings |
| 16 | **Escalation handoff** | Agent hits its limit → human takes over with full transcript | Summarise the conversation so far; never drop the raw transcript | 🟡 |
| 17 | **Checklist / SOP Q&A** | "What's the procedure for X?" → grounded answer | Retrieval over the actual SOP, with citation | 🟡 |

Pattern 16 is the one nobody builds and everybody needs. **Every conversation needs a
human escalation path, and the handoff must carry the transcript** — the failure mode that
destroys trust fastest is a customer who explains their problem twice. Ninety per cent of
"our chatbot was terrible" stories are a missing pattern 16.

### Stage 4 — Close and collect

| # | Pattern | Trigger → outcome | AI's actual job | Tier |
|---|---|---|---|---|
| 18 | **Invoice reminder** | Invoice ages past terms → escalating reminder sequence | Tone laddering across the sequence | 🟡 |
| 19 | **Payment status assistant** | "Has that been paid?" → answer from the ledger | Phrase a retrieved balance. Nothing else. | 🔴 |
| 20 | **Document collection** | Missing form / certificate / insurance doc → chased until received | Track what's outstanding; draft the chase | 🟡 |

Pattern 19 is red for the same reason coverage is: a model that guesses at a payment state
creates a dispute. Read the ledger or say you can't.

### Stage 5 — Retain and grow

| # | Pattern | Trigger → outcome | AI's actual job | Tier |
|---|---|---|---|---|
| 21 | **Satisfaction check-in** | Job closed + N days → sentiment captured | Read sentiment from a free-text reply | 🟡 |
| 22 | **Review request** | Positive sentiment confirmed → review link sent | Gate on 21. Never ask an unhappy customer. | 🟡 |
| 23 | **Complaint triage** | Negative sentiment or explicit complaint → owner assigned, clock started | Classify severity; route; never resolve autonomously | 🔴 |
| 24 | **Reactivation** | Dormant customer or recurring service due → re-engaged | Personalise on service history | 🟡 |
| 25 | **Renewal / upsell prompt** | Contract or plan approaching end → offer made | Draft the offer against actual usage | 🟡 |

Patterns 21 and 22 must be built as one thing. A review request that fires without a
sentiment gate is a machine for collecting one-star reviews at scale.

---

## 3 · What the board doesn't cost in

The infographic's implicit promise is that the idea is the hard part. It isn't. Four costs
sit between any cell on that grid and a working system.

**Integration surface.** The agent is perhaps 10% of the build. Auth, sync, retry,
reconciliation and the sandbox you test it in are the rest. Grouped by what they touch:

| System class | Australian examples | Patterns that need it |
|---|---|---|
| Telephony / SMS | Twilio, MessageMedia | 1, 7, 8, 10, 11, 18 |
| Field service / job management | ServiceM8, Jobber, simPRO, Tradify | 5, 6, 9, 12, 13, 14 |
| Practice management | Cliniko, Halaxy, Best Practice | 4, 6, 7, 10, 15 |
| Booking / rostering | Fresha, Timely, Square | 6, 8, 9, 12 |
| Accounting | Xero, MYOB | 18, 19, 20, 25 |
| CRM / case | Salesforce, HubSpot | 2, 3, 16, 21, 23, 24 |

**Data readiness.** Pattern 22 needs to know the job finished *and* went well. Pattern 1
needs call logs joined to customer records. Pattern 19 needs a ledger the agent can read.
Most small operators cannot answer these questions today, and no amount of model quality
fixes an unqueryable "job complete" flag.

**Volume economics.** A three-van plumbing business misses maybe fifteen calls a week. At
ten minutes recovered per call that is two and a half hours — real, but small. The
economics concentrate in businesses with contact-centre-scale volume. Meridian, at 12,400
cases a month and 35 seats, is on the right side of that line. The solo tradie in the
board's hero image is not, and selling to them is selling a hobby.

**"Owned, not rented."** The board's closing claim is positioning, not architecture. You
own the orchestration logic, the prompts and the data. You rent the model API, the SMS
gateway, the scheduling SaaS and the host. Worth saying plainly to any client who repeats
the phrase back to you.

---

## 4 · The Australian compliance layer

Roughly a third of the board's cells carry an obligation the board never mentions. General
notes, to be confirmed before any of these ship:

| Regime | Bites which patterns | The practical constraint |
|---|---|---|
| **Spam Act 2003 (Cth)** | 21, 22, 24, 25 — and arguably 9, 12 | Commercial electronic messages need consent, sender identification and a functional unsubscribe. Review requests, reactivation and upsell offers are commercial. |
| **Spam Act — transactional carve-out** | 7, 8, 10, 11, 18, 20 | Messages about an existing booking or debt are generally factual rather than commercial. Still collect consent at booking; the line moves the moment an offer is attached. |
| **Do Not Call Register Act 2006** | Any outbound-dialling variant | Applies if a pattern places marketing calls rather than sends messages. |
| **Privacy Act 1988 — APPs** | Everything that stores a conversation | APP 3 collection, APP 5 notification, APP 6 use and disclosure, APP 11 security. Transcripts are personal information. |
| **Privacy Act — health** | The entire clinics column: 4, 6, 10, 15, 20 | Health service providers are covered **regardless of turnover** — the small-business exemption does not apply. State law stacks on top (e.g. NSW *Health Records and Information Privacy Act 2002*). |
| **Australian Consumer Law** | 4, 5, 17, 19, 23 — and every coverage answer | Statements about coverage, remedy, repair rights or price are representations. A generated one is misleading conduct risk. |

Add to this a practice standard rather than a statute: **disclose that the customer is
talking to an AI**, and make the escalation path visible in the first message. Not yet a
general legal requirement in Australia, but it is where expectations are heading and it
costs nothing.

---

## 5 · The map to Meridian

Which of the twenty-five the engagement already answers, and where.

| # | Pattern | Status in the Meridian build | Component |
|---|---|---|---|
| 2 | Enquiry triage | ✅ Covered | Case record types + `Meridian Case Routing` assignment rule |
| 4 | Urgency triage | 🟨 Partial — tier-based, not content-based | Retailer VIP → Escalations queue |
| 16 | Escalation handoff | ✅ Designed, build pending | Week 3 agent, transcript-carrying handoff |
| 17 | Checklist / SOP Q&A | 🟨 Planned | Agent FAQ topic |
| 19 | *Coverage* status assistant | ✅ Covered — the red-tier exemplar | `Is_Currently_Covered__c` formula, read by a retrieved action |
| 21 | Satisfaction check-in | ❌ Out of scope | — |
| 22 | Review request | ❌ Out of scope | — |
| 23 | Complaint triage | 🟨 Partial | `Resolution_Category__c` captures outcome, not severity routing |
| 13 | Visit note summariser | ❌ Out of scope | Case comments exist; no summarisation |
| 18/20 | Invoice, document collection | ❌ Out of scope | No billing object in the data model |

Two observations worth carrying into the Week 6 debrief:

1. **Meridian is a single-cell deployment done properly.** The whole engagement is
   essentially pattern 19 plus 2, 16 and 17, built to a standard the board never
   describes: a computed coverage fact, a restricted picklist so imports can't invent a
   tier, an SLA clock on business hours, and deflection counted before go-live rather
   than after. The board sells breadth. The value is in depth.
2. **The gaps are deliberate, not oversights.** Patterns 21–25 need a marketing consent
   model Meridian doesn't have, and 18–20 need a billing integration outside the six-week
   scope. Saying which cells you *chose not to build*, and why, is the difference between
   a scoped engagement and an unfinished one.

---

## 6 · The pick-one test

If a client points at the grid and asks which cell to build first, four criteria, in
order:

1. **Frequency** — does it happen daily? Weekly is too rare to learn from.
2. **Blast radius** — green tier only for a first build. Never start on a red cell.
3. **Baseline** — can you measure today's performance *before* you build? If not, you
   cannot prove the thing worked, and an unprovable win gets switched off in month four.
4. **Single integration** — one system to write to. Two systems triples the timeline.

Scored that way, the board's own "best places to start" box is right: missed-call
follow-up, bookings and reminders, enquiry qualification, review follow-up. Patterns 1, 6,
7 and 21/22 — three green, one amber, all with an obvious baseline.

The Meridian corollary, and the rule this engagement is built on: **define how you count a
deflected conversation, build the report, then launch.** Not the other way around.
