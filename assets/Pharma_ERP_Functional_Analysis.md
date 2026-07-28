# Pharma ERP — Material Management Backbone
## Functional & Operational Analysis of the Menu/Submenu Structure

---

## A. Business Process Flow

The menu structure encodes a single, continuous **Procure → Receive → Inspect → Produce → Store → Release → Dispatch → Report → Comply** lifecycle. Below is that lifecycle traced module-by-module, with every decision/approval gate called out explicitly.

**1. Demand & Planning (Production, Master)**
Production Planning and Material Requirement Planning (MRP) consume Product Master, BOM, and Recipes/Formulas data together with Sales Order demand (from Sales & Dispatch) and current Raw Material Inventory (Warehouse) to generate net material requirements. This is the trigger point for Procurement.
*Decision gate:* MRP output must be reviewed/approved before it is allowed to generate a Purchase Requisition.

**2. Procurement (Procurement)**
Purchase Requisition → Request for Quotation (RFQ) → Vendor Quotation → Quotation Comparison → Purchase Order (PO) → Purchase Contract (for standing/annual rate contracts) → Vendor Advance (if applicable).
*Decision gate:* Quotation Comparison must be signed off (commercial + technical) before a PO can be raised. PO itself requires an Approval Matrix (Administration) sign-off tied to value thresholds.

**3. Goods Receipt (Goods Receipt)**
Against an approved PO, Material Receipt is logged, a Goods Receipt Note (GRN) is created, and Batch Creation assigns a unique batch/lot number (per Batch Numbering rules in Master). Label Printing and Barcode Generation follow immediately so the physical material is traceable from the dock onward.
*Decision gate:* GRN cannot be closed unless it references an approved PO/Vendor Quotation and quantity/quality tolerances are met; discrepancies route to Supplier Returns.

**4. Quality Control — Incoming (Quality Control)**
Received batches are held in a quarantine status pending Incoming Quality Inspection. Sampling → Test Requests → Test Results → Certificate of Analysis (COA) generation. If results are unusual, they route to OOS/OOT investigation, which may trigger a Deviation and, if root-caused to a systemic issue, a CAPA.
*Decision gate:* Only material with an approved COA and closed IQI status is released from quarantine into Raw Material Inventory (Warehouse). Material with pending or failed QC cannot be issued to Production.

**5. Warehousing — Raw Material (Warehouse)**
Released raw material enters Raw Material Inventory under Bin Management, subject to FEFO/FIFO issuance logic driven by Expiry Management and Batch Management. Stock Adjustment, Cycle Count, and Physical Stock Verification maintain inventory accuracy against book stock.

**6. Production (Production)**
Production Schedule (derived from Production Planning/MRP) generates a Production Order, which spawns a Batch Manufacturing Record (BMR). Material Issue draws from Raw Material Inventory (FEFO/FIFO enforced) and decrements Warehouse stock while creating WIP Inventory.
Line Clearance and Equipment Cleaning records must be closed before Process Execution can start. Process Execution is monitored via In-Process Inspection (feeding In-Process Quality in QC). Machine Allocation tracks equipment/line utilization against the order.
Post-processing, the batch moves to packaging: Batch Packaging Record (BPR), Yield Calculation, and Production Completion. Exceptions during the run are handled via Rework or Scrap Management. The order is closed via Batch Closure.
*Decision gate:* Line Clearance + Equipment Cleaning sign-off is mandatory before Process Execution; Yield outside tolerance may trigger a Deviation in QC before Batch Closure is permitted.

**7. Quality Control — In-Process & Final (Quality Control)**
In-Process Inspection results feed In-Process Quality. At batch completion, Final Product Quality testing is performed, culminating in Batch Release — the single most gated event in the whole system.
*Decision gate:* Batch Release requires: closed COA (raw material and in-process), closed Final Product Quality testing, no open Deviation/OOS/OOT against the batch, and (where applicable) a favorable Stability Studies read-point. Change Control sign-off is required if the batch was manufactured under a modified process/spec.

**8. Finished Goods (Finished Goods)**
Only released batches move into Finished Goods Receipt. Product Release is the commercial release checkpoint (distinct from QA Batch Release — commonly QA release then Product/Marketing release). Labeling & Coding, Serialization (regulatory track-and-trace), and Palletization prepare the batch for storage/shipment. Batch Traceability here is the forward link from raw material batch → production batch → FG batch. Hold/Release governs any FG batch placed under commercial or regulatory hold (e.g., pending recall investigation).
*Decision gate:* FG cannot transfer to Sales & Dispatch while in Hold status.

