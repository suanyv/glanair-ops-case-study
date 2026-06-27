# GlanAir Home Solutions — Customer Operations System Case Study

> A reconstruction of a real consumer-facing operation, reframed through the lens of SaaS customer operations: account management, support workflows, health monitoring, SLA enforcement, and scalable lifecycle tracking.

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Operational Context](#operational-context)
- [Key Operational Problems](#key-operational-problems)
- [System Design](#system-design)
- [Customer Health Model](#customer-health-model)
- [Support Operations](#support-operations)
- [Automation & Workflow Triggers](#automation--workflow-triggers)
- [Metrics](#metrics)
- [Scalability & Observability](#scalability--observability)
- [Lessons Learned](#lessons-learned)
- [Why This Project Matters](#why-this-project-matters)

---

## Executive Summary

GlanAir Home Solutions was a direct-to-consumer product pilot selling a single SKU — a HEPA dehumidifier — via online marketplace listing in Ireland. Over 32 days in February–March 2026, the operation processed **20 completed transactions**, handled **30+ customer enquiries**, resolved **15+ recurring support requests**, and achieved **5.0/5 customer satisfaction signals** across all documented interactions, under constrained inventory and without any dedicated tooling.

Every customer interaction was managed manually: inbound messages triaged by eye, responses drafted from memory, purchase coordination handled over WhatsApp, and satisfaction signals collected informally. The operation worked — but only because volume stayed low.

This project reconstructs that operation as a structured customer operations system. The goal is not to document a CRM implementation. It is to demonstrate how the operational problems that emerged at small scale map directly to the challenges that arise in high-volume SaaS customer success environments — and how structured systems, automated workflows, and observability infrastructure address them.

The reconstructed system includes: centralised account and contact management, opportunity tracking, a support case queue with SLA monitoring, customer health records, a FAQ knowledge base, inventory alert automation, and an AI-assisted support routing agent. Screenshots of the live system are included throughout.

---

## Operational Context

The GlanAir pilot operated through a single marketplace listing on Adverts.ie. Inbound interest arrived via the platform's messaging system. There was no CRM, no ticketing system, no structured handoff process, and no way to distinguish between a buyer who had already purchased and someone who was still evaluating.

**Acquisition** was entirely inbound. Buyers discovered the listing organically and initiated contact. The seller's job was to respond quickly, answer questions accurately, and coordinate collection — typically from a residential address in Dublin 8, cash or Revolut on collection.

**Account management** was implicit. There were no accounts. Each conversation was a standalone thread. If a buyer asked a question, purchased, and then followed up post-sale, those interactions lived in separate message threads with no linking context. Repeat enquirers and purchasers were indistinguishable in the queue.

**Support load** was heavier than expected for a €39 product. Buyers asked detailed questions about room suitability, allergen capture, noise levels, energy consumption, warranty coverage, and collection logistics. Many of these questions recurred across multiple buyers without any knowledge base to draw from. Each required a fresh, manually drafted response.

**Inventory constraints** introduced pressure. Stock was fixed at 20 units. As units sold, the window for new buyers narrowed. When stock hit zero on 19 March 2026, five qualified buyers were still actively enquiring. There was no mechanism to notify them systematically or track demand overflow for future planning.

**Response time** was a competitive variable. Buyers on marketplace platforms compare multiple listings. Slow responses lose sales. The operation maintained a sub-30-minute response target informally — but that target was never monitored, never enforced, and never visible.

---

## Key Operational Problems

These are not complaints about running a small operation manually. They are operational failure modes that would become serious at higher volume.

**Fragmented customer state.** Each buyer existed as an isolated message thread. There was no unified view of a customer's enquiry history, purchase status, and post-sale follow-up in one place. A buyer who asked three questions across three days required mental recall to serve consistently.

**Repetitive support load with no deflection mechanism.** Approximately 15 distinct questions recurred across the buyer pool. "Does it help with allergies?" "Is the filter included?" "Does it come with a warranty?" "Can I collect in the evenings?" These were answered individually, from memory, every time. The absence of a knowledge base meant support cost scaled linearly with volume.

**No centralised visibility into pipeline status.** There was no way to see, at a glance, how many buyers were actively enquiring, how many had confirmed purchase intent, how many had collected, and how many had gone quiet. Decisions about whether to hold stock for a specific buyer were made without data.

**Inventory blind spots.** Stock levels were tracked mentally. When inventory dropped to critical levels, there was no alert, no task generated, no notification to pending buyers. The sell-out event on 19 March was handled reactively.

**No lifecycle health tracking.** Post-sale follow-up happened when remembered, not systematically. There was no record of which buyers had been checked in with, which had reported satisfaction, and which had gone silent after collection. Customer health was invisible.

**Response time had no enforcement layer.** The informal 30-minute target was aspirational. If a message came in during a busy period and sat unanswered for two hours, there was no alert, no escalation, no record of the breach.

---

## System Design

The reconstructed system maps the GlanAir operation onto a customer operations architecture that would be sustainable at higher volume. The design prioritises operational reasoning over any specific tooling — the same logic applies whether the underlying platform is Salesforce, HubSpot, Zendesk, or a purpose-built SaaS CS tool.

### Data Model

```
Account
  └── Contacts (15 records)
        └── Opportunities (20 records — Closed Won / Closed Lost)
        └── Cases (30+ records)
              └── Customer Health Records (7 records)

Inventory Log (custom object)
FAQ Knowledge Records (15 records)
```

**Accounts** represent the organisational entity — in this case, GlanAir Home Solutions itself as the vendor. In a multi-product or B2B context, each buyer organisation would be a separate account. The account record holds SLA tier, customer priority, and upsell potential.

**Contacts** represent individual buyers — 15 records corresponding to buyers who engaged meaningfully. Each contact links to their purchase history, support cases, and health record. This is the unified view that was absent in the original operation.

**Opportunities** track the commercial lifecycle of each enquiry. The 20 Closed Won records correspond to completed sales at €39 each. The 5 Closed Lost records correspond to buyers who were active in the pipeline when stock ran out on 19 March. A Lead-to-Opportunity automation triggers on new contact creation, advancing inbound enquiries through Needs Analysis before manual qualification.

**Cases** are the primary support record. 30+ cases capture the full range of customer interactions: pre-sale product questions, collection coordination, post-sale follow-up, and stock availability checks. Cases carry subject, status, priority, and SLA due date fields that enable queue management and breach detection.

**Customer Health Records** are a custom object linking to the contact. Seven records were created — one per buyer who received structured post-sale follow-up. Fields include: satisfaction score, follow-up date, feedback notes, and health status (Green / Amber / Red). A post-sale trigger flow creates the health record automatically when an opportunity closes as Won.

**FAQ Knowledge Records** are a custom object with 15 entries covering the questions that recurred most frequently across the buyer pool. Each record stores the question, the canonical answer, and a category tag (Product Specs, Health & Allergens, Pricing, Logistics, Warranty). This base feeds the AI support routing agent.

**Inventory Logs** track units in, units sold, units remaining, and restock alert status. A threshold-triggered automation fires when remaining units drop to or below a defined level, creating a high-priority restock alert task.

See `/diagrams/` for visual representations of key flows.

---

## Customer Health Model

In a SaaS context, customer health scoring is a leading indicator of churn risk and expansion potential. The same logic applies at any scale: you need a structured signal that something is wrong before the customer disengages.

The GlanAir health model is lightweight but structurally sound.

### Health Record Fields

| Field | Type | Purpose |
|---|---|---|
| Health Status | Picklist (Green / Amber / Red) | Overall account risk signal |
| Satisfaction Score | Number (1–5) | Post-sale CSAT proxy |
| Last Follow-Up Date | Date | Recency of engagement |
| Follow-Up Completed | Checkbox | Whether post-sale check-in occurred |
| Feedback Notes | Text | Qualitative signal from interaction |
| Response Time (mins) | Number | Actual response time at point of enquiry |
| Risk Indicators | Text | Free-form flags (e.g. "buyer reported unit fault") |

### Health Scoring Logic

```
Green:  Satisfaction ≥ 4  AND  Follow-Up Completed = true  AND  No open risk flags
Amber:  Satisfaction = 3  OR   Follow-Up pending > 3 days  OR   One unresolved case
Red:    Satisfaction ≤ 2  OR   Follow-Up never completed   OR   Fault or complaint logged
```

### Risk Indicators Observed in This Operation

- Buyers who asked multiple questions without purchasing (possible objection not surfaced)
- Buyers who confirmed collection intent but did not follow through within 48 hours
- Stock-out notification recipients — these are accounts with confirmed purchase intent and no resolution path
- Post-sale silence — buyers who collected and sent no follow-up signal

In a SaaS context, these translate directly to: users who complete onboarding but show no product engagement; accounts approaching renewal without QBR completion; customers who opened a support ticket and received no response within SLA.

---

## Support Operations

### Case Categories Observed

| Category | Example Cases | Volume |
|---|---|---|
| Product Specifications | Room size suitability, noise levels, weight, air quality display | High |
| Health & Allergens | Asthma suitability, dust capture, pet odours, cigarette smoke | High |
| Pricing & Offers | Total cost, payment methods, negotiation attempts | Medium |
| Logistics | Collection hours, evening availability, Dublin address | Medium |
| Warranty & After-Sales | Warranty duration, filter replacement, fault reporting | Medium |
| Stock Availability | "Do you have more than one?", "Is it still available?" | Medium |
| Post-Sale Follow-Up | Satisfaction check, collection confirmation | Low |

### SLA Logic

The operation maintained an informal sub-30-minute response target. The reconstructed system formalises this as an automated SLA monitor.

**SLA Flow (GlanAir SLA Response Time Monitor — V1):**
1. Case created → SLA due date set to 30 minutes from creation
2. Scheduled path fires at T+30 minutes
3. Decision node: Is case still open?
   - Yes → SLA Breached → Create SLA Breach Alert task, high priority
   - No → Case resolved within SLA → no action

This is the minimum viable SLA enforcement layer. At higher volume, this logic would feed a breach-rate dashboard, enable tier-based SLA differentiation, and trigger escalation to a supervisor queue.

### FAQ Deflection

15 FAQ records were catalogued from recurring case subjects. Estimated deflection opportunity: approximately 50% of pre-sale cases could have been resolved without human handling if an FAQ routing layer had been active at launch.

**FAQ Routing Flow (GlanAir Agentforce FAQ Routing — V1):**
- Case arrives → Decision: FAQ match found?
  - Yes → Tag case for AI agent handling, route to Agentforce
  - No → Create manual review task, route to human queue

### AI Support Agent

The GlanAir Support Agent (built in Agentforce) operates with 5 domain-specific subagents:

- **Allergen and Health Claims** — handles HEPA filter questions, asthma and dust queries
- **Product Specifications** — room size, noise, weight, power consumption
- **Pricing and Offers** — cost, payment methods, bundle questions
- **Collection and Logistics** — hours, address, WhatsApp coordination
- **Warranty and After Sales** — warranty terms, filter replacement, fault handling

The agent demonstrates structured knowledge routing: ambiguous queries are classified before response generation, reducing hallucination risk on factual product claims. This is directly analogous to how a well-structured CS knowledge base reduces escalation rate in a SaaS support environment.

---

## Automation & Workflow Triggers

Five record-triggered flows were built and activated:

| Flow | Trigger | Action |
|---|---|---|
| Lead to Opportunity Conversion | New Contact created | Creates linked Opportunity in Needs Analysis stage |
| Post-Sale Customer Health Trigger | Opportunity → Closed Won | Creates Customer Health Record + 3-day follow-up task |
| Inventory Threshold Alert | Inventory Log updated, units remaining ≤ threshold | Creates high-priority restock alert task |
| SLA Response Time Monitor | Case created | Scheduled path at T+30 min checks if case still open; creates breach alert if yes |
| Agentforce FAQ Routing | Case created | Routes to AI agent if FAQ match detected; creates manual review task if not |

These flows encode operational decisions that were previously implicit. They are the difference between a system that generates visibility and one that requires constant manual attention to stay in view.

---

## Metrics

| Metric | Value |
|---|---|
| Total units available | 20 |
| Units sold | 20 |
| Sell-through rate | 100% |
| Sale price per unit | €39 |
| Total revenue | €780 |
| Days to sell out | 32 |
| Closed Lost (stock-out) | 5 qualified buyers |
| Total customer interactions (cases) | 30+ |
| Contacts (buyers engaged) | 15 |
| Positive feedback events (CSAT signals) | 7 |
| Informal response time target | < 30 minutes |
| FAQ records catalogued | 15 |
| Estimated FAQ-deflectable cases | ~15 (50% of pre-sale volume) |
| Customer Health Records created | 7 |
| Automation flows active | 5 |
| AI agent subagents | 5 |
| Demand exceeding supply at sell-out | 5+ active buyers unresolved |

The 5 Closed Lost records are operationally significant. They represent confirmed demand that could not be fulfilled — a demand signal that, if captured systematically, would justify a restock decision. In a SaaS context, these are the accounts that churned not because of dissatisfaction but because of a capacity or availability failure on the vendor side.

---

## Scalability & Observability

This section addresses the most important question the project raises: **what breaks when volume increases?**

### Visibility Gaps at Scale

At 20 customers, mental models work. A single operator can hold the full pipeline in their head. At 200 customers, or 2,000, the following signals become invisible without instrumentation:

- **Response time distribution.** At low volume, you feel when you're slow. At high volume, you need a time-series view of response latency by case category, time of day, and agent. Breaches become invisible until they cause churn.

- **Support volume by category.** In this operation, health/allergen questions and logistics questions dominated. That signal only became clear in retrospect. A live case volume dashboard by category would have triggered earlier FAQ creation.

- **Accounts approaching risk.** Seven health records were created manually after the fact. In a live operation with 200 accounts, health status needs to be computed automatically from activity signals — last contact date, open case count, CSAT score, response time history — and surfaced as a risk queue before customers disengage.

- **Pipeline conversion rate.** The operation converted 20 of approximately 25 engaged buyers — roughly 80%. That number is not in any report. It exists only because the records were counted manually for this case study. At scale, conversion rate by cohort, by acquisition source, and by product question cluster needs to be a live metric.

- **Demand overflow.** Five buyers were in active pipeline when stock hit zero. There was no waitlist, no demand capture mechanism, no way to quantify unmet demand for a restock decision. The operational signal was available — the cases existed — but there was no query or alert to surface it.

### What Should Be Automated

- **Health score recalculation** on any change to a linked case, opportunity, or satisfaction record. Health should be a computed field, not a manually updated one.
- **At-risk account alerts** when health drops to Amber or Red, or when no activity has been logged against an account for more than N days.
- **SLA breach escalation** beyond task creation — Slack notification, supervisor assignment, case priority bump.
- **FAQ gap detection** — when a case is routed to human queue after failing FAQ match, that question should be automatically flagged for knowledge base review.
- **Demand capture** — cases opened on sold-out accounts should trigger a waitlist record and a notification workflow.

### Monitoring Needs

The operational signals that matter in this system map directly to the metrics that matter in a SaaS CS environment:

| Operational Signal | SaaS Equivalent |
|---|---|
| Response time per case | Time-to-first-response by tier |
| Open cases by category | Support volume by product area |
| Health status distribution | Account health heatmap |
| Closed Lost (stock-out) count | Churn by reason code |
| FAQ match rate | Self-serve deflection rate |
| Post-sale follow-up completion rate | QBR completion rate |
| Days since last customer contact | Account engagement recency |

Datadog's core value proposition — making complex systems observable — applies equally to customer operations infrastructure. The same instinct that drives engineers to instrument their services drives CS operations teams to build dashboards that surface account risk before customers raise it themselves. Visibility is not a nice-to-have. It is the mechanism by which proactive support becomes possible.

### Proactive vs Reactive Support

The original operation was entirely reactive. Every interaction was initiated by the customer. The reconstructed system creates three proactive touchpoints:

1. **Post-sale follow-up task** — triggered automatically 3 days after close, ensuring every buyer receives a check-in before the return window closes.
2. **SLA breach alert** — surfaces cases that have gone unanswered before the customer has to follow up themselves.
3. **Inventory alert** — creates visibility into supply pressure before it causes pipeline failures.

This is the structural difference between a support function and a customer success function. Support waits for problems to be reported. Customer success detects risk signals before they become problems.

---

## Lessons Learned

**Manual systems have a volume ceiling that arrives faster than expected.** The operation handled 20 customers without breaking. At 40 or 60 customers, the same approach would have produced missed messages, inconsistent responses, and untracked follow-ups. The ceiling is not a function of effort — it is a function of cognitive load. Structured systems reduce cognitive load by externalising state.

**Repetitive support load is a system design problem, not a training problem.** Answering the same question 15 times is not a sign that the operator needs to be faster. It is a sign that a knowledge base needs to exist. The first time a question recurs, it should be captured. By the third time, it should be deflectable.

**Inventory as a customer success signal.** Stock availability directly determined whether qualified, engaged buyers could become customers. The five Closed Lost records represent not a sales failure but an operational visibility failure — no alert, no waitlist, no demand capture, no proactive outreach. Treating inventory state as a customer success signal, not just a logistics metric, would have changed the outcome for those five accounts.

**Response time is a retention variable.** In a marketplace context, buyers comparing multiple listings will close with whoever responds first and most clearly. In a SaaS context, time-to-first-response on a support ticket affects renewal probability. The mechanism is the same: customers interpret responsiveness as a proxy for reliability.

**Instrumentation enables proactive action.** The health records, the SLA monitor, the post-sale follow-up trigger — these are not administrative overhead. They are the infrastructure that makes proactive customer success possible at scale. Without them, the operator is always in reactive mode, always one step behind the customer.

---

## Why This Project Matters

This project was built to demonstrate one specific thing: that I understand how customer operations systems behave under load and how structured workflows improve customer outcomes.

The skills demonstrated here map directly to a Customer Success Associate role at a company like Datadog:

**High-volume account management.** The ability to design a data model that gives every account a consistent, structured record — contacts, interactions, health status, open issues — is the foundation of managing a large book of business without losing signal on any account.

**Retention awareness.** The Customer Health Model is a direct application of retention thinking. Identifying risk signals before customers disengage, creating systematic follow-up touchpoints, and tracking satisfaction at the account level are the operational mechanics of retention.

**Proactive support.** The SLA monitor and post-sale follow-up automation demonstrate the operational difference between reacting to customer problems and detecting them in advance. At Datadog's scale, where customers are running critical production infrastructure, proactive communication is not optional.

**Operational coordination.** The five automation flows represent the kind of cross-functional coordination — between sales, support, and operations — that CS associates navigate daily. Knowing when to trigger a handoff, how to structure an escalation, and what information needs to move between systems is operational literacy.

**Identifying usage and risk patterns.** The FAQ categorisation, the demand overflow analysis, and the health scoring logic are all exercises in pattern recognition from operational data. CS associates who can read their account data and surface risk trends to their team add disproportionate value beyond their individual book of business.

**Customer advocacy under pressure.** Managing 5 Closed Lost buyers at sell-out — communicating transparently, maintaining relationship quality with accounts that couldn't be converted due to supply constraints, and documenting demand signals for future planning — is the kind of situation where customer advocacy either builds or destroys trust. The operational response matters as much as the commercial outcome.

---

## Repository Structure

```
/
├── README.md                          # This document — main case study
├── docs/
│   ├── system-design.md               # Data model and object relationships
│   ├── automation-flows.md            # Flow documentation and logic
│   ├── support-operations-runbook.md  # Case handling procedures
│   └── health-model-spec.md           # Customer health scoring specification
├── diagrams/
│   ├── customer-inquiry-flow.svg      # Inbound enquiry to resolution
│   ├── customer-lifecycle.svg         # Full buyer lifecycle map
│   ├── support-workflow.svg           # Case triage and routing
│   ├── escalation-path.svg            # SLA breach escalation logic
│   └── inventory-alert-workflow.svg   # Stock threshold trigger flow
├── screenshots/                       # Live system screenshots
│   └── [system screenshots]
└── metrics/
    ├── operations-summary.md          # Aggregated operational metrics
    └── support-load-analysis.md       # Case volume and category breakdown
```

---

*Built as a portfolio project to demonstrate customer operations systems thinking. All customer data is anonymised or fictionalised. The operational events described reflect a real product pilot; the CRM reconstruction was built subsequently as a learning exercise.*
## API Troubleshooting Fundamentals

Demonstrating the ability to authenticate, query, and diagnose API failures using Postman against the GlanAir Salesforce org.

- **OAuth 2.0 authentication** (Authorization Code with PKCE) — the same handshake used by Shopify, HubSpot, Stripe, and most B2B SaaS integrations
- **Full CRUD cycle** — GET, POST, PATCH, DELETE against live case and contact data
- **Error diagnosis** — deliberately broken requests reproducing 401, 400, and 404 failures with documented root causes and fixes
- **Rate limit observability** — reading `Sforce-Limit-Info` headers to monitor API quota

Full walkthrough with screenshots: [`https://github.com/suanyv/api-troubleshooting-fundamentals`](https://github.com/suanyv/api-troubleshooting-fundamentals)

