# Week 2 — The case management backbone

**Org:** `devorg` · **Time:** ~4 hours of Setup · **You build all of this by hand.**

Priya approved the proposal with one condition: *"Mel's team sees it before I do. Every
screen."* Daniel granted a sandbox and a warning: document every change.

---

## The ERD, decided

Draw this before you click — the engagement asks for it and you'll be asked to defend
it in the Week 6 debrief.

```
Account (customer)
   │
   ├──< Contact
   │
   └──< Asset  ────────<  Warranty_Entitlement__c
         (the appliance)     (the policy)
              │                     │
              └────── Case ─────────┘
                  (the claim)
```

### The three relationship decisions and why

**`Warranty_Entitlement__c` → Asset is a Lookup, not Master-Detail.**

Master-detail is tempting: a warranty without an appliance is meaningless, and it would
give you roll-up summaries. Reject it anyway. Meridian *replaces* appliances under
warranty — when a dishwasher is written off and swapped, the policy has to move to the
new Asset. Master-detail blocks reparenting by default, and you'd be unpicking it in
month three. Make the lookup **required** so you keep the integrity without the rigidity.

Say that sentence out loud in an interview and you sound like someone who has been
burned before.

**`Case` → `Warranty_Entitlement__c` is a Lookup, and it has to be.**

Not a choice — a platform limit. A standard object cannot be the *detail* side of a
master-detail with a custom object. Which means **no roll-up summary of claims against
the claim limit.** You'll need a flow or a formula for that. Note the constraint in your
build log now; it comes up again in Week 5 when someone asks why claim totals drifted.

**`Case` → Asset uses the standard `AssetId` field.**

Already on Case. Don't create a custom one. Reaching for a custom field when the
standard model already has it is the most common junior tell.

---

## 2.1 · Build `Warranty_Entitlement__c`

`Setup → Object Manager → Create → Custom Object`

- Label `Warranty Entitlement` / plural `Warranty Entitlements`
- Record Name: **Auto Number**, format `WE-{00000}`, starting 1
- Enable **Allow Reports**, **Allow Activities**, **Track Field History**, **Allow Search**
- Deployment Status: Deployed

| Field | Type | Configuration | Why it exists |
|---|---|---|---|
| **Asset** | Lookup(Asset) | Required. Child relationship name `Warranty_Entitlements` | The appliance covered |
| **Account** | Lookup(Account) | Required | The policy holder |
| **Plan Tier** | Picklist | **Restricted.** `Standard`, `Priority`, `Retailer VIP` | Drives the SLA tier in 2.4 — and a restricted list is what stops an import inventing a fourth tier |
| **Retailer** | Picklist | **Restricted.** `Harvey & Co`, `Domaine Living`, `Southgate Electrical`, `Direct` | Meridian administers for retailers; reporting by retailer is a renewal conversation |
| **Coverage Start** | Date | Required | |
| **Coverage End** | Date | Required | |
| **Claim Limit** | Currency(16,2) | | The cap on what this policy will pay |
| **Claims Paid To Date** | Currency(16,2) | Default 0 | Maintained by flow — the roll-up you can't have |
| **Policy Status** | Picklist | **Restricted.** `Active`, `Expired`, `Cancelled`, `Claim Limit Reached` | |
| **Is Currently Covered** | Formula (Checkbox) | see below | **The field the agent reads.** |

The formula — build this one carefully, it's the spine of the whole engagement:

```
AND(
  ISPICKVAL(Policy_Status__c, "Active"),
  TODAY() >= Coverage_Start__c,
  TODAY() <= Coverage_End__c,
  OR( ISBLANK(Claim_Limit__c), Claims_Paid_To_Date__c < Claim_Limit__c )
)
```

Why this matters more than it looks: in Week 3 the agent must answer *"is my dishwasher
covered?"* **without the LLM deciding.** A formula field makes coverage a computed fact
the agent retrieves, not a judgement it generates. When Daniel runs his security review
and asks how you stop the AI inventing coverage, this field is your answer.

