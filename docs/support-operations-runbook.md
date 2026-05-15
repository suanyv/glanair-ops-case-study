# Support Operations Runbook — GlanAir

## Purpose

This document describes the support handling procedures for the GlanAir customer operations system. It is written as a runbook — a reference for consistent case handling, escalation decisions, and SLA compliance.

---

## Case Intake

All inbound customer enquiries are logged as Cases linked to the relevant Contact record. If no Contact exists for the enquirer, one is created before the Case is opened.

**Required fields on case creation:**
- Subject (descriptive, following the format: `[Category] — [Brief Description]`)
- Status: New
- Priority: Medium (default; adjust based on classification below)
- Contact name

The SLA Response Time Monitor flow fires automatically on case creation, setting SLA Due Date to 30 minutes from creation time.

---

## Case Priority Classification

| Priority | Criteria |
|---|---|
| Critical | Fault or safety concern reported; buyer unable to collect confirmed purchase |
| High | SLA breached; stock-out buyer with no resolution; post-sale complaint |
| Medium | Pre-sale product question; logistics coordination; standard warranty enquiry |
| Low | General interest; browsing query with no clear purchase intent |

---

## Case Categories and Handling

### Product Specifications

Questions about room suitability, noise levels, weight, power consumption, air quality display, timer function, remote control.

**Handling:** Check FAQ Knowledge Records first. If a matching FAQ exists, apply the canonical answer and close the case. If no match exists, draft a response and flag the question for FAQ addition.

**Target resolution time:** < 30 minutes (first response); same session for close.

---

### Health & Allergens

Questions about asthma suitability, dust and pollen capture, pet odours, cigarette smoke.

**Handling:** These questions carry implicit health sensitivity. Responses should be accurate and cite the product specification (HEPA filter, 99.97% dust/pollen capture) without making medical claims beyond documented product specifications. Route to Allergen and Health Claims subagent if Agentforce is active.

**Target resolution time:** < 30 minutes.

---

### Pricing & Offers

Questions about total cost, what is included in the price, payment methods, negotiation.

**Handling:** Standard response: €39, filter included, 2-year warranty included, cash or Revolut on collection. Do not negotiate below listed price without explicit authorisation.

**Target resolution time:** < 15 minutes (straightforward factual response).

---

### Collection & Logistics

Questions about collection hours, address, evening availability, WhatsApp coordination.

**Handling:** Confirm availability before providing address. Evening collections available after 6 PM on weekdays from Dublin 8. Confirm specific time via WhatsApp before buyer travels.

**Target resolution time:** < 30 minutes for initial response; coordination may extend to same-day close.

---

### Stock Availability

"Is it still available?" / "Do you have more than one?"

**Handling:** If units remain, confirm availability and move to qualification. If stock is at zero, log as Closed Lost on the associated Opportunity, update the case with stock-out notification language, and document the buyer as unmet demand in the Inventory Log notes.

**Target resolution time:** < 15 minutes.

---

### Post-Sale Follow-Up

Satisfaction check, collection confirmation, fault reports.

**Handling:** Post-sale cases are created automatically by the Post-Sale Health Trigger flow 3 days after close. Update the linked Customer Health Record with satisfaction score and feedback notes. If a fault or complaint is logged, escalate to High priority and create a follow-up task within 24 hours.

**Target resolution time:** First contact within 3 days; resolution within 5 days.

---

## SLA Monitoring

The SLA Response Time Monitor flow runs on all cases. At T+30 minutes from creation:

1. System checks if case status is still New or Working.
2. If open: creates a high-priority SLA Breach Alert task assigned to the case owner.
3. If closed: no action.

**Escalation on breach:** If a case has breached SLA and the SLA Breach Alert task has not been actioned within 1 hour, escalate case priority to High and reassign to the next available owner.

---

## Escalation Path

```
Standard Case
    │
    ├── Resolved within SLA → Close case, update health record if post-sale
    │
    ├── SLA Breach Alert generated → Action within 60 minutes
    │       │
    │       └── Not actioned → Priority bump to High, reassign
    │
    └── Fault / Complaint → Immediate escalation to Critical
            │
            └── Resolution plan within 24 hours
                    │
                    └── If unresolved at 48 hours → Management review
```

---

## FAQ Knowledge Base Management

FAQ records should be reviewed and updated under two conditions:

1. A new question type appears in the case queue that does not match any existing FAQ record. Add the question and canonical answer within 24 hours of first appearance.
2. A product specification changes (e.g. new SKU, revised warranty terms). Update all affected FAQ records before any cases can be opened against the new specification.

**FAQ gap detection:** When the Agentforce FAQ Routing flow creates a Manual Review Task (no FAQ match found), the case subject should be reviewed as a potential FAQ addition candidate.

---

## Demand Overflow Protocol

When all inventory units are sold:

1. Update Inventory Log: Units Remaining = 0, Restock Alert Triggered = true.
2. All subsequent enquiries for the sold-out product: open case, log as stock-out notification.
3. Create Opportunity with stage Closed Lost and close date = today. Add note: "Closed Lost — No Stock."
4. Update Inventory Log notes with count of unresolved active buyers.
5. If restock is planned: create a task to notify all Closed Lost contacts by restock date.