**9. Warehousing — Finished Goods (Warehouse / Finished Goods)**
Finished Goods Inventory is maintained with the same Bin Management, Expiry Management, and Near Expiry Stock disciplines as raw material, ensuring FEFO allocation at the point of sale.

**10. Sales & Dispatch (Sales & Dispatch)**
Sales Order → Pick List (FEFO-driven from FG Inventory) → Delivery Order → Packing → Shipment → Invoice. Logistics Tracking monitors in-transit status; the Dispatch Register is the system of record for what left the plant, when, and against which order. Customer Returns re-enters the system as a reverse-logistics event that must reconcile against original batch numbers for traceability.
*Decision gate:* Pick List cannot allocate a batch that is on Hold, expired, or lacking Product Release status.

**11. Reporting (Reports)**
Every module above feeds the Reports layer: Inventory, Procurement, Production, Quality, Warehouse, Dispatch, Expiry, and Batch Traceability reports are module-specific extracts; Audit Reports and User Activity draw from Compliance/Administration logs; the Executive Dashboard and Custom Reports aggregate across all of the above for leadership consumption.

**12. Compliance (Compliance — runs in parallel to every step above, not after it)**
Audit Trail and Electronic Signatures are embedded at every data-entry and approval point across all modules (21 CFR Part 11 requirement — not a discrete workflow step but a continuous control). Document Management and SOP Management govern the controlled documents referenced by Production, QC, and Warehouse. Validation Documents support equipment/process qualification status referenced during Line Clearance and Batch Release. Recall Management is the emergency reverse-workflow, triggered from Batch Traceability data and executing Hold/Release and Customer Returns in coordination.

**13. Administration (Administration — governs the system underneath all of the above)**
User Management, Roles & Permissions, Menu Access, and Approval Matrix determine who can execute which decision gate in steps 1–12. Number Series and Fiscal Year underpin document numbering integrity (tying back to Batch Numbering/Document Numbering in Master). API Integration and Import/Export handle interfacing with external systems (e.g., accounting, government e-way bill/serialization portals). System Health and Backup & Restore are operational-continuity controls required for Part 11 compliant systems.

---

## B. User Roles

| Role | Primary Modules Used | Key Responsibilities |
|---|---|---|
| **Purchase Officer / Procurement Executive** | Procurement, Master (Vendors), Reports (Procurement) | Raise Purchase Requisition/RFQ, manage Vendor Quotations & Comparison, issue PO/Purchase Contract, track Vendor Advance & Purchase Invoice, manage Debit Notes/Vendor Returns |
| **Warehouse Executive / Store Keeper** | Goods Receipt, Warehouse, Master (Storage Locations) | Post Material Receipt, create GRN, manage Bin/Batch storage, execute Stock Transfer/Adjustment, Cycle Count, enforce FEFO/FIFO issuance |
| **QC Analyst** | Quality Control | Perform Sampling, log Test Requests/Results, issue COA, execute In-Process & Final Product Quality checks, initiate OOS/OOT investigations |
| **QA Head / QA Manager** | Quality Control, Compliance | Approve Batch Release, own Deviations/CAPA/Change Control closure, oversee Stability Studies program, sign off Recall Management decisions |
| **Production Supervisor** | Production | Execute Production Order, maintain BMR, manage Material Issue, oversee Line Clearance/Equipment Cleaning, Process Execution, Machine Allocation |
| **Production/Packing Operator** | Production | Execute assigned process/packing steps under BMR/BPR, record in-process data, flag exceptions for Rework/Scrap |
| **Plant Head / Operations Head** | Production Dashboard, Warehouse Dashboard, Reports, Executive Dashboard | Monitor plant-wide throughput, yield, and OEE-type indicators; resolve cross-module escalations (line clearance delays, material shortages) |
| **Dispatch / Logistics Executive** | Sales & Dispatch, Finished Goods | Convert Sales Order to Delivery Order, build Pick List, manage Packing/Shipment, maintain Dispatch Register, track Logistics Tracking status |
| **Finished Goods / Warehouse Supervisor (FG)** | Finished Goods, Warehouse | Manage Product Release, Labeling & Coding, Serialization, Palletization, FG Hold/Release, FG stock accuracy |
| **Sales Coordinator / Customer Service** | Sales & Dispatch, Master (Customers) | Create Sales Order, process Customer Returns, coordinate Invoice generation |
| **Compliance Officer / Regulatory Affairs** | Compliance | Maintain SOP Management, Document Management, Validation Documents, Regulatory Compliance filings, own Recall Management execution |
| **Internal/External Auditor** | Compliance, Reports (Audit Reports) | Review Audit Trail, Electronic Signatures, User Activity logs for Part 11/Annex 11 conformance |
| **System Administrator** | Administration | Manage User Management, Roles & Permissions, Menu Access, Approval Matrix, Number Series, API Integration, Backup & Restore, System Health |
| **Finance / Accounts Executive** | Procurement (Purchase Invoice, Debit Note), Sales & Dispatch (Invoice) | Reconcile vendor and customer financial documents against GRN/Dispatch records |
| **Executive Leadership (CEO/COO/Plant Head)** | Reports (Executive Dashboard) | Consume cross-functional KPIs for strategic decisions — capacity, compliance risk, cost, service level |

