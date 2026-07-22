# Functional Requirements Document — Cargoflow OMS

| | |
|---|---|
| **Author** | Ayush Yadav |
| **Status** | v2.0 — prototype scope |
| **Related** | [User stories](user-stories.md) · [Process flows](process-flows.md) · [API contract](api-contract.yaml) · [Test scenarios](test-scenarios.md) · [Prioritisation](prioritization-moscow.md) |

## 1. Purpose & background

Orders reach the order desk as **unstructured text** (forwarded email, PDF, chat). Operators re-key them into the OMS, which is slow, error-prone, and defers data-quality problems to dispatch. This document specifies an order-management workflow that (a) speeds intake with optional AI extraction, (b) prices orders from a maintained item master, (c) tracks what is *ordered → available → delivered*, and (d) moves each order through a lifecycle whose side-effecting steps — approval, dispatch, delivery — are evidenced and auditable.

### Goals
- Reduce manual keying effort and transcription errors at intake.
- Make order value a calculated, catalogue-driven number rather than a typed guess.
- Surface data-quality and fulfilment gaps early (at approval and at delivery), not after the fact.
- Keep a human accountable and an audit trail for every state change.

### Non-goals (this version)
- Real payment, invoicing, tax or multi-currency.
- Real carrier/OCR integrations (both are simulated; the [API contract](api-contract.yaml) defines the real interfaces).
- Authentication, roles and permissions (assumed provided by the host platform).

## 2. Definitions
- **Item master** — catalogue of items, each with a value per unit, used to price orders.
- **Ordered / Available / Delivered value** — order value at three points: as ordered, after partial approval, and after delivery receipt.
- **Partial approval** — deciding, per item, what can actually be fulfilled (quantity available), rejecting the rest.
- **Confidence score** — model-reported likelihood (0–1) that an AI-extracted field is correct.
- **POD** — proof of delivery: document plus received-quantity capture recorded when a shipment is handed over.
- **Exception** — a review state an order can be flagged into (with remarks) from any active stage; on resolve it returns to the stage it came from.

## 3. Order lifecycle

`New → Confirmed → Picking → Shipped → Arrived → Delivered`, with two off-path states: **Exception** (reviewable, reversible) and **Not Available** (nothing fulfillable; terminal).

- **FR-L1** The system shall advance an order one stage at a time along the primary path.
- **FR-L2** The transition **Arrived → Delivered** shall require a recorded POD; it shall not be a plain advance.
- **FR-L3** The transition into **Shipped** shall create shipment tracking data (carrier, tracking number, destination).
- **FR-L4** Any active order may be flagged as an **Exception** with mandatory remarks; the order retains the stage it was flagged from, and **resolve** returns it to that stage (not a fixed stage).
- **FR-L5** Confirming a **New** order with no items selected shall, after explicit confirmation, move it to **Not Available**.
- **FR-L6** Every state change shall append a timestamped entry (with note where applicable) to the order's history.

## 4. Intake & order creation

### FR-1 Add order (form + optional AI extract)
- **FR-1.1** The add-order form (customer, PO/reference, ship-to, delivery date, items) shall be fully usable by manual entry.
- **FR-1.2** The system shall optionally accept pasted order text and, on request, extract the above fields to pre-fill the form, returning a **confidence score** per field and highlighting low-confidence fields (< 0.80) for review.
- **FR-1.3** The extractor shall not fabricate values; unknown fields are left empty.

### FR-2 Item entry & the item master
- **FR-2.1** The item name field shall be a **smart search** over the item master (type-ahead, keyboard-navigable, single-select).
- **FR-2.2** Selecting a master item shall auto-fill its **value per unit**.
- **FR-2.3** If an entered/extracted item is **not in the master**, the row shall be flagged and offer **Add to master**, which requires a value per unit.
- **FR-2.4** The item master shall support add, inline edit of value per unit, and delete.

