# Automation Flows — GlanAir

Five record-triggered flows are active in the GlanAir system. This document describes the trigger condition, logic, and operational purpose of each.

---

## Flow 1: Lead to Opportunity Conversion

**File:** `Flow1_Active` (GlanAir — Lead to Opportunity Conversion — V1)  
**Type:** Record-Triggered  
**Trigger:** New Contact created  
**Status:** Active

**Logic:**
```
Contact created
    └── Run Immediately
            └── Create Linked Opportunity
                    ├── Name: [OpportunityName variable]
                    ├── Stage: Needs Analysis
                    ├── Close Date: Today + 14 days
                    ├── Amount: €39
                    └── Lead Source: [LeadSourceOther variable]
```

**Purpose:** Every inbound enquirer who becomes a Contact record should have an associated Opportunity in Needs Analysis. This ensures the full pipeline is visible — not just closed deals. Without this, the view of active buyer interest is incomplete.

**Operational impact:** Surfaces the full enquiry pipeline. Enables pipeline-stage filtering to distinguish active prospects from converted buyers from lost opportunities.

---

## Flow 2: Post-Sale Customer Health Trigger

**File:** `Flow2_PostSaleHealth_Active` (GlanAir — Post Sale Customer Health Trigger — V1)  
**Type:** Record-Triggered  
**Trigger:** Opportunity updated → Stage = Closed Won  
**Status:** Active

**Logic:**
```
Opportunity → Closed Won
    └── Run Immediately
            ├── Create Customer Health Record
            │       ├── Health Status: Green (default)
            │       ├── Satisfaction Score: null (pending follow-up)
            │       └── Linked to: Contact on Opportunity
            │
            └── Create 3-Day Follow-Up Task
                    ├── Subject: "Post-Sale Follow Up — Check customer satisfaction"
                    ├── Status: Not Started
                    ├── Priority: High
                    └── Due Date: Today + 3 days
```

**Purpose:** Ensures no buyer exits the purchase flow without a structured follow-up touchpoint. The 3-day window is based on the assumption that most buyers collect within 48 hours; the follow-up lands after collection but before the return window closes.

**Operational impact:** Creates the health record baseline and the follow-up obligation automatically. Removes reliance on memory for post-sale engagement.

---

## Flow 3: Inventory Threshold Alert

**File:** `Flow3_InventoryAlert_Active` (GlanAir — Inventory Threshold Alert — V1)  
**Type:** Record-Triggered  
**Trigger:** Inventory Log record updated, Units Remaining ≤ threshold AND Restock Alert Triggered = false  
**Status:** Active

**Logic:**
```
Inventory Log updated
    └── Run Immediately
            ├── Update Record: Restock Alert Triggered = true
            └── Create Restock Alert Task
                    ├── Subject: "RESTOCK ALERT — Units Remaining at or below threshold"
                    ├── Status: Not Started
                    ├── Priority: High
                    └── Due Date: Today
```

**Purpose:** Generates a visible, actionable alert when stock drops to a critical level. In the pilot, this would have fired with approximately 3–5 units remaining, giving time to assess demand overflow and prepare communications for waiting buyers.

**Operational impact:** Converts an invisible operational state (stock level) into an actionable signal. Prevents the sell-out scenario from being discovered reactively.

---

## Flow 4: SLA Response Time Monitor

**File:** `Flow4_SLAMonitor_Active` (GlanAir SLA Response Time Monitor — V1)  
**Type:** Record-Triggered with Scheduled Paths  
**Trigger:** Case created  
**Status:** Active

**Logic:**
```
Case created
    ├── Run Immediately
    │       └── Set SLA Due Date = Created Date + 30 minutes
    │
    └── Scheduled Path: 30 Minute SLA Check (T+30 from Case Created Date)
            └── Decision: Is Case Still Open?
                    ├── Yes (SLA Breached)
                    │       └── Create SLA Breach Alert Task
                    │               ├── Priority: High
                    │               └── Due Date: Today
                    └── No → End
```

**Purpose:** Enforces the sub-30-minute response target formally. Converts an aspirational target into a monitored commitment with a visible breach signal.

**Operational impact:** At scale, SLA breach rate becomes a reportable metric. Breach patterns surface staffing gaps, peak-load periods, and case category bottlenecks that would otherwise be invisible.

---

## Flow 5: Agentforce FAQ Routing

**File:** `Flow5_FAQRouting_Active` (GlanAir Agentforce FAQ Routing — V1)  
**Type:** Record-Triggered  
**Trigger:** Case created  
**Status:** Active

**Logic:**
```
Case created
    └── Decision: FAQ Match Found?
            ├── Yes → Route to Agent
            │       └── Update Record: Tag Case for Agent handling
            │
            └── No → Route to Human
                    └── Create Manual Review Task
                            └── Flag for human queue and FAQ gap review
```

**Purpose:** Implements the first-pass triage layer for the support queue. Known questions go to the AI agent with the appropriate knowledge context. Novel questions go to the human queue and generate a signal for knowledge base expansion.

**Operational impact:** As the FAQ base grows, agent-handled percentage increases and human queue volume decreases. The FAQ gap signal is a continuous improvement mechanism — every manual review task is a knowledge base improvement opportunity.

---

## Flow Interaction

The five flows interact across the customer lifecycle:

```
Contact Created → Flow 1 → Opportunity created (Needs Analysis)
                              │
                              └── Opportunity → Closed Won → Flow 2 → Health Record + Follow-Up Task
                                                                │
                                                                └── [Follow-Up Case opened] → Flow 4 → SLA Monitor
                                                                                               Flow 5 → FAQ Routing

Inventory Log updated → Flow 3 → Restock Alert Task

Any Case opened → Flow 4 → SLA Monitor
                  Flow 5 → FAQ Routing
```

No flow depends on another completing first. Each runs independently on its trigger condition, which avoids cascading failures and simplifies debugging.