---

## C. Dashboard Requirements

*(Data/metrics only — no layout or visual specification)*

**Procurement Dashboard**
- Open Purchase Requisitions awaiting RFQ/PO conversion
- PO status breakdown (draft / approved / partially received / closed)
- Vendor Quotation turnaround time and comparison outcomes
- Purchase Order value by vendor, category, and plant
- Vendor Advance outstanding balances
- Purchase Invoice vs. PO/GRN mismatches (three-way match status)
- Vendor Return / Debit Note volume and value
- Vendor on-time delivery and rejection rate

**Receipt Dashboard**
- Pending Receipts against open POs
- GRN volume and average GRN-to-quarantine-release cycle time
- Batches awaiting Batch Creation / labeling
- Supplier Return volume by reason code

**Quality Dashboard**
- Samples pending testing (by age, by material/product)
- Test Results turnaround time vs. SLA
- COA issuance status (raw material and finished product)
- Open Deviations, CAPA, and Change Control items (by age/severity)
- OOS/OOT count and trend, with investigation status
- Batch Release queue — batches pending, batches released, average release cycle time
- Stability Study schedule and upcoming/overdue pull-points

**Production Dashboard**
- Production Order status (planned / released / in-process / completed)
- BMR/BPR completion and open-exception status
- Line Clearance and Equipment Cleaning compliance rate
- Material Issue vs. Production Order requirement (variance)
- Yield performance vs. standard/theoretical yield
- Rework and Scrap volume and cost
- Machine/Line utilization and downtime
- Batch Closure backlog

**Warehouse Dashboard**
- Raw Material, WIP, and Finished Goods Inventory levels vs. reorder/safety stock
- Inventory Valuation summary
- Near Expiry Stock (by threshold band, e.g., 30/60/90 days)
- Cycle Count / Physical Stock Verification variance
- Stock Transfer and Stock Adjustment activity log
- FEFO/FIFO compliance exceptions (non-sequential issuance)

**Finished Goods Dashboard**
- FG batches pending Product Release
- Batches on Hold (with hold reason and age)
- Serialization/coding completion status
- Batch Traceability lookup readiness (raw material → FG linkage integrity)
- Palletization/dispatch-readiness queue

**Dispatch Dashboard**
- Open Sales Orders vs. Delivery Orders raised
- Pick List fulfillment status (FEFO adherence)
- Shipment/Logistics Tracking in-transit status
- Dispatch Register — daily/weekly dispatch volume and value
- Customer Return volume and reason trend
- On-time-in-full (OTIF) delivery performance

**Executive Dashboard**
- Cross-module KPI summary (Business + Executive tier from Section D)
- Plant-wide OTIF, right-first-time, and compliance-risk indicators
- Inventory value and working-capital exposure (raw material + WIP + FG)
- Open high-severity Deviation/CAPA/OOS count (compliance risk exposure)
- Revenue/dispatch value trend vs. plan
- Recall Management status, if any active recall
- Audit readiness indicator (Audit Trail exceptions, overdue SOP reviews)

---

## D. KPI Matrix

