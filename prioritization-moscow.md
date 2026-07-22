# Prioritisation — MoSCoW & Value/Effort

How scope was decided, and what was consciously deferred — the reasoning a PO brings to a roadmap conversation.

## MoSCoW

### Must have (built in the prototype)
- Add order via **manual form or optional AI extract** with per-field confidence *(FR-1)*
- **Item master** + **smart-search** item selection + **not-in-master → add** *(FR-2)*
- **Item-wise order value**, snapshotted at creation *(FR-3)*
- Create rules incl. missing-essentials → Exception *(FR-4)*
- **Partial approval** with Available ≤ Ordered validation and no-items → Not Available *(FR-5)*
- **Ordered / Available / Delivered value tracking** *(FR-6)*
- Lifecycle incl. **Arrived** stage, **Exception with remarks / return-to-stage**, **Not Available** *(FR-L)*
- **Shipment tracking** map across in-transit / arrived / delivered *(FR-7)*
- **POD gated at Arrival**, received-qty capture, discrepancy remarks, **editable POD with audit** *(FR-8)*
- Console (filter, search, KPIs) + **full-page detail** with tabs; Item master view *(FR-9)*

*Rationale: together these prove the thesis — fast, accurate intake; catalogue-driven value; and a lifecycle where approval, dispatch and delivery are evidenced, so value and data quality stay trustworthy end to end.*

### Should have (next)
- Configurable confidence threshold in the UI (currently a documented constant)
- Customer master + validation of the extracted customer
- Duplicate-order detection on PO/reference
- Bulk intake (multiple orders / file upload)
- Item master fields beyond price (SKU, category, tax) — kept in sync with "add to master"

### Could have (later, if value proven)
- Real carrier tracking + OCR/LLM extraction integrations (interfaces already specified)
- Extraction feedback loop to improve the model on low-confidence fields
- SLA timers on exceptions and arrivals
- Role-based views (operator vs lead) and permissions

### Won't have (this version)
- Payments, invoicing, tax, multi-currency
- Inventory allocation / warehouse management
- Auth & permissions (assumed from host platform)

## Value vs Effort (the Should/Could backlog)

```
   High │  Customer-master        Real carrier +
 V      │  validation ★           OCR integrations
 A      │
 L      │  Configurable           Bulk / file
 U      │  threshold ★            intake ★
 E      │
   Low  │  Duplicate              Extraction
        │  detection              feedback loop
        └───────────────────────────────────────
            Low            Effort           High
```
★ = strongest near-term candidates.

### First thing I'd build next
**Configurable confidence threshold + customer-master validation.** The threshold is already the core governance lever — exposing it turns a constant into an operations control at low effort. Customer validation removes the most common low-confidence field, cutting review load. Together they move the metric that matters in v3: **% of orders created clean on first pass.**

## Success metrics
- **Intake time per order** vs manual keying
- **% orders created clean** (no Exception) on first pass
- **Field-level review rate** (which extracted fields flag most → where to invest)
- **Value leakage caught** at approval/delivery vs discovered later
- **Exceptions resolved before dispatch** vs errors found at/after dispatch