### FR-3 Item-wise order value
- **FR-3.1** Order value shall be calculated as Σ (value per unit × quantity), shown live in the form.
- **FR-3.2** The value per unit shall be **snapshotted onto the order** at creation; later master edits do not alter existing orders.

### FR-4 Create rules
- **FR-4.1** A non-empty **customer** and at least one valid item are required to create an order.
- **FR-4.2** If delivery date or order value is missing at creation, the order is created as **Exception** (flagged); otherwise **New**.

## 5. Partial approval (New orders)
- **FR-5.1** For a New order, each item shall show a checkbox (available), an **Ordered Qty**, and an editable **Available Qty**.
- **FR-5.2** **Available Qty must not exceed Ordered Qty**; violations block confirmation with a clear error.
- **FR-5.3** On confirm, unticked items are marked **Rejected**; ticked items keep their available quantity; the order moves to **Confirmed**.
- **FR-5.4** If no item is selected, the system shall warn that the order will become **Not Available** and require a second confirmation.

## 6. Value tracking
- **FR-6.1** The system shall compute **Ordered value** (Σ unit × ordered), **Available value** (Σ unit × available, excluding rejected) and **Delivered value** (Σ unit × received, excluding rejected).
- **FR-6.2** The order's effective/displayed value shall follow its stage: ordered (pre-approval), available (post-approval), delivered (post-POD); the console shows the effective value with the original struck through when reduced.

## 7. Shipment tracking
- **FR-7.1** Shipped, Arrived and Delivered orders shall show a map with origin, destination, route and shipment position, plus carrier, tracking number and ETA.
- **FR-7.2** Position/state shall reflect the stage: **in transit** (Shipped), **at destination** (Arrived), **trip ended** (Delivered).
- **FR-7.3** Tracking is a **simulated** stand-in for a carrier tracking API (see [API contract](api-contract.yaml)).

## 8. Proof of delivery
- **FR-8.1** POD capture shall be available only in **Arrived** (i.e. after the shipment has reached the destination) — never while in transit.
- **FR-8.2** POD shall require a document upload and per-item **Received Qty** against the available (shipped) quantity.
- **FR-8.3** **Received Qty must not exceed Available Qty**; violations block completion.
- **FR-8.4** Unticked or short-received items shall be marked **Rejected/Short** and require **remarks**.
- **FR-8.5** On completion the order moves to **Delivered** and the POD (document, received-by, delivered-at, remarks) is stored and displayed.
- **FR-8.6** A submitted POD shall be **editable** (received quantities and details), applying the same validation and logging a **"POD corrected"** history event.

## 9. Console & detail
- **FR-9.1** The console shall list orders (id, customer, item count, value, ETA, status), filter by status, and search by id/customer/reference.
- **FR-9.2** Aggregate KPIs: open orders, orders needing review, open value, delivered count.
- **FR-9.3** Each order shall open as a **full page** with **Order details** and **Status timeline** tabs.
- **FR-9.4** A separate **Item master** view shall sit alongside the order console.

## 10. Business rules
- **BR-1** Confidence threshold = 0.80 (governance control for review load vs error risk).
- **BR-2** Prices are snapshotted at order creation (an order is priced at the agreed rate of that day).
- **BR-3** Value follows fulfilment reality: reductions at approval and delivery flow through to the order's value.
- **BR-4** Delivery must be *evidenced* (POD) — "Delivered" is never a plain click.
- **BR-5** Rejections are captured at two distinct points (approval and delivery) and both require remarks.
- **BR-6** Corrections to completed records (POD) are permitted but **logged**, never silent.

## 11. Assumptions & dependencies
- Extraction (LLM/OCR) and carrier-tracking services exposing the [documented API](api-contract.yaml) exist; the prototype simulates both on-device.
- Customer master and real pricing/tax live elsewhere and are out of scope for v2.

## 12. Acceptance
Satisfied when the [user stories](user-stories.md) pass their acceptance criteria and the [test scenarios](test-scenarios.md) — including edge cases — behave as specified.