| KPI Name | Tier | Formula / Basis | Owning Module | Frequency |
|---|---|---|---|---|
| Purchase Order Cycle Time | Operational | Time from Purchase Requisition to PO approval | Procurement | Daily/Weekly |
| Vendor On-Time Delivery Rate | Business | On-time GRNs ÷ Total GRNs against PO due date | Procurement / Goods Receipt | Monthly |
| Vendor Rejection Rate | Operational | Rejected quantity ÷ Total received quantity (from IQI) | Quality Control | Monthly |
| Three-Way Match Exception Rate | Operational | Invoices with PO/GRN mismatch ÷ Total invoices | Procurement | Monthly |
| GRN-to-Quarantine-Release Cycle Time | Operational | Time from GRN posting to QC release | Goods Receipt / Quality Control | Weekly |
| Right-First-Time (RFT) Rate | Business | Batches with no Deviation/Rework/OOS ÷ Total batches | Production / Quality Control | Monthly |
| OOS/OOT Incidence Rate | Operational | OOS/OOT count ÷ Total tests performed | Quality Control | Monthly |
| Batch Release Cycle Time | Business | Time from Production Completion to Batch Release | Quality Control | Monthly |
| Deviation/CAPA Closure Rate | Operational | Closed within SLA ÷ Total raised | Quality Control | Monthly |
| CAPA Aging | Operational | Average days open for unresolved CAPA | Quality Control | Weekly |
| Yield Variance | Operational | (Actual Yield − Standard Yield) ÷ Standard Yield | Production | Per Batch |
| Line/Machine Utilization | Operational | Productive time ÷ Available time | Production | Daily/Weekly |
| Rework/Scrap Cost Ratio | Business | Cost of Rework + Scrap ÷ Total production cost | Production | Monthly |
| Line Clearance Compliance Rate | Operational | Line Clearances completed on time ÷ Total required | Production | Weekly |
| Inventory Accuracy | Operational | Matched count (Cycle Count/Physical Verification) ÷ Total counted | Warehouse | Monthly |
| Inventory Turnover | Business | COGS (or issue value) ÷ Average Inventory Value | Warehouse | Monthly/Quarterly |
| Near-Expiry Stock Value | Business | Value of stock within expiry threshold band | Warehouse | Weekly |
| FEFO/FIFO Compliance Rate | Operational | Sequential issuances ÷ Total issuances | Warehouse | Monthly |
| Working Capital in Inventory | Executive | Sum of RM + WIP + FG Inventory Valuation | Warehouse / Finished Goods | Monthly |
| Product Release Cycle Time | Operational | Time from Batch Release (QA) to Product Release (commercial) | Finished Goods | Weekly |
| On-Time-In-Full (OTIF) | Business | Orders delivered complete & on time ÷ Total orders | Sales & Dispatch | Monthly |
| Order Fulfillment Cycle Time | Operational | Time from Sales Order to Shipment | Sales & Dispatch | Weekly |
| Customer Return Rate | Business | Returned quantity/value ÷ Dispatched quantity/value | Sales & Dispatch | Monthly |
| Batch Traceability Completeness | Executive | Batches with unbroken RM→FG trace ÷ Total batches | Reports / Compliance | Quarterly |
| Audit Trail Exception Count | Executive | Count of unresolved audit trail anomalies | Compliance | Monthly |
| SOP/Validation Document Overdue Rate | Executive | Overdue reviews ÷ Total scheduled reviews | Compliance | Monthly |
| Compliance Risk Index | Executive | Weighted composite of open Deviation/CAPA/OOS/Recall status | Compliance | Monthly |
| Recall Response Time | Business | Time from Recall trigger to full traceability report | Compliance | Per Event |

---

## E. Critical Alerts Matrix

