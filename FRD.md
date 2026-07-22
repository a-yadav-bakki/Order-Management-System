# Functional Requirements Document — Cargoflow OMS (AI-assisted intake)

| | |
|---|---|
| **Author** | Ayush Yadav |
| **Status** | v1.0 — prototype scope |
| **Related** | [User stories](user-stories.md) · [API contract](api-contract.yaml) · [Test scenarios](test-scenarios.md) |

## 1. Purpose & background

Orders reach the order desk as **unstructured text** (forwarded email, PDF, chat). Operators manually re-key them into the OMS. This is slow, inconsistent, and defers error detection to dispatch, where corrections are expensive. This document specifies an **AI-assisted intake** capability that extracts structured orders from raw text, scores field-level confidence, and routes low-confidence fields to a human before the order is created.

### Goals
- Reduce manual keying effort and transcription errors on order intake.
- Surface data-quality problems **at intake**, not at dispatch.
- Keep a human accountable for every order created (human-in-the-loop).

### Non-goals (this version)
- Automated fulfilment, inventory allocation, or invoicing.
- Multi-currency, tax, or pricing calculation.
- Authentication, roles, and permissions (assumed handled by the host platform).

## 2. Definitions
- **Confidence score** — model-reported likelihood (0–1) that an extracted field value is correct.
- **Confidence threshold** — configurable cut-off (default **0.80**) below which a field is flagged for human review.
- **Exception** — an order state indicating it requires attention before it can progress (e.g. created with unresolved fields).
- **Human-in-the-loop (HITL)** — operator confirmation/correction step before the extracted order becomes a record.

## 3. Functional requirements

### FR-1 Raw order capture
- **FR-1.1** The system shall provide a free-text input accepting pasted order text of arbitrary format.
- **FR-1.2** The system shall provide sample inputs representing common real-world formats (email, chat, structured PO).

### FR-2 Extraction
- **FR-2.1** On request, the system shall extract these fields from raw text: *customer, PO/reference, ship-to address, delivery date, order value, line items (qty + description)*.
- **FR-2.2** The system shall return a **confidence score (0–1)** for each extracted field.
- **FR-2.3** The system shall normalise dates to `YYYY-MM-DD`, including relative expressions (e.g. "Friday", "by 29 Jul").
- **FR-2.4** The system shall parse multiple line-item notations (bulleted, "3 x item", "25  item", "a + b").
- **FR-2.5** Where a field cannot be confidently determined, the system shall return an empty value with a low confidence score rather than a fabricated value.

### FR-3 Human-in-the-loop review
- **FR-3.1** The system shall display every extracted field with its confidence score.
- **FR-3.2** The system shall visually flag any field whose confidence is **below the threshold**.
- **FR-3.3** The system shall allow the operator to edit any field before creation.
- **FR-3.4** Editing a field shall mark it as human-verified (confidence = 1.0) and clear its flag.
- **FR-3.5** The system shall display a running count of fields still flagged for review.

### FR-4 Order creation
- **FR-4.1** The system shall require a non-empty **customer** to create an order.
- **FR-4.2** If any field remains below threshold at creation, the order shall be created in **Exception** state and flagged.
- **FR-4.3** If all fields are at/above threshold (or human-verified), the order shall be created in **New** state.
- **FR-4.4** Every order shall record a creation entry in its status history with a timestamp.

### FR-5 Order lifecycle
- **FR-5.1** Valid statuses: `New → Confirmed → Picking → Shipped → Delivered`, plus `Exception`.
- **FR-5.2** The system shall allow advancing an order one stage at a time.
- **FR-5.3** The system shall allow flagging any in-progress order as an Exception, and resolving an Exception back to Confirmed.
- **FR-5.4** Every status change shall append a timestamped entry to the order's history.

### FR-6 Order console
- **FR-6.1** The system shall list all orders with id, customer, item summary, value, ETA, and status.
- **FR-6.2** The system shall filter orders by status and search by id/customer/reference.
- **FR-6.3** The system shall show an order detail view including full field set, line items, and status timeline.
- **FR-6.4** The system shall display aggregate KPIs: open orders, orders needing review, open value, delivered count.

## 4. Business rules
- **BR-1** Confidence threshold default = **0.80**; treated as a configurable governance control (raising it increases review load and reduces error risk).
- **BR-2** An order is never created *silently* with sub-threshold data — it is either reviewed or created as an Exception.
- **BR-3** Missing **order value** or **delivery date** on a created order marks it for review (these block downstream dispatch).
- **BR-4** The extractor must never invent values (BR against hallucinated data) — empty + low confidence is the correct output for unknowns.

## 5. Assumptions & dependencies
- An extraction service (LLM/OCR) exposing the [documented API](api-contract.yaml) is available; the prototype simulates it locally.
- Customer master data and pricing exist elsewhere; validation against them is out of scope for v1.

## 6. Acceptance
This FRD is satisfied when all [user stories](user-stories.md) pass their acceptance criteria and the [test scenarios](test-scenarios.md) — including edge cases — behave as specified.
