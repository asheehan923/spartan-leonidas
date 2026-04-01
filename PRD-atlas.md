# Atlas - Product Requirements Document

**Client:** Spartan Energy Services (Delmar, CEO)
**Author:** Alex Sheehan, VectisOS
**Date:** 2026-03-31
**Status:** Discovery / Draft
**Source:** Impromptu Google Meet, March 31 2026 ([Recording](https://fathom.video/share/TusB8eU4sR1FJ4sgHf7zebx-Y9En1qAN))

---

## Executive Summary

Spartan Energy Services is a $70M/year oilfield services company running at ~87% equipment utilization across 30+ simultaneous jobs. The CEO currently spends roughly a third of his time manually managing a sprawling Excel spreadsheet that tracks valve inventory, job assignments, consignment equipment from multiple partners (~$7M in third-party assets), and sub-rentals (~$200K/month). The spreadsheet has grown organically since October 2024 and is the single source of truth for the entire company's asset deployment, but it cannot:

- Track asset **location** (geo-trackers fail ~35% of the time)
- Distinguish utilization across **ownership classes** (owned vs. consignment vs. sub-rental)
- Provide **district-level** visibility (Keithville LA, Midland TX, Pleasanton TX)
- **Forecast** beyond a ~2-week window with any reliability
- Integrate with **KPA** (their quality/certification management system)
- Generate **loadout sheets** or quality packs tied to job planning
- Track **sub-assemblies** (frac stacks built from individual components)

The result: the CEO and President spend hours daily robbing Peter to pay Paul -- converting valve sizes, shuffling assets between districts, and manually reconciling consignment reports. Logistics costs run $580K/year, inflated by poor asset visibility and last-minute hot shots.

**The goal is to build a custom asset management and job planning system that:**

1. **Eliminates the Excel bottleneck** -- give Delmar and his operations team a purpose-built tool that replaces the manual spreadsheet with structured data entry and automated roll-ups
2. **Tracks assets by ownership class** -- clearly separate owned, consignment (per partner), and sub-rented equipment with automated monthly consignment reporting
3. **Provides real-time inventory by district** -- answer "how many 7-1/16" 15K hydraulic valves are available in Midland right now?" instantly
4. **Enables 4-week rolling job forecasts** -- structured job/work order entry with equipment requirements that auto-calculate future availability and flag shortfalls
5. **Calculates utilization by asset type and ownership** -- replace the manual utilization formulas with live dashboards showing company-wide and per-district utilization
6. **Reduces CEO time on asset planning by 50%+** -- move from a one-man bottleneck to a system district managers can interact with directly
7. **Mobile check-in/check-out** -- a field-facing mobile app that creates verified loadout tickets when equipment ships and receiving tickets when it returns, replacing handwritten delivery tickets and eliminating the 4-day data entry lag
8. **Smart serial number verification** -- if the quantity leaving matches the quantity returning, no serial-level audit needed; if there's a discrepancy, the app forces serial number verification to identify what's missing
9. **Sub-assembly management** -- build, deploy, and track frac stacks and other assemblies composed of individual serialized components, with mixed ownership support

This is not an AI problem -- it's a data structure and workflow problem. AI accelerates the *development* of the solution, reducing the cost to build custom software by ~95% compared to two years ago.

---

## 1. Background & Context

### 1.1 Company Profile
- **Company:** Spartan Energy Services
- **Revenue:** ~$70M/year (~$16M/month run rate)
- **Growth:** Doubled in size in the last ~6 months
- **Operations:** ~30 simultaneous jobs at any given time
- **Districts:** Keithville, Louisiana | Midland, Texas | Pleasanton, Texas
- **Key People:** Delmar (CEO, asset planning), Don (President, sales), Gabriel (operations), quality manager (KPA admin)
- **Existing Systems:** Excel (asset planning), KPA (quality/certs), Basis (supposed to track everything but underutilized), geo-trackers (unreliable)

### 1.2 Current State (Excel Spreadsheet)
The spreadsheet is organized as:
- **Rows:** One per active job (customer + well site)
- **Columns:** Equipment types (valve size/type/actuation combinations)
- **Cells:** Quantity needed per job, with color coding:
  - Checkered = sub-rented equipment
  - Pink = consignment equipment
  - Green = owned equipment
  - Black = previously impossible allocations that were solved
  - Red = utilization over 75%
- **Bottom section:** Auto-summing formulas showing total deployed, total owned, available, utilization %, and revenue estimates
- **Update frequency:** Every 2 days (too large for daily updates)
- **Forecast horizon:** ~2 weeks reliable, beyond that is speculative

### 1.3 Equipment Taxonomy (from transcript + screenshots)
Primary tracked assets are **serviceless valves** (hydraulic actuated, ~200% more expensive than standard but faster to maintain), plus spools, tees, crosses, and other frac iron. Key dimensions:

| Dimension | Values Observed |
|-----------|----------------|
| **Type** | Wing Valve, Master Valve (Serviceless), Zipper Valve, Rotating Spool, Spacer Spool, Studded Tee, Cross, Accumulator, Javelin |
| **Size** | 3", 4", 5", 7-1/16" |
| **Pressure Rating** | 10K, 15K |
| **Actuation** | Hydraulic (HYD), Manual (Man) |
| **Brand/Variant** | Best Way, Maverick (MAVI/MV), SE, Encompass, Crescent 50, State Ranger, Blue May, State Queen C3, State Project, Mark 5 |

Each combination (e.g., "Wing, Hydraulic, 4" 15K, Best Way") is a distinct inventory line.

#### Confirmed Equipment Types (from screenshots)

**Individual Assets (Components):**

| Spreadsheet Name | Type | Size | Pressure | Actuation | Brand |
|-----------------|------|------|----------|-----------|-------|
| Wings Hyd 4 15 | Wing Valve | 4" | 15K | Hydraulic | -- |
| Wings Man 4 15 | Wing Valve | 4" | 15K | Manual | -- |
| 5 15 HYD wing | Wing Valve | 5" | 15K | Hydraulic | -- |
| 5 15 Man wing | Wing Valve | 5" | 15K | Manual | -- |
| 5 15 HYD | Hydraulic Valve | 5" | 15K | Hydraulic | -- |
| 7 15 SERVICE LESS MASTERS | Master Valve (Serviceless) | 7" | 15K | -- | -- |
| 7 15 Encompass MASTERS | Master Valve | 7" | 15K | -- | Encompass |
| 7 15 Crescent 50 #3 MASTERS | Master Valve | 7" | 15K | -- | Crescent 50 #3 |
| 7 15 State Ranger MASTERS | Master Valve | 7" | 15K | -- | State Ranger |
| 7 15 Blue May MASTERS | Master Valve | 7" | 15K | -- | Blue May |
| 7 15 state queen c3 MASTERS | Master Valve | 7" | 15K | -- | State Queen C3 |
| 7 15 State project MASTERS | Master Valve | 7" | 15K | -- | State Project |
| 7 10 ZIPPERS | Zipper Valve | 7" | 10K | -- | -- |
| 7 15 ZIPPERS | Zipper Valve | 7" | 15K | -- | -- |
| 7 15 ZIPPERS MV Hyd Valves | Zipper Valve | 7" | 15K | Hydraulic | Maverick |
| 7 15 ZIPPERS SE Hyd Valves | Zipper Valve | 7" | 15K | Hydraulic | SE |
| 7 15 ZIPPERS SE ZIPPER | Zipper Valve | 7" | 15K | -- | SE |
| 7 15 Hyd Valves SE | Hydraulic Valve | 7" | 15K | Hydraulic | SE |
| 7 1/16 15M rotating spool | Rotating Spool | 7-1/16" | 15K | -- | -- |
| 7 1/16 X 10 spacer spool | Spacer Spool | 7-1/16" | 10K | -- | -- |
| 7" 15M studded tee | Studded Tee | 7" | 15K | -- | -- |
| 5" 15M X 3 1/16 15M cross | Cross | 5" x 3-1/16" | 15K | -- | -- |
| JAVELIN | Javelin (zipper manifold) | -- | -- | -- | -- |

**Javelin:** A type of zipper manifold that uses a swing arm (similar to a cement truck). It replaces traditional zipper manifold technology. Tracked as a standalone asset, not a sub-assembly.

**Sub-Assemblies (combinations of components):**

| Spreadsheet Name | Components (likely) | Notes |
|-----------------|-------------------|-------|
| 7 10 STACK | 7" 10K frac stack assembly | Components TBD -- need assembly definition |
| 7 15 STACK | 7" 15K frac stack assembly | Components TBD -- need assembly definition |
| 715 HYD retain Stack & Zippers Out | Master valve + wing valves + zippers + spools + tees | Tracked as owned (white) and consignment (pink) separately; "Balance left in" tracks remainder |
| 5 15 STACK MAVI VALVES MARK 5 | Maverick Mark 5 valves assembled as a stack | Purple-coded in spreadsheet |

**Labor Tracking (in scope -- Phase 1):**
The spreadsheet also tracks "Spartan Hands needed" and "Contract Hands" per job. This is included in Phase 1 as F11 (Workforce Management). Priority: Spartan internal labor first, 3rd-party contract labor second.

> **Note:** Additional equipment types exist beyond what is shown in these screenshots. The taxonomy table above will be expanded as more data is provided. Screenshots are stored in `VectisOS/Projects/Spartan/` for reference.

### 1.4 Ownership Classes
| Class | Description | Volume | Tracking |
|-------|------------|--------|----------|
| **Owned** | Spartan-purchased assets | ~$14M total value | Serial numbers, asset numbers |
| **Consignment** | Partner-owned, Spartan-operated | ~$7M across 3+ partners | Separate Excel tabs per partner, monthly revenue reports |
| **Sub-rental** | Rented from third parties as needed | ~$200K/month | Ad hoc, not systematically tracked |

Consignment partners receive monthly reports showing: asset serial numbers, job assignments, days on job, rental rate, total revenue. One partner earns $77K/month; another $40K/month. Equity accrual: 25-33 cents per rental dollar toward future purchase.

### 1.5 Pain Points (Ranked)
1. **CEO time sink** -- 1/3 of CEO time on spreadsheet management
2. **No district-level visibility** -- can't tell a DM "you have 18 valves available in your yard"
3. **No asset location tracking** -- geo-trackers only 65-70% reliable; forklifts destroy them
4. **Consignment tracked separately** -- can't see blended utilization in one view
5. **Sub-rentals not tracked** -- no visibility into how much is being sub-rented or from whom
6. **Inconsistent job rows** -- spreadsheet grew organically; not every job tracks the same equipment columns
7. **No integration with KPA** -- quality certs and asset tracking are disconnected
8. **Delayed data entry** -- delivery tickets are handwritten, entered 4 days late into Basis
9. **Logistics costs** -- $580K/year in trucking, inflated by poor asset visibility
10. **No loadout/quality pack generation** -- only ~85% of customers get quality packs when busy

---

## 2. User Flows (Detected from Transcript)

These are the workflows Delmar described during the call, documented as current-state ("as-is") flows and the target future-state ("to-be") flows the system should enable.

### UF1: Daily/Bi-Daily Asset Planning (CEO + President)

The core operational heartbeat of the company. Every 2 days, the CEO and President review all active and upcoming jobs, identify equipment shortfalls, and develop a plan to cover them. This is the workflow that consumes a third of the CEO's time and is the primary target for automation.

**Current State:**
1. Delmar opens the master Excel spreadsheet (updated every 2 days)
2. Scrolls through 30+ job rows reviewing equipment allocations
3. Checks the bottom summary section for total deployed, total owned, available, and utilization %
4. Identifies equipment types where utilization is at or above 100% (red/negative availability)
5. For each shortfall, manually searches job rows for upcoming end dates ("who's coming in with some freedom?")
6. Evaluates options: convert valve sizes (e.g., 4" → 3"), pull from consignment, arrange sub-rental
7. Calls DMs to communicate changes ("you need to convert this to three-inch wings")
8. Updates spreadsheet cells manually with new allocations
9. Process repeats every 2 days; takes several hours each session with the President

**Future State:**
1. CEO opens the **Job Planning Board** (F2) → sees 4-week rolling forecast with auto-calculated availability
2. System highlights shortfalls in red with specific dates when equipment will go negative
3. CEO clicks a shortfall → system runs the **Shortage Resolution Waterfall** (UF3) and presents options in priority order: local owned → local consignment → local sub-rental → cross-district owned → cross-district consignment → cross-district sub-rental
4. CEO uses **"what-if" mode** to test size conversions (drag 4" → 3") and see the ripple effect across all jobs
5. Once a plan is set, CEO assigns equipment (F3) following the waterfall priority → system updates all dashboards in real time
6. DMs receive notifications of changes in their district view (F6)
7. Total planning time drops from hours to minutes for routine adjustments

---

### UF2: New Job Setup & Equipment Forecasting

When a new job is won or a frac schedule is received, the equipment requirements must be entered into the system and checked against available assets. This flow determines whether Spartan can take the work and what it will cost to resource it.

**Current State:**
1. Sales/operations receives a frac schedule from a customer (e.g., a 6-well pad)
2. Delmar determines what equipment is needed: type, size, pressure, quantity per well
3. Adds a new row to the spreadsheet with the customer name and well site
4. Fills in quantities per equipment column
5. Scrolls to the bottom to see if the new job pushes any equipment type into negative availability
6. If negative, begins the shortage resolution flow (UF3)
7. Jobs beyond ~2 weeks are entered speculatively ("the future don't look good... I'm going to be short six right here")

**Future State:**
1. Sales, DM or CEO creates a new **Work Order** (F2) with customer, well site, district, start/end dates
2. Selects equipment requirements from a standardized equipment type picker (not free-text columns)
3. System immediately shows impact on availability: "adding this job makes 7-1/16" 15K hydraulic wings go to -6 on April 15"
4. If shortfall detected, system automatically runs the **Shortage Resolution Waterfall** (UF3) and presents sourcing options in priority order (local owned → local consignment → local sub-rental → cross-district owned → cross-district consignment → cross-district sub-rental)
5. CEO can accept the suggested sourcing, modify it, or defer the job
6. Approved jobs appear on the 4-week forecast for all users with appropriate access
7. Equipment requirements can specify individual components or sub-assemblies (e.g., "2x 715 HYD Frac Stack")

---

### UF3: Shortage Resolution ("Robbing Peter to Pay Paul")

When demand exceeds supply for a specific equipment type, the CEO must find a way to fill the gap. This is the most complex and time-consuming workflow -- it requires evaluating job timelines, size conversions, consignment availability, cross-district transfers, and sub-rentals. The Shortage Resolution Waterfall defines the strict priority order the system follows to resolve shortfalls.

**Current State:**
1. Delmar sees a negative number in the availability summary (e.g., "-6 on 7-1/16" 15K wing valves")
2. Scrolls through all job rows looking for jobs with end dates that release that equipment type before the shortfall date
3. If none found, evaluates size conversion: "can I convert this customer from 4" to 3" wings?"
4. Calls the customer to pitch the conversion ("tell them why three-inch is a better deal for them")
5. If conversion accepted, updates the spreadsheet -- the released 4" valves become available
6. If still short, checks consignment availability across partners (separate Excel tabs)
7. If still short, arranges a sub-rental from a third party
8. Updates the spreadsheet with the solution; previously "black" cells (impossible) may turn green (resolved)

**Future State:**
1. System proactively alerts: "7-1/16" 15K wing valves will be at -6 on April 15 at Midland"
2. System runs the **Shortage Resolution Waterfall** -- a strict priority order that exhausts all options at the local district before looking elsewhere:

**Shortage Resolution Waterfall (priority order):**

| Priority | Source | Location | Description |
|----------|--------|----------|-------------|
| **1** | Spartan-owned assets | Local district | Available owned assets at the district where the job is |
| **2** | Consignment assets | Local district | Available consignment partner assets at the same district |
| **3** | Sub-rental assets | Local district | Rent from a third party and deliver to the local district |
| **4** | Spartan-owned assets | Other district(s) | Transfer owned assets from another district (triggers F10 cross-district optimization) |
| **5** | Consignment assets | Other district(s) | Transfer consignment assets from another district |
| **6** | Sub-rental assets | Other district(s) | Rent from a third party near another district and ship to the job |

At each priority level, the system checks whether the source district can spare the assets without creating a shortfall for its own upcoming jobs. If pulling from another district would cause a problem there, it flags the trade-off and moves to the next option.

3. Alert includes **auto-suggested resolutions** following the waterfall:
   - Priority 1: "4x owned wing valves available in Midland yard"
   - Priority 2: "Consignment partner MAV has 3 available in Midland"
   - Shortfall remaining: -6 + 4 + 3 = still need 1 more
   - Priority 4: "Keithville has 8 owned wing valves surplus through April 20 (Job #1042 ends April 12). Transfer 1x from Keithville → Midland (~570 mi, ~8.5 hr, est. $X scheduled / $Y hot shot)"
   - Alternative: "Converting Job #1038 from 4" to 3" would free 4 wing valves locally (Priority 1) -- eliminates need for cross-district transfer"
4. CEO or President selects a resolution → system updates all affected jobs and recalculates availability
5. If cross-district transfer is selected, system auto-generates transfer order (F9) and loadout ticket (F7)
6. If sub-rental is needed, system logs the sub-rental commitment and tracks cost

---

### UF4: Consignment Partner Onboarding & Management

Spartan manages ~$7M in equipment owned by consignment partners who earn monthly revenue in exchange for Spartan operating their assets. Each partnership has unique terms. This flow covers bringing a new partner's equipment into the system, tracking it alongside owned assets, and maintaining the trust-based relationship through accurate, transparent reporting.

**Current State:**
1. Delmar negotiates a deal with a competitor or partner: "give me your equipment, I'll manage it, send you mailbox money"
2. Partner delivers equipment → Delmar creates a new tab in Excel with all asset serial numbers
3. Equipment is logged with partner name, asset numbers, and agreed terms (rental rate, equity accrual %)
4. As equipment is deployed, Delmar manually notes which job it's on in both the main planning sheet (pink cells) and the consignment tab
5. At month end, Delmar compiles a report: asset serial number, job name, days on job, rate, total revenue
6. Sends the report to the partner along with a check
7. Equity accrual is tracked separately (25-33 cents per rental dollar toward future purchase)

**Example from transcript:** One partner gave Spartan ~$900K+ in hydraulic valves. Monthly revenue to partner: ~$77K. Another partner: ~$40K/month. A third partner contributed $2.8M in accumulators.

**Future State:**
1. New consignment partner is created in the system with contact info, terms (rate, equity %, standby rate)
2. Partner's assets are registered in the Equipment Registry with ownership_class = "consignment" and linked to the partner
3. When deployed, the system automatically tracks days on job per asset
4. At month end, CEO clicks **"Generate Consignment Report"** (F4) → system auto-generates the report matching the current Excel format
5. Equity accrual is auto-calculated and shown as a running total per partner
6. Both Delmar and the partner can see utilization of the consignment fleet in their respective views

---

### UF5: Equipment Location Tracking & Recovery

Knowing where assets are at any given moment is critical for planning and shortage resolution. Today this relies on unreliable geo-trackers (65-70% accuracy), KPA records, and phone calls. This flow replaces that detective work with a system of record based on check-in/check-out logs.

**Current State:**
1. Delmar needs to find specific assets (e.g., "where are my 7-1/16" 15K best way hydraulic valves?")
2. Checks geo-trackers → only 65-70% reliability ("forklifts are probably the number one thing that f--- them up")
3. If not found via tracker, checks KPA system (quality guy found some valves this way: "these valves are right here")
4. If still not found, calls DMs: "that last job it was on -- you should have it"
5. Old school detective work: "the last job it was on, you haven't used it since, so it ought to be..."
6. This morning's example: Delmar discovered consignment equipment was at a different location than expected, highlighted it 10 minutes before the call

**Future State:**
1. Every equipment movement is logged via check-in/check-out (F7)
2. System shows last known location for every asset: "Asset #MAV-1042 -- last checked in at Keithville yard on March 28"
3. CEO can query: "show me all 7-1/16" 15K hydraulics" → map/list view with district, status, and job assignment
4. Discrepancy alerts: "10 valves checked out to Job #1038 on March 15, but job ended March 25 and only 8 were checked back in"
5. Geo-trackers become supplementary (nice-to-have), not primary -- the system of record is the check-in/check-out log

---

### UF6: District Manager Equipment Communication

District managers currently have no independent visibility into their own assets -- they rely entirely on the CEO to tell them what's available and what to do. This flow gives DMs self-service access to their district's assets and a structured way to request what they need, reducing phone calls and giving the CEO back time.

**Current State:**
1. CEO determines from the spreadsheet what equipment is available company-wide
2. CEO calls the DM: "you need to do this -- that can't be 4-inch wings, it's got to be 3-inch wings"
3. DM may push back: "I've got all my flow crosses set up already"
4. CEO goes back to the spreadsheet to find another solution
5. DM has no independent view of what's available in their district -- they rely entirely on the CEO's instructions

**Future State:**
1. DM opens the **District Manager View** (F6) → sees all assets in their district by type, status, and ownership class (owned, consignment, sub-rental)
2. DM can answer their own questions: "I have 18 seven-inch 15K hydraulics available in my yard -- 12 owned, 4 consignment (MAV), 2 consignment (3E)"
3. When a new job comes in, DM enters equipment needs via a standardized form
4. System checks local availability following the waterfall priority (owned → consignment → sub-rental at this district first)
5. If local assets are insufficient, system flags the gap and suggests cross-district sourcing per the waterfall (owned at other districts → consignment at other districts → sub-rental at other districts)
6. DM submits a **sourcing request** with the system's recommendation → CEO or President reviews and approves from the planning board
7. Communication shifts from phone calls to system-mediated requests with full context

---

### UF7: Loadout -- Shipping Equipment to a Job Site

When equipment ships from a yard to a job site, a verified record must be created of exactly what was sent. Today this is a handwritten delivery ticket entered into Basis 4 days late. This flow replaces that with a mobile-verified loadout ticket that updates asset status in real time and is immediately visible to the receiving field crew.

**Current State:**
1. DM or operations decides what to send to a job site
2. Yard crew loads equipment onto trucks
3. Someone writes a handwritten delivery ticket listing what was sent
4. Delivery ticket is supposed to be entered into Basis (their ERP) but is typically delayed 4 days
5. Field drawings are created by a dedicated person showing the well site layout
6. A quality pack (pictures of accumulators, serial numbers, etc.) is assembled -- but only ~85% of customers get one when Spartan is busy

**Future State:**
1. Work order is approved (UF2) → system generates a **loadout ticket** (F7) listing all required equipment by type + quantity
2. Yard supervisor opens the mobile app → pulls up the loadout ticket for the job
3. Supervisor walks the yard verifying each line item: "10x 7-1/16" 15K wing valves -- check"
4. For each line, supervisor confirms the quantity. Serial numbers are optional at check-out (available on demand but not required)
5. A second person (verifier) opens the app and confirms the loadout
6. System marks the loadout as "verified" → equipment status changes to "in transit"
7. Loadout ticket is immediately visible to the field crew at the destination (F8)
8. Equipment arrives → field crew confirms receipt in the app
9. No handwritten tickets. No 4-day lag. Real-time visibility for everyone.

---

### UF8: Check-In -- Receiving Equipment Back from a Job

When equipment returns from a completed job, it must be verified against what was originally sent. This is where assets go "missing" today -- returns aren't logged promptly, and discrepancies aren't caught until someone needs the equipment and can't find it. The smart serial number verification (only required on quantity mismatch) keeps the process fast while catching losses.

**Current State:**
1. Job ends or equipment is released
2. Equipment arrives back at a yard (may be a different yard than it shipped from)
3. Yard crew unloads and stages the equipment
4. Someone is supposed to update the tracking systems but it's often delayed or incomplete
5. Equipment may sit in the yard "lost" until someone physically finds it or traces it back through KPA records

**Future State:**
1. Job ends → system shows which equipment should be returning (linked to the original loadout ticket)
2. Yard supervisor opens the mobile app → selects the job/loadout ticket
3. Supervisor enters returning quantities per equipment type:
   - **Quantity matches loadout:** Fast path -- no serial number verification needed. Equipment is marked "available" at this yard.
   - **Quantity is less than loadout:** App requires serial number drill-down for that equipment type. Supervisor must identify which specific assets returned (by serial number scan, manual entry, or selection from a list). Missing assets are flagged as "discrepancy."
4. Discrepancies are routed to the DM and operations team with details: "Loadout #2045 sent 10x wing valves to Job #1038. Only 9 returned. Missing: Asset #SPT-4821 (7-1/16" 15K HYD Wing, owned)."
5. Returned equipment is immediately available in the inventory for the receiving yard -- no waiting for someone to update a spreadsheet.

---

### UF9: Sub-Assembly Build & Deploy

Frac stacks and other assemblies are built from multiple individual components (valves, spools, tees, crosses) that ship and deploy as a single unit. Today, the composition of a stack lives in Delmar's head. This flow formalizes assembly definitions with visual cross-section diagrams, tracks which serialized assets are in each slot, and handles mixed ownership (owned + consignment components in one stack).

**Current State:**
1. Delmar tracks frac stacks (e.g., "715 HYD retain Stack & Zippers Out") as single line items in the spreadsheet
2. The individual components that make up a stack are not explicitly linked -- Delmar knows from experience what goes into a stack
3. When a stack comes back, if a component is damaged, it's manually tracked and replaced
4. Some stacks have mixed ownership (owned + consignment components in the same assembly)

**Future State:**
1. CEO or operations defines an **assembly template** (F3b): "a 7-15 HYD Frac Stack requires 1x master valve (7" 15K), 2x wing valves (7" 15K HYD), 1x rotating spool (7-1/16" 15K), 1x studded tee (7" 15K), 2x zipper valves (7" 15K)"
2. Yard supervisor opens the **Assembly Builder** and selects the template → the **cross-section diagram** is displayed with empty slots labeled by component type
3. Supervisor clicks on each slot in the cross-section to assign specific serialized assets. As assets are assigned, the slot fills in with the component image and is color-coded by ownership (green = owned, pink = consignment). System suggests assets following the waterfall priority: owned assets at this district first, then consignment assets at this district. Cross-district sourcing for individual components requires CEO/President approval.
4. System validates all slots are filled (visual: all slots in the cross-section are filled, no outlines remaining) → marks the assembly as "ready". Mixed ownership is supported and visually obvious from the color coding.
5. Assembly gets its own asset ID and appears in the inventory as a deployable unit with its cross-section diagram as the visual representation
6. Individual components show status "in assembly" and are no longer counted as individually available
7. On loadout tickets, the stack appears as a single line item (e.g., "1x 715 HYD Frac Stack") with the cross-section diagram expandable for detail
8. On check-in, if the assembly returns intact (quantity match), the entire stack is checked in at once
9. If a component was swapped in the field, the assembly is "broken down" in the system and components are reconciled individually
10. Consignment reporting still tracks individual components: "Partner MAV's valve #MAV-301 was deployed as part of Stack #STK-15 on Job #1038 for 22 days"

---

### UF10: Monthly Consignment Reporting

At month end, each consignment partner receives a report showing exactly which of their assets were deployed, on which jobs, for how many days, at what rate, and the total revenue owed. This is the foundation of Delmar's trust-based partnerships. The report format must match what partners already receive -- accuracy and transparency are non-negotiable.

**Current State:**
1. At month end, Delmar opens the consignment partner's Excel tab
2. Reviews each asset: which job was it on, how many days, what rate
3. Calculates total revenue per asset and totals for the partner
4. Generates a report (one per partner) and sends it with payment
5. Tracks equity accrual separately
6. Process is repeated for each partner (currently 3+ partners)

**Future State:**
1. CEO clicks "Generate Report" for a partner → system auto-generates the report
2. Report includes: asset serial number, asset type, job name, days deployed, rate, line total, grand total
3. Matches the format Delmar currently sends (partners are used to this format -- don't break it)
4. Standby days are calculated separately at the standby rate
5. Equity accrual is shown as a running balance: "Total earned toward purchase: $X of $Y total value"
6. Report can be exported as PDF or Excel and emailed directly to the partner

---

### UF11: Capital Planning & Equipment Procurement

When an equipment type is chronically at high utilization, Delmar must decide whether to buy more assets, arrange a new consignment deal, or continue sub-renting. This flow uses waterfall utilization data to show the true cost of being undercapitalized -- how often the system resorts to cross-district transfers and sub-rentals -- and frames procurement decisions in terms of which waterfall tier they strengthen.

**Current State:**
1. Delmar reviews utilization trends in the spreadsheet over time
2. Identifies equipment types that are chronically at 100%+ utilization
3. Decides whether to buy more equipment, arrange consignment, or continue sub-renting
4. Updates the spreadsheet's capital plan section (yellow/blue cells): "I bought three more of this"
5. Adjusts the "total owned" numbers to reflect new purchases

**Future State:**
1. System provides utilization trend reports (F5): "7-1/16" 15K HYD wing valves have been at 95%+ utilization for 3 consecutive months"
2. System shows how often the waterfall is reaching each priority level for this asset type:
   - "Last 90 days: 85% of demand filled at Priority 1 (local owned), 10% at Priority 2 (local consignment), 3% at Priority 4 (cross-district owned), 2% at Priority 3 (local sub-rental)"
   - "Cross-district transfers for this asset type cost an estimated $X over the last 90 days"
   - "Sub-rental spend on this asset type: $Y over the last 90 days"
3. System shows the cost comparison aligned to the waterfall tiers:
   - **Buy (move to Priority 1):** One-time purchase + maintenance cost. Eliminates reliance on Priority 2-6 for these assets.
   - **Arrange consignment (strengthen Priority 2):** Ongoing rate - equity accrual. Reduces cross-district transfers and sub-rentals.
   - **Continue sub-renting (Priority 3/6):** Spot rate per use. Highest per-unit cost but no capital commitment.
4. CEO makes a procurement decision → adds new assets to the registry at a specific district
5. System recalculates all utilization, availability forecasts, and waterfall recommendations with the new assets included

---

### UF12: Inter-District Equipment Transfer

Moving assets between Keithville, Midland, and Pleasanton to fill shortfalls that can't be resolved locally. Transfers are triggered when the Shortage Resolution Waterfall reaches Priority 4-6 -- all local options have been exhausted. Every transfer has a logistics cost ($580K/year total), so the system must justify the move and track the expense.

**Current State:**
1. CEO identifies that District A has surplus equipment and District B has a shortfall
2. CEO calls DM of District A: "send 5 wing valves to Midland"
3. Equipment is loaded and shipped (hot shot trucking if urgent -- contributes to $580K/year logistics cost)
4. Receiving DM's yard crew stages the equipment -- may or may not log receipt
5. CEO manually updates the spreadsheet to reflect the transfer
6. No systematic tracking of transfer frequency, cost, or patterns

**Future State:**
1. Waterfall reaches Priority 4+ → system identifies which districts have surplus of the needed asset type (owned first, then consignment, then sub-rental -- maintaining the waterfall within the source district too)
2. System recommends a source district, factoring in: that district's own upcoming job commitments, route distance/cost, and whether the transfer can arrive in time
3. CEO or President approves the transfer → system creates a **transfer order** specifying: from (Keithville) → to (Midland), equipment type, quantity, ownership class, trucking method
4. Loadout ticket (F7) is generated at the sending yard
5. Yard supervisor checks out the equipment via mobile app
6. Receiving yard checks in the equipment via mobile app
7. Transfer is logged with timestamp, origin, destination, ownership class, and cost (trucking estimate)
8. Over time, system can show transfer patterns to inform where to pre-position assets

---

### UF13: Multi-District Supply Optimization

The system-level orchestration that runs when the Shortage Resolution Waterfall reaches Priority 4-6. Instead of the CEO manually scrolling through job rows to figure out which district can spare equipment, the system evaluates all districts' current and forecasted availability, calculates logistics costs, and presents an optimized sourcing recommendation. Midland and Pleasanton are close enough (~330 mi) to often be interchangeable for the same jobs.

**Current State:**
1. A job comes in requiring equipment that exceeds the local district's inventory
2. Delmar manually scrolls the spreadsheet to figure out which district might have surplus
3. He checks whether pulling from another district would create a shortfall there
4. Calls are made, equipment is hot-shotted across districts ($580K/year in trucking)
5. No systematic evaluation of which sourcing district minimizes total disruption
6. For jobs in West Texas or South Texas (Midland, Pleasanton), either district could potentially service the job, but there's no visibility into which is the better source

**Future State:**

UF13 is the system-level orchestration that runs when the **Shortage Resolution Waterfall** (UF3) reaches Priority 4-6 (cross-district sourcing). The waterfall has already exhausted local options before this flow is triggered.

1. Work order is created with equipment requirements that exceed the local district's available assets (after local owned, local consignment, and local sub-rental have all been evaluated per UF3 Priorities 1-3)
2. System detects the remaining shortfall and runs the **Supply Optimization Engine** (F10):
   - **Step 1: Cross-district owned assets (Waterfall Priority 4)** -- For each shortfall line item, identify which other districts have surplus Spartan-owned assets *after accounting for their own upcoming jobs on the schedule*
   - **Step 2: Cross-district consignment assets (Waterfall Priority 5)** -- If owned assets from other districts don't fully cover the gap, check consignment assets at other districts
   - **Step 3: Cross-district sub-rental (Waterfall Priority 6)** -- If still short, identify sub-rental options near other districts
   - **Step 4: Consolidation vs. split sourcing** -- Evaluate whether it's better to source everything from one district (fewer trucks, simpler logistics) or split across districts (less disruption to each)
   - **Step 5: Midland/Pleasanton interchangeability** -- For jobs that either West Texas or South Texas could service (~330 mi / ~5 hr between them), evaluate which district is a better fit based on current and forecasted availability across all ownership classes
3. System presents a **sourcing recommendation** to the CEO/President, clearly showing which waterfall priority each line item comes from:
   - "Local (Priority 1-3): 4x owned + 3x consignment (MAV) at Midland = 7 filled locally"
   - "Cross-district (Priority 4): Source 2x owned wing valves from Keithville (they have 12 surplus through April 20). Keithville can cover their upcoming jobs with the remaining 10."
   - "Cross-district (Priority 5): Source 1x consignment (3E) wing valve from Pleasanton (3E has 4 idle at Pleasanton, no upcoming demand)."
   - "Alternative: Split differently -- all 3 from Keithville (single truck, lower total cost) vs. 2 Keithville + 1 Pleasanton (less impact on Keithville surplus)"
   - "Warning: Sourcing all 3 from Pleasanton leaves them at 0 surplus for Job #1055 starting April 18. Not recommended."
4. CEO or President reviews and approves/modifies the recommendation
5. Approved sourcing plan auto-generates transfer orders (F9) and loadout tickets (F7) for each sending district
6. System does NOT auto-execute -- all cross-district sourcing requires CEO or President confirmation

**Optimization Factors (weighted):**
- Waterfall priority: Always respect the ownership priority (owned → consignment → sub-rental) at each location tier
- Schedule impact: Does pulling from District B create a shortfall for District B's upcoming jobs?
- Logistics cost: Estimated trucking cost by route (Keithville↔Midland ~570mi, Keithville↔Pleasanton ~430mi, Midland↔Pleasanton ~330mi)
- Timing: Can the assets arrive before the job starts? (Lead time by route)
- Consolidation: Fewer trucks is generally better -- prefer sourcing from one district over splitting
- District workload: Avoid overloading one district's yard crew with too many simultaneous transfers

---

### UF14: Workforce / Labor Management

Every job requires hands -- Spartan employees and/or contract labor. Like equipment, labor follows a waterfall: Spartan internal first, then contract. With 30+ simultaneous jobs, knowing who is available and where is its own planning challenge. This flow tracks workforce allocation so the CEO can see labor utilization alongside equipment utilization.

**Current State:**
1. Delmar tracks "Spartan Hands needed" and "Contract Hands" per job in the spreadsheet
2. Internal Spartan employees are assigned first; contract labor fills the gaps
3. No systematic tracking of who is where, who is available, or how contract labor is sourced
4. When busy (30+ simultaneous jobs), labor allocation becomes another bottleneck

**Labor Assignment Waterfall (priority order):**

| Priority | Source | Location | Description |
|----------|--------|----------|-------------|
| **1** | Spartan employees | Local district | Internal hands based at the job's district |
| **2** | Spartan employees | Other district(s) | Internal hands transferred from another district (travel/per diem cost) |
| **3** | Contract labor | Local district | 3rd-party hands sourced locally |
| **4** | Contract labor | Other district(s) | 3rd-party hands sourced from another region |

**Future State:**
1. Each job/work order includes labor requirements: N total hands needed
2. System maintains a **workforce roster** (F11) of all Spartan employees and known contract labor providers
3. When a job is created, system runs the labor waterfall:
   - Priority 1: "3 Spartan employees available in Midland"
   - Priority 3: "1 more hand needed -- flag for contract labor sourcing in Midland"
   - "Job #1055 needs 4 hands. 3 filled internally (Priority 1), 1 contract hand needed (Priority 3)."
4. Spartan employees are assigned to jobs; contract labor gaps are flagged for the DM to source
5. CEO/President can see company-wide labor utilization: "85% of Spartan hands are deployed, 15 contract hands across 8 jobs"
6. Historical data shows labor cost trends and how often the waterfall reaches Priority 3-4 (informs hiring decisions: "if we hired 3 more hands in Midland, we'd eliminate 80% of contract labor spend there")

---

### UF15: Machine Shop / Maintenance Tracking

Assets in the machine shop are unavailable for deployment but today they're a black hole in the planning system -- Delmar knows the count is reduced but has no visibility into what's being repaired or when it will be ready. This flow makes the shop queue visible to the planning system, so expected completions can be factored into the waterfall and shop priorities can be informed by upcoming job demand.

**Current State:**
1. Delmar's spreadsheet shows "what's in the machine shop" as unavailable inventory
2. No visibility into what's being repaired, how long it's been there, or when it will be ready
3. Assets in the shop are effectively "lost" to the planning system -- they reduce the available count but there's no expected return date
4. KPA tracks some redress/test records but it's disconnected from the planning spreadsheet

**Future State:**
1. When an asset goes to the machine shop, a **maintenance order** is created (F12)
2. Order captures: asset ID, issue description, type (repair/redress/inspection/test), expected completion date
3. Asset status changes to "in shop" -- it's excluded from "available" counts but visible in a separate machine shop queue
4. Planning board (F2) factors in expected shop completions when running the waterfall: "5 wing valves currently in shop, 3 expected back by April 10 -- these are included in Priority 1 (local owned) availability projections for April 10+"
5. When the shop completes work, the maintenance order is closed → asset status returns to "available" → waterfall recalculates, potentially resolving shortfalls that previously required cross-district sourcing
6. Machine shop backlog is visible to the CEO: "12 assets in shop, oldest has been there 18 days, 4 expected back this week"
7. Shop priority insight: "Expediting 2 wing valves in the Midland shop (expected April 12) would eliminate the need for a cross-district transfer from Keithville for Job #1055"

---

## 3. Requirements

### 3.1 Core Data Model

#### Equipment Registry
- Asset ID (Spartan asset number)
- Serial number
- Equipment type (valve type, size, pressure, actuation, bore, brand/variant)
- Asset class: **component** | **sub-assembly**
- Ownership class (owned | consignment | sub-rental)
- Owner (if consignment: partner name/ID)
- Current status (deployed | available | in transit | in shop/maintenance | in assembly | decommissioned)
- Current location (district + yard/job site)
- Parent assembly ID (if this component is currently part of a sub-assembly)
- Acquisition date
- Acquisition cost
- Consignment terms (if applicable): rate, equity accrual %

#### Equipment Types (Catalog)
- Type ID
- Canonical name (e.g., "Wing Valve, 7-1/16", 15K, Hydraulic, Best Way")
- Type category (valve | spool | tee | cross | accumulator | javelin)
- Size
- Pressure rating
- Actuation type
- Brand/variant
- Display name (short form for mobile: "Wings Hyd 7 15 BW")
- **Component image:** Photo or technical illustration of the equipment type (used on detail pages, loadout tickets, and mobile app for visual identification)
- **Cross-section diagram:** Optional -- used when this component type appears as a slot in a sub-assembly, showing where it sits in the assembly

#### Sub-Assembly Definitions (Bill of Materials)
A sub-assembly is a reusable configuration of components that ships and deploys as a single unit (e.g., a frac stack).

- Assembly definition ID
- Assembly name (e.g., "715 HYD Frac Stack", "5 15 MAVI Stack Mark 5")
- Component slots: list of (equipment type + quantity required)
  - e.g., 1x 7" 15K Master Valve + 2x 7" 15K Wing Valve Hyd + 1x 7-1/16" 15K Rotating Spool + ...
- **Assembly cross-section diagram:** A visual representation (cross-section or exploded view) showing how components fit together in the assembly. Each slot in the diagram is labeled and interactive -- clicking a slot highlights the component type and, when building an instance, shows the assigned asset.
- Notes / configuration rules

#### Sub-Assembly Instances
A specific, built sub-assembly with actual assets assigned to each slot.

- Instance ID (treated as an asset itself -- gets its own asset number)
- Assembly definition ID (which template it follows)
- Component assignments: list of (slot → specific asset ID with serial number)
- Build date
- Built by (user)
- Current status (same as equipment registry: deployed | available | etc.)
- Ownership class (can be mixed -- some components owned, some consignment)
- Ownership note: if components come from different ownership classes, the assembly inherits the "most restrictive" class for billing purposes, but individual component ownership is preserved for consignment reporting

#### Jobs / Work Orders
- Job ID
- Customer name
- Well site / pad name
- District / region (primary district; system may suggest sourcing from other districts)
- Status (planned | active | completed)
- Start date, estimated end date
- Equipment requirements (list of equipment type + quantity needed; can include sub-assemblies)
- Equipment assignments (specific asset IDs / assembly instance IDs assigned)
- Frac schedule reference (if available; schedules arrive via email or phone)
- Labor requirements:
  - Spartan hands needed (count)
  - Contract hands needed (count)
  - Labor assignments (linked to workforce roster)
  - Priority: Spartan internal labor first, 3rd-party contract labor second

#### Workforce Roster
- Person ID
- Name
- Type: **Spartan** | **contract**
- If contract: company name, contact info
- Home district
- Current assignment (job ID or "available")
- Skills / certifications (future: link to KPA)
- Status (active | unavailable | on leave)

#### Machine Shop / Maintenance Orders
- Work order ID
- Asset ID (the equipment being repaired/maintained)
- Type: repair | redress | inspection | test
- Status (queued | in progress | complete | failed)
- Entered shop date
- Expected completion date
- Completed date
- Notes (what was found, what was done)
- Returned to inventory: yes/no (when complete, asset status changes from "in shop" to "available")

#### Loadout / Receiving Tickets
- Ticket ID
- Ticket type (loadout | receiving | transfer-out | transfer-in)
- Linked work order / job ID (or transfer order ID)
- District / yard of origin or destination
- Line items: equipment type + quantity (+ optional serial numbers)
- For sub-assemblies: assembly instance ID as a single line item, with component detail expandable
- Created by (user ID)
- Verified by (user ID -- must be different from creator for loadouts)
- Timestamp (created, verified)
- Status (draft | verified | discrepancy)
- Discrepancy notes (if receiving quantity != loadout quantity)
- Discrepancy resolution (resolved | escalated | written off)

#### Transfer Orders
- Transfer ID
- From district / yard
- To district / yard
- Equipment type + quantity (or specific asset IDs)
- Reason (job requirement | rebalancing | maintenance)
- Linked loadout ticket (sending) and receiving ticket (receiving)
- Trucking method (company truck | hot shot | scheduled freight)
- Estimated cost

#### Consignment Partners
- Partner ID
- Partner name
- Contact info
- Equipment on consignment (linked to equipment registry)
- Terms (unique per partner -- no two deals are the same):
  - Rental rate (per day or per month)
  - Equity accrual rate (cents per rental dollar) -- e.g., 25¢/dollar or 33¢/dollar
  - Standby rate (if different from active rate)
  - Contract start date
  - Special terms / notes
- Monthly report history (generated reports with date, total revenue, equity accrual)

#### Asset Movement Log
- Movement ID
- Asset ID
- From (district/yard/job + status)
- To (district/yard/job + status)
- Timestamp
- Triggered by (loadout ticket, receiving ticket, transfer, manual adjustment)
- User who logged the movement

### 3.2 Functional Requirements

#### F1: Equipment Inventory Dashboard
- Real-time view of all equipment by type, size, pressure, actuation
- Filter by: ownership class, district, status, brand
- Show: total owned, total deployed, available, in shop, in assembly, utilization %
- Color coding matching Delmar's mental model (red = >75% util, etc.)
- Drill-down from summary to individual assets with serial numbers and current location
- **Component detail page:** Shows the equipment type image/illustration, specs (size, pressure, actuation, brand), current status, location, ownership class, assignment history, and maintenance history
- **Sub-assembly detail page:** Shows the assembly cross-section diagram with each component slot labeled and color-coded by ownership class (owned vs. consignment). Clicking a slot shows the assigned asset's detail page.
- Separate views for individual components and sub-assemblies

#### F2: Job Planning Board
- Create/edit jobs with equipment requirements (supports both components and sub-assemblies)
- 4-week rolling forecast view showing availability by equipment type per day
- Auto-calculate future availability based on planned job start/end dates
- Flag shortfalls (negative availability) with alerts and suggested resolutions (UF3)
- Support "what-if" scenarios: "if I convert this job from 4" to 3", what happens?"
- Show job rows with customer, well site, district, dates, and equipment allocations (familiar layout for Delmar)
- Capital plan view: what equipment would need to be purchased to eliminate all shortfalls

#### F3: Equipment Assignment
- Assign specific assets (by asset ID) or sub-assembly instances to jobs
- Track which assets are on which job at any point in time
- Support partial assignments (job needs 10, 6 are owned, 4 are consignment)
- Log all movements (asset X moved from Job A to Job B on date Y)
- **Assignment priority (Shortage Resolution Waterfall):**
  1. Spartan-owned assets at the local district
  2. Consignment assets at the local district
  3. Sub-rental assets at the local district
  4. Spartan-owned assets from another district (triggers cross-district transfer)
  5. Consignment assets from another district
  6. Sub-rental assets from another district
- System always exhausts local options before suggesting cross-district transfers

#### F3b: Sub-Assembly Management
Sub-assemblies (e.g., frac stacks) are combinations of individual assets that ship, deploy, and return as a single unit.

**Assembly Builder:**
- Define reusable assembly templates (bill of materials): "a 715 HYD frac stack requires 1x master valve, 2x wing valves, 1x rotating spool, 1x studded tee, ..."
- **Visual cross-section view:** The builder displays the assembly's cross-section diagram with each component slot labeled. Empty slots are shown as outlines; filled slots show the component image and assigned asset ID. Ownership class is color-coded (e.g., green = owned, pink = consignment) matching Delmar's existing mental model from the spreadsheet.
- Build a specific instance of an assembly by clicking on slots in the cross-section to assign real assets (with serial numbers)
- System validates that all required slots are filled before marking the assembly as "ready"
- Mixed ownership: an assembly can contain both owned and consignment components -- system tracks each component's ownership independently

**Assembly Lifecycle:**
- **Build:** Select a template, assign components → assembly instance is created, component assets are marked as "in assembly"
- **Deploy:** Assign the assembly to a job (treated as a single line item on loadout tickets)
- **Return:** Check in the assembly as a unit; if all components return, fast path. If a component is swapped or missing, the assembly is "broken down" and components are reconciled individually
- **Reconfigure:** Swap out a component (e.g., replace a damaged valve) without breaking the whole assembly -- update the slot assignment, log the change

**Inventory Impact:**
- When a component is assigned to an assembly, it is no longer counted as individually "available" -- it's available *as part of the assembly*
- Assembly-level utilization rolls up from component utilization
- Consignment reporting still tracks individual components (partner X's valve was deployed as part of assembly Y on job Z for N days)

#### F4: Consignment Management
- Track consignment equipment within the same system as owned equipment
- Auto-generate monthly consignment reports per partner (matching current Excel format: serial number, job, days, rate, total)
- Calculate equity accrual per partner with running balance
- Show consignment utilization alongside owned utilization
- Support standby rate calculations (different rate when equipment is assigned but idle)
- Report export: PDF and Excel formats

#### F5: Utilization Reporting
- Company-wide utilization by equipment type
- District-level utilization
- Ownership-class utilization (owned vs. consignment vs. sub-rental)
- Historical utilization trends (weekly, monthly)
- Revenue estimates based on utilization x rental rates (current benchmark functionality from spreadsheet)
- Equipment type trending: flag types that have been >90% for 3+ consecutive periods (capital planning trigger)

#### F6: District Manager View
- Simplified view for DMs showing their district's assets by type, status, and ownership
- "What's available in my yard?" query with real-time counts
- Input form for job equipment needs (standardized equipment type picker -- not ad hoc columns)
- View incoming transfers and expected arrivals
- Loadout/receiving ticket history for the district

#### F7: Mobile Check-In / Check-Out (Mobile App)
A mobile application for yard supervisors, logistics personnel, and field crews to manage equipment movements in real time.

**Check-Out (Loadout) Flow:**
1. Work order is created (F2) and assets are assigned (F3)
2. System generates a **loadout ticket** listing equipment types + quantities for the job
3. Yard supervisor opens the mobile app, pulls up the loadout ticket
4. Each line item shows the **component image** alongside the asset name and quantity -- visual confirmation helps yard crews identify equipment quickly, especially for similar-looking valves with different specs
5. For sub-assemblies: shown as a single line item (e.g., "1x 715 HYD Frac Stack") with expandable cross-section diagram showing all components
6. A second person (verifier) confirms the loadout in the app
7. Loadout is marked "verified" -- equipment status changes to "in transit" then "deployed"
8. Loadout ticket is visible to field crew at the destination (see F8)

**Check-In (Receiving) Flow:**
1. Equipment returns to yard from a completed or modified job
2. Yard supervisor opens the mobile app, pulls up the original loadout ticket
3. Supervisor enters the returning quantities per equipment type
4. **Quantity match:** If returning quantity equals the loadout quantity for an equipment type, no serial number check required -- fast path
5. **Quantity mismatch:** If returning quantity is less than loadout quantity, the app requires serial number verification for that equipment type to identify which specific assets are missing
6. For sub-assemblies: if the stack returns intact, check in as a unit. If components were swapped or are missing, break down the assembly and reconcile individually
7. Discrepancies are flagged and routed to the DM / operations team
8. Verified returns update equipment status back to "available" at that district/yard

**Serial Number Drill-Down:**
- Default view shows asset name + quantity (fast for high-volume operations)
- Serial number detail is available on demand but only *required* when quantities don't match
- Supports barcode/QR scan for serial number entry (future: NFC tags if geo-trackers are replaced)

#### F8: Field Crew View (Mobile App)
- View incoming equipment for their job (loadout ticket with line items and **component images** for visual identification on arrival)
- View job schedule and upcoming equipment changes
- Confirm receipt of equipment on-site (visual reference helps field crews verify they received the right equipment, especially when multiple similar valve types are on the same truck)
- View loadout ticket details (what was sent, when, from which yard)
- View sub-assembly cross-section diagram (what's in the stack they received, with each component labeled)

#### F9: Transfer Management
- Create transfer orders between districts
- Generate loadout/receiving tickets for transfers (same check-in/check-out flow as jobs)
- Track transfer cost (trucking method, estimated cost)
- Transfer history and pattern analysis (which routes are most common -- informs pre-positioning)

#### F10: Multi-District Supply Optimization
When a work order requires assets that exceed a district's available inventory, the system suggests optimal sourcing across districts.

**District Route Matrix:**

| Route | Distance | Drive Time | Notes |
|-------|----------|-----------|-------|
| Keithville, LA ↔ Midland, TX | ~570 mi | ~8.5 hr | Longest route (I-20 west). Hot shot is expensive. |
| Keithville, LA ↔ Pleasanton, TX | ~430 mi | ~6.5 hr | Mid-range (I-49/US-59 south). |
| Midland, TX ↔ Pleasanton, TX | ~330 mi | ~5 hr | Shortest route (I-10/US-87). These two districts can often serve the same jobs. |

**Trucking Cost Framework:**
System uses configurable assumed rates per route. No actual cost data exists today -- rates are set during initial configuration and adjusted over time.

| Trucking Method | Rate Model | Notes |
|----------------|-----------|-------|
| **Scheduled freight** | $/mile (lower rate, planned in advance) | Default for non-urgent transfers with 3+ days lead time |
| **Hot shot** | $/mile (premium rate, ~2x scheduled) | Triggered when lead time is <48 hours or equipment is urgently needed |

Rates are entered per route pair and method. System uses these to calculate estimated transfer cost in sourcing recommendations.

**Core Logic:**
1. Compare job requirements against the primary district's current + forecasted availability
2. For each shortfall, query other districts' surplus (available minus their own upcoming job commitments)
3. Rank sourcing options by: schedule impact (don't rob a district that needs it), estimated logistics cost (route distance × rate), timing (can it arrive before job start given drive time?), ownership priority (owned → consignment → sub-rental)
4. Special case: **Midland ↔ Pleasanton interchangeability** -- for jobs that either could service (330 mi / 5 hr between them), evaluate both as primary and recommend the better fit
5. Present sourcing recommendation with alternatives to CEO/President for approval
6. **Either the CEO or the President can approve** cross-district sourcing -- no auto-execution

**Outputs:**
- Sourcing recommendation with line-by-line breakdown: "Source 6x from Keithville, 2x from consignment partner MAV"
- Impact analysis: "This leaves Keithville with 4 surplus through April 20 (sufficient for their schedule)"
- Alternative options ranked by total cost/disruption
- One-click approval → auto-generates transfer orders and loadout tickets

#### F11: Workforce Management
Track and assign labor (Spartan employees and contract hands) to jobs.

**Core Features:**
- Maintain a workforce roster: Spartan employees + known contract labor providers
- Assign labor to jobs with priority: **Spartan internal first, contract labor second**
- Track who is deployed where and who is available, by district
- Each work order specifies hands needed; system checks availability and flags gaps
- Company-wide labor utilization dashboard: % Spartan hands deployed, total contract hands in use, cost
- DMs can see their district's labor availability and request additional hands
- Historical labor data to inform hiring decisions

#### F12: Machine Shop / Maintenance Tracking
Track assets in the machine shop so they're visible to the planning system.

**Core Features:**
- Create maintenance orders when assets go to the shop: asset ID, issue, type (repair/redress/inspection/test), expected completion date
- Maintenance order detail shows the **component image** so shop personnel can visually confirm the asset type
- Asset status changes to "in shop" -- excluded from "available" but visible in a separate queue
- Planning board (F2) factors in expected shop completions: "3 wing valves expected back by April 10"
- Machine shop dashboard: total assets in shop, aging (how long each has been there), expected return dates
- When work is complete, close the order → asset returns to "available" at its district
- Notification: "Asset #SPT-1201 (7-1/16" 15K Wing Valve) completed redress, now available in Keithville"

### 3.3 User Roles

| Role | Access | Platform |
|------|--------|----------|
| **CEO / President** | Full access: all districts, planning, forecasting, consignment, reporting, assembly management | Web (desktop) |
| **District Manager** | District-level: inventory, job planning, assignments, loadout approval, transfers | Web + Mobile |
| **Yard Supervisor** | Check-in/check-out, loadout ticket creation/verification, yard inventory, assembly build | Mobile (primary), Web |
| **Quality Manager** | Equipment certs, quality pack generation (Phase 3: KPA integration) | Web |
| **Logistics** | Loadout/receiving tickets, transfer orders, equipment movement tracking, trucking coordination | Web + Mobile |
| **Field Crew** | View incoming loadouts, job schedule, confirm receipt | Mobile only |

### 3.4 Non-Functional Requirements
- **Simplicity:** Must be understandable by operations guys, not just the CEO
- **Speed:** Dashboard must load in <2 seconds; no waiting on complex calculations
- **Mobile-first for field users:** Check-in/check-out must work on a phone in a yard with one hand
- **Offline tolerance:** System should handle delayed data entry gracefully (reality: tickets are 4 days late). Mobile app should queue actions when offline and sync when connectivity returns.
- **Confidentiality:** Delmar is extremely protective of business data -- no shared/public cloud without explicit approval. Discuss hosting options on-site.
- **Audit trail:** Every asset movement, status change, and assignment must be logged with user + timestamp. Delmar's reputation is built on honesty -- the system must be the proof.

### 3.5 Out of Scope (Phase 1)
- Full KPA integration (Phase 3 -- need to talk to quality manager first)
- Automated frac schedule ingestion (schedules arrive via email/phone -- manual entry for now)
- Geo-tracker / NFC tag integration (current hardware is unreliable)
- Quality pack generation tied to loadout tickets (Phase 3)
- Basis ERP integration (Phase 3 -- need to assess what Basis tracks and whether data is exportable)
- Automated sub-rental procurement
- Customer-facing portal

### 3.6 Equipment Tagging Recommendations
Current state: assets are labeled with painted, stamped, or tagged asset numbers. For the mobile check-in/check-out system (F7) to reach full efficiency, durable machine-readable tags are recommended.

**Recommended tag options for oilfield environments:**

| Tag Type | Durability | Read Method | Cost/unit | Notes |
|----------|-----------|-------------|-----------|-------|
| **Laser-etched metal QR plates** (stainless steel or aluminum) | Extreme -- survives heat, chemicals, impact, weather | Phone camera scan | $2-5 | Weld-on or rivet-on. Most common in oilfield asset tracking. Cannot be ripped off by forklifts. |
| **Ceramic-on-metal RFID tags** (e.g., Xerafy, Omni-ID) | Extreme -- rated for high temp, high pressure, chemical exposure | RFID reader or phone (NFC) | $8-15 | Designed for oil & gas. Embed in metal surface or epoxy-mount. Survives what kills plastic tags. |
| **Tamper-evident metal tags** (e.g., MSA Safety, InfoChip) | High -- stainless with riveted or welded mount | Phone camera (QR/barcode) or RFID | $5-10 | Industrial standard for valve tracking. Some include embedded RFID + visual QR combo. |
| **UV/chemical-resistant vinyl labels** (e.g., Brady B-593) | Moderate -- lasts 5-10 years outdoors, less durable than metal | Phone camera scan | $0.50-2 | Lowest cost option. Good for lower-abuse environments (spools, tees). Not suitable for high-impact zones. |

**Recommendation:** Laser-etched stainless steel QR plates for high-value assets (valves, Javelins, accumulators) -- they're virtually indestructible, forklift-proof, and readable with any phone camera. Supplement with ceramic RFID for sub-assemblies where you want to scan the whole stack at once. Budget estimate: ~$3-5 per asset × ~500 high-utilization assets = $1,500-2,500 for initial rollout. This is a Phase 2 initiative (alongside F7 mobile app) -- not required for Phase 0 or Phase 1.

---

## 4. Design System -- "Atlas"

Atlas is Spartan Energy Services' internal asset management platform. The visual identity is derived from the Spartan Energy brand -- their logo, website (spartansenergy.com), and the industrial character of their operations.

### 4.1 Color Palette

**Primary Colors (from Spartan logo + website):**

| Token | Hex | Usage |
|-------|-----|-------|
| `spartan-red` | `#B32317` | Primary brand accent. CTA buttons, active states, critical alerts, navigation highlights. Drawn from the helmet plume and wordmark. |
| `spartan-red-dark` | `#8B1A1A` | Hover/pressed states on red elements. Deeper crimson from logo shadow. |
| `spartan-red-light` | `#B44028` | Secondary red variant (burnt terracotta from website). Used for warm accents, links. |
| `spartan-gold` | `#C5A55A` | Accent highlight. Drawn from the helmet's metallic gold. Used sparingly for premium indicators, consignment partner badges, achievement states. |
| `spartan-gold-light` | `#D4BC7C` | Gold hover states, subtle highlights on light backgrounds. |

**Neutral Colors (from website + logo):**

| Token | Hex | Usage |
|-------|-----|-------|
| `charcoal` | `#231F20` | Primary text color. From the logo and website body text. |
| `slate-dark` | `#333333` | Secondary text, sidebar backgrounds, table headers. |
| `slate` | `#555555` | Tertiary text, labels, placeholders. |
| `steel` | `#888888` | Disabled text, borders, dividers. |
| `silver` | `#D1D5DB` | Light borders, input outlines, card separators. |
| `fog` | `#F3F4F6` | Page backgrounds, alternating table row stripes. |
| `white` | `#FFFFFF` | Card backgrounds, content areas, input fields. |

**Semantic / Status Colors:**

| Token | Hex | Usage |
|-------|-----|-------|
| `status-available` | `#16A34A` | Available assets, healthy utilization, completed actions. Green = good. |
| `status-deployed` | `#2563EB` | Deployed/in-transit assets, active jobs, in-progress states. Blue = working. |
| `status-warning` | `#D97706` | Utilization 75-90%, approaching shortfall, items needing attention. Amber = watch. |
| `status-critical` | `#B32317` | Utilization >90%, shortfalls, discrepancies, overdue shop items. Uses `spartan-red` -- red = act now. |
| `status-shop` | `#7C3AED` | Assets in machine shop / maintenance. Purple = out of rotation. |
| `status-inactive` | `#9CA3AF` | Decommissioned, standby, unavailable. Grey = dormant. |

**Ownership Class Colors (matching Delmar's spreadsheet mental model):**

| Token | Hex | Usage | Spreadsheet Equivalent |
|-------|-----|-------|----------------------|
| `owned` | `#16A34A` (green) | Spartan-owned assets | Green cells |
| `consignment` | `#EC4899` (pink) | Consignment partner assets | Pink cells |
| `consignment-alt` | `#F59E0B` (amber) | Consignment -- secondary partner differentiation | -- |
| `sub-rental` | `#6366F1` (indigo checkered pattern) | Sub-rented assets | Checkered cells |

### 4.2 Typography

Derived from spartansenergy.com's font stack:

| Role | Font | Fallback Stack | Weight | Usage |
|------|------|---------------|--------|-------|
| **Headings** | PT Serif | Georgia, Times New Roman, serif | 700 (Bold) | Page titles, section headers, dashboard KPI labels. Conveys authority and tradition. |
| **Body** | PT Sans | Helvetica, Arial, sans-serif | 400 (Regular), 600 (Semi-Bold) | All body text, form labels, table content, descriptions. Clean and readable. |
| **Data / Technical** | Montserrat | system-ui, sans-serif | 500 (Medium), 700 (Bold) | Numbers in dashboards, utilization percentages, asset IDs, serial numbers. Monospaced feel for alignment. |

**Font Sizes:**

| Element | Desktop | Mobile |
|---------|---------|--------|
| H1 (Page title) | 32px | 24px |
| H2 (Section) | 24px | 20px |
| H3 (Card title) | 18px | 16px |
| Body | 16px | 14px |
| Small / Caption | 13px | 12px |
| KPI Number | 36px | 28px |

### 4.3 Layout & Components

**Overall Feel:** Industrial, professional, bold, no-nonsense. Clean white content areas with dark accents. The website's hero treatment (full-width oilfield photography with dark overlay) informs the dashboard header style.

**Navigation:**
- Desktop: Left sidebar (dark `charcoal` background, white text, `spartan-red` active indicator) -- mirrors the website's left-aligned nav
- Mobile: Bottom tab bar (white background, `spartan-red` active icon)

**Cards:**
- White background, `silver` border (1px), 0px border-radius (sharp corners -- matches the website's angular, industrial aesthetic)
- Subtle shadow on hover for interactive cards
- Section headers use PT Serif with a thin `spartan-red` underline (matching the website's "OUR SERVICES" treatment)

**Tables:**
- `fog` alternating row stripes
- `slate-dark` header row with white text
- Ownership class indicated by a colored dot or left-border stripe (green/pink/indigo)
- Utilization cells use background color fill matching status colors

**Buttons:**
- Primary: `spartan-red` background, white text, sharp corners, uppercase text (matching "VIEW SERVICES" button on the website)
- Secondary: white background, `spartan-red` border and text
- Danger: `spartan-red-dark` background (for destructive actions)
- Disabled: `steel` background, `slate` text

**Dashboard KPIs:**
- Large Montserrat numbers with PT Serif labels below
- Color-coded by status (green = healthy, amber = watch, red = critical)
- Utilization gauges use `spartan-red` fill on dark background (echoing the hero section's dark-over-industrial imagery)

**Assembly Cross-Section Diagrams:**
- Dark `charcoal` background (like the hero section) with component outlines in `silver`
- Filled slots show component images with ownership border color (green/pink)
- Empty slots show dashed `steel` outlines
- Interactive hover highlights in `spartan-gold`

**Mobile-Specific:**
- Large touch targets (minimum 48px)
- High contrast for outdoor visibility in yard conditions
- Status colors at full saturation (no pastels -- needs to be readable in direct sunlight)
- Loadout ticket line items: component image (left) + name + quantity (right), full-width rows for easy one-handed scrolling

### 4.4 Iconography

- Line-style icons (not filled) matching the angular, industrial feel
- `charcoal` default, `spartan-red` for active/selected states
- Equipment type icons should be simplified silhouettes matching the component images in the catalog

### 4.5 Dark Mode

Not planned for Phase 0-1. If requested, the dark theme would invert to:
- Background: `charcoal` → `slate-dark`
- Cards: `#1F2937`
- Text: `white` / `fog`
- `spartan-red` and status colors remain unchanged (already high-contrast)

---

## 5. Proposed Architecture

### Web Application (Desktop Users)
- **Frontend:** Next.js + Tailwind CSS
- **Backend:** Supabase (PostgreSQL + Auth + Row-Level Security)
- **Hosting:** Vercel (frontend) + Supabase cloud
- **Why:** Fast to build, real-time subscriptions for live dashboards, role-based access per user role

### Mobile Application (Field / Yard Users)
- **Option A: React Native (Expo)** -- single codebase for iOS + Android (mixed device fleet confirmed), shares business logic with web app, camera access for barcode/QR scanning
- **Option B: Progressive Web App (PWA)** -- no app store deployment, works on any phone browser, limited offline/camera capabilities
- **Recommendation:** Start with PWA -- good cell coverage is confirmed at yards and well sites, and PWA avoids app store deployment friction for a mixed iOS/Android fleet. Revisit React Native only if camera-based tag scanning becomes a primary workflow (Phase 2 tagging rollout).
- **Key mobile screens:** Loadout ticket view, check-out verification, check-in with quantity match, serial number drill-down, assembly component view, field crew job view

### Phasing
- **Phase 0:** Web app prototype -- F1 (inventory dashboard) + F2 (job planning board). Validate with Delmar on-site in April.
- **Phase 1:** F3 (equipment assignment) + F3b (sub-assembly management) + F4 (consignment) + F5 (utilization) + F6 (DM view) + F9 (transfers) + F10 (multi-district supply optimization) + F11 (workforce management) + F12 (machine shop tracking). Core operational system.
- **Phase 2:** F7 (mobile check-in/check-out) + F8 (field crew view) + durable asset tagging rollout. Replaces handwritten delivery tickets.
- **Phase 3:** KPA integration, quality pack generation, Basis data import, advanced reporting.

---

## 6. Answered Questions

| # | Question | Answer |
|---|----------|--------|
| 1 | Equipment taxonomy | 23+ types confirmed from screenshots (Section 1.3). More to come. |
| 2 | District structure | Three districts: Keithville LA, Midland TX, Pleasanton TX |
| 3 | User roles | CEO, President, DMs, yard supervisors, quality manager, logistics, field crew (Section 3.3) |
| 4 | KPA details | Unknown system details. Integration deferred to Phase 3. Assess on-site. |
| 5 | Basis | Unknown what it tracks or data exportability. Deferred to Phase 3. |
| 6 | JAVELIN | Zipper manifold with swing arm (like cement truck). Replaces traditional zipper tech. Standalone asset. |
| 7 | Consignment contracts | **Unique per partner** -- no two deals are the same. Data model supports per-partner terms. |
| 8 | Frac schedules | Arrive via **email or phone**. Manual entry for now; automated ingestion deferred. |
| 9 | Machine shop tracking | **Yes, needed in Phase 1.** Added as F12 and UF15. |
| 10 | Historical data | Current Excel is **not importable** in its current format. Will need manual data seeding or a custom import script. |
| 11 | Budget/pricing model | **VectisOS consulting engagement** (not a product play). |
| 12 | Mobile devices | **Mixed iOS and Android** across yards and field crews. |
| 13 | Connectivity | **Good cell reception** at yards and well sites. PWA is viable. |
| 14 | Loadout verification | **Typically a supervisor** serves as the second verifier. |
| 15 | Equipment labeling | Currently **painted, stamped, or tagged** with asset numbers (not machine-readable). Open to durable tags -- see Section 3.6 for recommendations. |
| 16 | Sub-assembly definitions | **Relatively fixed templates, but can vary by customer/job.** System needs both: standard templates + ability to customize per job. |
| 17 | SE brand | Unknown -- **confirm on-site**. |
| 18 | Master valve brands | **Interchangeable** on jobs -- customers don't specify brands. |
| 19 | Labor tracking | **In scope for Phase 1.** Spartan internal labor first, 3rd-party contract labor second. Added as F11 and UF14. |
| 20 | Data seeding | **Spreadsheet photo → AI conversion.** Photograph the current Excel, use AI to extract and structure the data into an importable dataset. Build a custom import script for initial load. |
| 21 | Contract labor providers | **Multiple providers**, likely ad hoc. Catalog main providers on-site for workforce roster seed. |
| 22 | Machine shop capacity | **Assume no constraints for now.** Each district assumed to have shop access. Document actual setup on-site. |
| 23 | Trucking cost data | **Not tracked today.** System will use assumed rates per route (scheduled vs. hot shot) to support sourcing decisions. Rates to be configured during setup. |
| 24 | District distances | See route table in F10. Keithville↔Midland ~570mi/8.5hr, Keithville↔Pleasanton ~430mi/6.5hr, Midland↔Pleasanton ~330mi/5hr. |
| 25 | Approval workflow | **Either CEO or President can approve independently.** No dollar threshold -- either can approve any cross-district sourcing decision. |

## 7. Remaining Open Questions

1. **SE brand:** What does "SE" stand for in "7 15 ZIPPERS SE Hyd Valves"? *(Confirm on-site)*
2. **Trucking rate benchmarks:** Need assumed $/mile or flat rates for scheduled vs. hot shot between each district pair. Delmar or ops team can provide ballpark during on-site.
3. **Contract labor provider list:** Probably multiple providers -- need to catalog the main ones on-site for the workforce roster seed.

---

## 8. Action Items

| Item | Owner | Target Date |
|------|-------|------------|
| Draft process/workflow document | Alex | Mid-April 2026 |
| Schedule on-site visit to Lafayette | Alex | Late April 2026 |
| Add equipment screenshots to project folder | Alex | Done (4 screenshots) |
| Get complete equipment taxonomy from Delmar | Alex (on-site) | April 2026 |
| Document sub-assembly configurations on-site | Alex (on-site) | April 2026 |
| Talk to quality manager about KPA | Alex (on-site) | April 2026 |
| Assess Basis: what it tracks, exportability | Alex (on-site) | April 2026 |
| Document loadout/receiving ticket workflow on-site | Alex (on-site) | April 2026 |
| Confirm SE brand meaning | Alex (on-site) | April 2026 |
| Map district-to-district distances and transit times | Alex (on-site) | April 2026 |
| Document machine shop workflow and capacity per district | Alex (on-site) | April 2026 |
| Identify contract labor providers and relationships | Alex (on-site) | April 2026 |
| Data seeding: photograph spreadsheet → AI extraction → import script | Alex (on-site) | April 2026 |
| Research durable asset tag vendors (laser-etched QR plates) | Alex | April 2026 |
| Source/create component images and assembly cross-section diagrams | Alex (on-site) | April 2026 |
| Build Phase 0 prototype (F1 + F2) | Alex | TBD after on-site |
| Scope VectisOS consulting SOW for Spartan engagement | Alex | May 2026 |

---

## 9. Confidentiality Notice

All information in this document is confidential to Spartan Energy Services and VectisOS. Delmar has explicitly requested that no business details be shared externally. This PRD is for internal VectisOS project planning only.
