# System Design — GlanAir Customer Operations

## Overview

This document describes the data model and object relationships underpinning the GlanAir operations system. The model was designed to give a complete, unified view of each customer's lifecycle — from first enquiry to post-sale health — in a single connected record structure.

---

## Object Model

### Account

The top-level entity representing GlanAir Home Solutions as the vendor organisation.

Custom fields added beyond standard:
- `Customer Priority` — picklist (High / Medium / Low)
- `SLA` — picklist (Standard / Premium)
- `SLA Expiration Date` — date
- `SLA Serial Number` — text
- `Number of Locations` — number
- `Upsell Opportunity` — checkbox
- `Active` — checkbox

In a multi-product expansion, each buyer organisation would be modelled as a separate Account. In the pilot, this single Account serves as the parent for all contacts and opportunities.

### Contact (15 records)

Each Contact represents an individual buyer who engaged meaningfully with the listing. Contacts are linked to the GlanAir Home Solutions Account and carry their own opportunity, case, and health record history.

Key fields used:
- Name (marketplace username where real name unknown)
- Account Name
- Title (marketplace handle, used as a unique identifier)
- Last Activity date (surfaced in the "No Activity" pipeline view)

### Opportunity (20 records)

Each Opportunity represents a single unit sale or lost sale event.

- 15 Closed Won — sales completed between 19 Feb and 22 Feb 2026
- 5 Closed Lost — buyers who were active in pipeline when stock reached zero on 19 March 2026

Fields:
- `Opportunity Name` — format: "GlanAir Dehumidifier — [buyer handle]"
- `Stage` — Needs Analysis → Closed Won / Closed Lost
- `Close Date` — date of transaction or stock-out
- `Amount` — €39

The Lead-to-Opportunity automation creates a new Opportunity record in Needs Analysis stage whenever a new Contact is created, encoding the assumption that every inbound enquirer is a potential buyer until qualified otherwise.

### Case (30+ records)

Cases are the primary support record. Every significant customer interaction is modelled as a Case.

Case categories observed:
- Pre-sale product questions (product specs, allergen, energy)
- Logistics (collection hours, address, availability)
- Pricing (cost, payment methods)
- Stock availability (units remaining, waitlist)
- Post-sale follow-up (satisfaction, collection confirmation)
- Warranty enquiries

Fields:
- `Subject` — descriptive label matching the enquiry
- `Status` — New / Working / Escalated / Closed
- `Priority` — Low / Medium / High / Critical
- `Date/Time Opened`
- `SLA Due Date` — set automatically by the SLA Monitor flow

### Customer Health Record (7 records, custom object)

A custom object created post-sale for buyers who completed a transaction and received follow-up.

Records: CH-0000 through CH-0006

Fields:
- `Health Record ID` — auto-number (CH-XXXX)
- `Health Status` — Green / Amber / Red
- `Satisfaction Score` — number (1–5)
- `Last Follow-Up Date` — date
- `Follow-Up Completed` — checkbox
- `Feedback Notes` — long text
- `Response Time (mins)` — number
- `Risk Indicators` — text

### Inventory Log (custom object, 1 record)

Single record tracking the pilot batch inventory state.

- `Log ID` — INV-0000
- `Units In` — 20
- `Units Sold` — 20
- `Units Remaining` — 0
- `Restock Alert Triggered` — checkbox (checked)
- `Notes` — "GlanAir pilot batch. 20 units sold in 32 days at €39 each. Sold out 19/03/2026 with 10+ buyers still in active pipeline. No restock — pilot ended."

### FAQ Knowledge Records (15 records, custom object)

A knowledge base of 15 frequently asked questions categorised by domain.

Categories:
- Product Specifications (room size, noise, weight, power, timer)
- Health & Allergens (asthma, dust, pet odours, cigarette smoke)
- Pricing (cost, filter included, running cost)
- Logistics (collection hours, evening availability)
- Warranty & After Sales (warranty duration, filter replacement indicator)

---

## Relationship Diagram

```
GlanAir Home Solutions (Account)
    │
    ├── Contact: Aoife Murphy
    │     ├── Opportunity: Closed Won — 19/02/2026
    │     ├── Case: Allergens & Dust (pre-sale)
    │     └── Customer Health Record: CH-0004
    │
    ├── Contact: Darragh_92
    │     ├── Opportunity: Closed Won — 20/02/2026
    │     └── Case: Evening Collection (pre-sale)
    │
    ├── Contact: Buyer16
    │     ├── Opportunity: Closed Lost — 19/03/2026 (no stock)
    │     └── Case: Stock Check (post sell-out)
    │
    └── [12 additional contacts...]

Inventory Log: INV-0000
    └── Restock Alert → Task created (high priority)

FAQ Knowledge Records (15 entries)
    └── Feeds Agentforce Support Agent
```

---

## Design Decisions

**Why a custom Customer Health object rather than a field on Contact?**
Health state changes over time. A custom object allows multiple health snapshots per contact and supports historical trend analysis. A field on Contact only captures current state.

**Why model Closed Lost opportunities?**
The 5 stock-out losses are operationally significant — they represent confirmed demand. Modelling them as Closed Lost (rather than deleting the opportunity) preserves the signal for future planning and makes the demand overflow visible in reports.

**Why 7 health records rather than 20 (one per buyer)?**
Health records were created for buyers who received structured post-sale follow-up. Not all 20 buyers required active follow-up — some transactions were low-complexity and self-resolving. In a scaled system, health records would be created automatically for all closed-won accounts.
