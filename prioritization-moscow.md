# Prioritisation — MoSCoW & Value/Effort

How scope was decided for v1 of the AI-assisted intake, and what was consciously deferred. This is the reasoning a PO brings to a roadmap conversation, not just a feature dump.

## MoSCoW

### Must have (in v1 — shipped in the prototype)
- Raw-text intake with realistic samples *(FR-1)*
- Field extraction with per-field confidence *(FR-2)*
- Confidence threshold + visual flagging of low-confidence fields *(FR-3.1–3.2)*
- Human review/edit before creation, with verify-clears-flag *(FR-3.3–3.4)*
- Create rules: mandatory customer; sub-threshold → Exception *(FR-4)*
- Order lifecycle + exception handling with history *(FR-5)*
- Console: list, filter, search, detail, KPIs *(FR-6)*

*Rationale: together these prove the core thesis — AI speeds intake while a human stays accountable and bad data is caught early. Removing any one breaks that loop.*

### Should have (next, not in v1)
- Validate extracted customer against customer master data
- Duplicate-order detection on PO/reference
- Bulk intake (multiple orders from one paste / file upload)
- Configurable threshold in the UI (currently a documented constant)

### Could have (later, if value proven)
- Per-field extraction feedback loop to improve the model
- Confidence trend analytics ("which fields are hardest to extract?")
- SLA timers on exceptions
- Role-based views (operator vs lead)

### Won't have (this version — explicitly out)
- Pricing, tax, multi-currency
- Fulfilment / inventory allocation / invoicing
- Auth & permissions (assumed from host platform)

## Value vs Effort (the "Should/Could" backlog)

```
   High │  Customer-master        Bulk / file
 V      │  validation ★           intake ★
 A      │
 L      │  Duplicate              Configurable
 U      │  detection ★            threshold (UI)
 E      │
        │  Confidence             Feedback loop
   Low  │  analytics              to model
        └───────────────────────────────────────
            Low            Effort           High
```
★ = strongest near-term candidates: high operator value, moderate effort.

### First thing I'd build next
**Configurable threshold in the UI + customer-master validation.** The threshold is already the product's core lever — exposing it turns a hidden constant into an operations control, at low effort. Customer validation removes the single most common cause of a flagged customer field, directly cutting review load. Together they improve the metric that matters most in v2: **% of orders created clean on first pass.**

## Success metrics (how we'd know v1 worked)
- **Intake time per order** (target: material reduction vs manual keying)
- **% orders created clean** (no Exception) on first pass
- **Field-level review rate** (which fields get flagged most → where to invest next)
- **Exceptions resolved before dispatch** vs errors caught at/after dispatch