---

## 2.2 · Case record types and lifecycle

Two support processes, because a claim and an enquiry are different work:

| Record type | Statuses |
|---|---|
| **Warranty Claim** | New → Entitlement Check → Technician Dispatched → Awaiting Parts → Resolved → Closed |
| **General Enquiry** | New → Working → Resolved → Closed |

`Setup → Object Manager → Case → Support Processes` first (the process picks the
statuses), **then** Record Types (the record type applies the process). That order trips
everyone once.

Add the custom Case fields: `Warranty_Entitlement__c` (Lookup), `Resolution_Category__c`
(restricted picklist — Repaired / Replaced / Not Covered / Customer Cancelled / Duplicate).

**Validation rule** `Closure_Requires_Resolution_Category` on Case:

```
AND(
  ISPICKVAL(Status, "Closed"),
  ISBLANK(TEXT(Resolution_Category__c))
)
```

Error on the field, message written for an agent and not for you: *"Choose a resolution
category before closing — it's what the monthly report runs on."*

---

## 2.3 · Queues and routing

Three queues on Case: **Claims**, **Enquiries**, **Escalations**. Leave Queue Email blank
so members are notified individually.

One Case assignment rule, `Meridian Case Routing`, with ordered entries — **first match
wins**, so order most-specific first:

1. Record Type = Warranty Claim **AND** Plan Tier = Retailer VIP → Escalations
2. Record Type = Warranty Claim → Claims
3. everything else → Enquiries

Remember this org already has an **active** `WBC Holiday Routing` rule from the Wired
Brain Coffee demos. Activating yours deactivates theirs. That is fine and expected —
this org now belongs to Meridian — but note it in the build log so you're not confused
in six weeks.

---

## 2.4 · Entitlements and milestones — the *other* entitlement

This is where people conflate two different things. Keep them apart:

- `Warranty_Entitlement__c` = **is it covered?** (commercial)
- Standard **Entitlement** + Entitlement Process + Milestones = **how fast must we
  respond?** (service level)

`Setup → Entitlement Settings` → enable Entitlement Management.

Business Hours first — **Meridian Support Hours**, Australia/Sydney, Mon–Sat 8:00–18:00.
Milestone timers run against this; get it wrong and your 8-hour VIP SLA breaches
overnight while nobody is working.

Three entitlement processes, one per tier, each with a **First Response** milestone:

| Tier | First Response | Warning at |
|---|---|---|
| Standard | 48 business hours | 40h |
| Priority | 24 business hours | 20h |
| Retailer VIP | 8 business hours | 6h |

---

## 2.5 · The agent workspace

A Lightning record page for Case that Mel's team would actually want. The one
requirement that matters: **the entitlement check must be visible without scrolling.**
Her agents juggle four screens per call — the whole point of this engagement is removing
three of them.

Use a Rich Text component with a visibility filter on the related warranty's
`Is_Currently_Covered__c` so coverage state reads at a glance, in colour, above the fold.

---

## Checkpoints before you call Week 2 done

- [ ] A Warranty Claim case shows the claim status path; a General Enquiry doesn't
- [ ] Closing a case without a resolution category fails, with your message
- [ ] `Is_Currently_Covered__c` flips to false when you set Coverage End to yesterday
- [ ] A Retailer VIP claim lands in Escalations, a standard claim in Claims
- [ ] The milestone tracker appears on a case with an entitlement attached
- [ ] Change log started — date, component, reason, requirement ID. Daniel's price of admission.

---

## When you finish

Tell me and I'll seed the fifteen records — accounts, contacts, assets, warranty
entitlements and cases spanning all three SLA tiers, with a couple of deliberately
expired policies and one at its claim limit so Week 3's agent has real edge cases to
fail on. Hand-typing that is an hour you don't need to spend.

Then run the simulated Mel walkthrough: I'll play her, she'll have three objections,
and you change one design decision in response and log it with the reasoning. That
exercise is worth more in an interview than the config is.
