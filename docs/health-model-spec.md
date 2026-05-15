# Customer Health Model Specification — GlanAir

## Purpose

This document specifies the health scoring logic, risk indicators, and monitoring approach used in the GlanAir Customer Health Records system.

---

## Health Record Object

Custom object: `Customer_Health_Record__c`

| Field | Type | Required | Notes |
|---|---|---|---|
| Health Record ID | Auto-number (CH-XXXX) | Yes | System-generated |
| Contact | Lookup (Contact) | Yes | Linked buyer record |
| Opportunity | Lookup (Opportunity) | No | Associated sale |
| Health Status | Picklist | Yes | Green / Amber / Red |
| Satisfaction Score | Number (1–5) | No | Captured at follow-up |
| Last Follow-Up Date | Date | No | Date of most recent check-in |
| Follow-Up Completed | Checkbox | Yes | Default: false |
| Feedback Notes | Long Text | No | Qualitative signal |
| Response Time (mins) | Number | No | Actual response time recorded |
| Risk Indicators | Text (255) | No | Free-form risk flags |

---

## Health Scoring Logic

Health status is assigned based on the combination of satisfaction signal, follow-up completion, and risk indicators.

```
GREEN
  Conditions:
    - Satisfaction Score ≥ 4
    - Follow-Up Completed = true
    - Risk Indicators = null or empty
  
  Interpretation: Account is healthy. No action required beyond routine
  check-in at next scheduled touchpoint.

AMBER
  Conditions (any one):
    - Satisfaction Score = 3
    - Follow-Up pending for > 3 days after close date
    - One unresolved case open > 48 hours
    - No response to follow-up outreach within 48 hours
  
  Interpretation: Account needs attention. Create a follow-up task
  within 24 hours. Do not wait for the customer to escalate.

RED
  Conditions (any one):
    - Satisfaction Score ≤ 2
    - Follow-Up never completed (> 7 days post-close)
    - Fault or complaint logged as case
    - Buyer explicitly expressed dissatisfaction in any channel
  
  Interpretation: Account at risk. Immediate action required.
  Escalate to High priority case if not already open.
  Personal outreach required — do not rely on automated messaging.
```

---

## Risk Indicators Used in This Operation

The following risk flags were observed and documented across the 7 health records created:

- `COLLECTION_DELAY` — Buyer confirmed collection intent but did not follow through within 48 hours
- `MULTI_QUESTION_NO_PURCHASE` — Buyer asked 3+ pre-sale questions without progressing to purchase (possible unresolved objection)
- `STOCK_OUT_AFFECTED` — Buyer was in active pipeline when stock reached zero; no resolution path available
- `SILENT_POST_COLLECTION` — Buyer collected unit and sent no post-sale signal within 5 days
- `PRICE_OBJECTION` — Buyer attempted price negotiation; may indicate value perception gap

---

## Signals Not Captured in This Operation (Gaps)

These are health signals that existed in the raw data but were not structured or tracked:

- **Response time per interaction.** The sub-30-minute target was maintained informally. Actual response time was never recorded. In a scaled system, response time per case should be a health input — delayed responses are an early disengagement signal.

- **Multi-channel activity.** Some buyers communicated via the marketplace platform and also via WhatsApp. Cross-channel context was tracked mentally but not unified in the record. Health scoring should account for all engagement points, not just formally logged cases.

- **Question complexity as a purchase probability signal.** Buyers who asked detailed technical questions (HEPA specs, energy consumption calculations) had higher conversion rates than buyers who asked only about price. This pattern is visible in the case log but was not formalised as a health or lead-quality signal.

---

## Health Records: Current State

| Record | Status | Satisfaction | Follow-Up | Notes |
|---|---|---|---|---|
| CH-0000 | Green | 5 | Completed | First sale, smooth collection |
| CH-0001 | Green | 5 | Completed | Repeat enquiry converted |
| CH-0002 | Green | 5 | Completed | — |
| CH-0003 | Amber | 3 | Completed | Slow initial response noted |
| CH-0004 | Green | 5 | Completed | Allergen question → purchase |
| CH-0005 | Green | 5 | Completed | — |
| CH-0006 | Amber | null | Not Completed | Post-sale silent; no satisfaction signal captured |

---

## Health Model in a SaaS Context

The logic above is structurally identical to how enterprise SaaS companies score account health:

- **Satisfaction Score** → NPS or CSAT survey response
- **Follow-Up Completed** → QBR completed, onboarding milestone reached
- **Response Time** → Time-to-first-response on support tickets
- **Risk Indicators** → Usage drop signals, missed check-ins, support escalation flags
- **Health Status** → Account health score feeding CSM dashboard and renewal risk queue

The scale differs. The operational logic is the same.