| Alert | Trigger Condition | Source Module | Severity | Recipient Role |
|---|---|---|---|---|
| Out-of-Specification (OOS) Result | Test Result fails predefined specification limit | Quality Control | Critical | QC Analyst, QA Head |
| Out-of-Trend (OOT) Result | Result within spec but trending outside historical pattern | Quality Control | High | QA Head |
| Batch on Hold | Hold/Release status set to Hold on an FG or in-process batch | Finished Goods / Quality Control | Critical | QA Head, Plant Head |
| Overdue Deviation | Deviation open beyond defined SLA (e.g., 30 days) | Quality Control | High | QA Head, Compliance Officer |
| Overdue CAPA | CAPA action item past due date | Quality Control | High | QA Head |
| Near-Expiry Stock | Batch expiry within defined threshold (e.g., 90/60/30 days) | Warehouse | Medium–High (escalating as date nears) | Warehouse Executive, Plant Head |
| Expired Stock in Inventory | System date exceeds batch expiry date and stock not disposed | Warehouse | Critical | Warehouse Executive, QA Head |
| Line Clearance Not Completed | Process Execution attempted without closed Line Clearance record | Production | Critical | Production Supervisor, QA |
| Equipment Cleaning Overdue | Cleaning record not closed within validated cleaning cycle | Production | High | Production Supervisor |
| Yield Deviation Beyond Tolerance | Actual Yield outside defined % band of Standard Yield | Production | Medium | Production Supervisor, QA Head |
| Pending QC Release Beyond SLA | Material/batch in quarantine beyond target release time | Quality Control | Medium | QC Analyst, Warehouse Executive |
| Stock Below Reorder/Safety Level | Raw Material Inventory falls below defined threshold | Warehouse / Procurement | High | Purchase Officer, Warehouse Executive |
| GRN-PO Quantity/Price Mismatch | Received quantity/price variance beyond tolerance vs. PO | Goods Receipt | Medium | Purchase Officer, Warehouse Executive |
| Vendor Quotation/PO Approval Pending | Approval Matrix threshold breached, awaiting sign-off | Procurement / Administration | Medium | Approving Authority (per Approval Matrix) |
| Recall Triggered | Recall Management event initiated against a batch | Compliance | Critical | QA Head, Compliance Officer, Plant Head, Executive Leadership |
| Change Control Pending Impact Assessment | Change Control raised but not yet assessed/approved | Quality Control | Medium | QA Head |
| Audit Trail Anomaly | Unusual/void/backdated transaction detected in Audit Trail | Compliance | High | Compliance Officer, System Administrator |
| SOP/Validation Document Overdue for Review | Document review due date passed without revalidation | Compliance | Medium | Compliance Officer |
| Customer Return with Quality Complaint | Customer Return logged with a quality-related reason code | Sales & Dispatch / Quality Control | High | QA Head, Sales Coordinator |
| Serialization/Coding Failure | Serialization not completed/verified before dispatch eligibility | Finished Goods | High | FG Supervisor |
| Shipment Delay | Logistics Tracking status shows delay past promised date | Sales & Dispatch | Medium | Dispatch Executive |

---

## F. Recommended Dashboard Sections

*(Widget/section names and purpose only — no visual specification)*

**Procurement Dashboard**
- *RFQ-to-PO Funnel* — visibility of requisitions stuck at each procurement stage
- *Vendor Scorecard* — on-time delivery, rejection rate, price trend per vendor
- *Open Commitments* — PO value not yet received, by vendor and material category
- *Invoice Reconciliation Queue* — three-way match exceptions awaiting resolution

**Receipt Dashboard**
- *Pending Receipts Queue* — POs due/overdue for receipt
- *Quarantine Aging* — batches awaiting QC release, sorted by time in quarantine
- *Supplier Return Log* — active returns and reason codes

**Quality Dashboard**
- *Testing Backlog* — samples pending by material, priority, and age
- *Batch Release Queue* — batches awaiting release with blocking-item detail (open Deviation/OOS/COA)
- *Deviation & CAPA Tracker* — open items by age, severity, and owning module
- *OOS/OOT Trend* — incidence over time, by product/line
- *Stability Program Calendar* — upcoming and overdue stability pull-points

**Production Dashboard**
- *Order Status Board* — production orders by stage (planned/released/in-process/complete)
- *Line Clearance & Cleaning Compliance* — real-time status per line
- *Yield & Rework Tracker* — variance from standard yield, rework/scrap volumes
- *Machine Utilization Summary* — utilization and downtime by equipment/line

**Warehouse Dashboard**
- *Inventory Position* — RM/WIP/FG stock levels vs. thresholds
- *Expiry Watchlist* — near-expiry and expired stock by threshold band
- *Cycle Count Variance Log* — discrepancies from physical verification
- *FEFO/FIFO Exception List* — non-sequential issuances flagged for review

**Finished Goods Dashboard**
- *Release Pipeline* — batches from QA release to commercial Product Release
- *Hold Register* — FG batches on hold with reason and duration
- *Traceability Integrity Check* — batches with incomplete RM-to-FG linkage

**Dispatch Dashboard**
- *Order Fulfillment Funnel* — Sales Order → Delivery Order → Shipment progression
- *Pick List FEFO Compliance* — allocation adherence to expiry sequencing
- *Dispatch Register Summary* — dispatch volume/value by period, customer, product
- *Customer Returns Log* — return volume and reason trend

**Executive Dashboard**
- *Enterprise KPI Summary* — Business + Executive tier KPIs across all modules
- *Compliance Risk Panel* — open high-severity Deviation/CAPA/OOS/Recall status
- *Working Capital & Inventory Exposure* — RM+WIP+FG valuation trend
- *Service Level Summary* — OTIF, order fulfillment cycle time
- *Audit Readiness Panel* — audit trail exceptions, overdue SOP/validation reviews

---

*All modules, submodules, and terminology referenced above are drawn directly from the supplied menu/submenu structure; no additional modules have been introduced.*
