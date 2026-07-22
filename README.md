# Cargoflow OMS — AI-assisted Order Management Console

> A portfolio case study by **Ayush Yadav** — Senior Business Analyst / Product Owner
> Demonstrating the full path from an ambiguous operational problem to a working, testable product: discovery → requirements → API contract → prototype → test scenarios.

**[▶ Live prototype](https://a-yadav-bakki.github.io/Order-Management-System/)** · **[FRD](FRD.md)** · **[User stories](user-stories.md)** · **[Process flows](process-flows.md)** · **[API contract](api-contract.yaml)** · **[Test scenarios](test-scenarios.md)**

---

## Why this exists

Most portfolios show *either* a screenshot *or* a document. This repo shows the **thinking that connects them** — the artifacts a BA/PO actually produces, ending in a clickable prototype you can validate against the acceptance criteria.

The problem is drawn from real logistics operations: **orders arrive as unstructured text** — forwarded emails, PDFs, WhatsApp messages — and a human re-keys them into the OMS. This is slow and error-prone, and mistakes surface late (a missing delivery date only becomes a problem at dispatch).

## The product thesis

> Let operators paste raw order text and have the system extract the structured order, **scoring each field's confidence** and routing only the uncertain fields to a human. Certain fields flow straight through; uncertain ones are flagged before they become downstream exceptions.

This is a **human-in-the-loop** design: AI does the heavy lifting, the operator owns the judgment. The confidence threshold is the product's core control — set it high and you review more but ship fewer errors; set it low and you move faster but risk bad data reaching dispatch.

## Who it's for

| Persona | Goal | Pain today |
|---|---|---|
| **Order desk operator** | Enter orders fast and correctly | Re-keying from messy sources; typos surface late |
| **Operations lead** | Keep orders moving; catch exceptions early | No visibility into which orders are risky until dispatch |
| **Customer** | Order gets fulfilled on time | Errors cause delays and reorders |

## What the prototype does

- **AI-assisted intake (modal)** — click **+ Add new order**, paste any order text (three realistic samples included), and extract structured fields in a pop-up.
- **Confidence scoring + human-in-the-loop** — every field gets a confidence score; anything below the **80% threshold** is highlighted for review, and the order can't be silently created with bad data.
- **Inline edit for New orders** — open a New order and use the edit (✎) control to correct fields inline; confirming records an **"Edited"** event in the status timeline.
- **Partial approval flow** — for a New order, tick the line items you can fulfil and set an **Available Qty** against the **Ordered Qty**. Unticked lines are marked **Rejected**; confirming applies the approval and moves the order to Confirmed (or to Exception if nothing can be fulfilled).
- **Tabbed order detail** — each order opens in a side panel with **Order details** (fields, items, lifecycle actions) and **Status timeline** tabs.
- **Proof of Delivery (POD)** — a **Shipped** order can't skip straight to Delivered; the operator uploads a POD/receiving document (image or PDF), records who received it and when, and only then does the order move to **Delivered** with the POD stored and shown on the order.
- **Item master & item-wise value** — a separate **Item master** tab holds a catalogue of items and their value per unit. Order value is calculated line by line (value per unit × quantity), and the price is snapshotted onto the order at creation.
- **Value tracking across the flow** — each order tracks **Ordered → Available → Delivered** value. If 10 are ordered, 9 approved and 8 received, the effective value steps down accordingly, so the console and KPIs always reflect what will actually be invoiced.
- **Order lifecycle** — New → Confirmed → Picking → Shipped → Delivered (POD-gated), plus **Exception** (returns to its prior status on resolve) and **Not Available** (nothing fulfillable).
- **Order console** — filter by status, search, and live KPIs (open orders, needs review, open value, delivered).

### Deliberate design decisions (the BA parts)
- **Confidence threshold as a governance lever**, not a hidden constant — documented in the [FRD](FRD.md) with rationale.
- **Orders created with unresolved fields become Exceptions, not silent records** — bad data is made *visible*, not blocked, because operators sometimes must proceed on partial info.
- **Every status change is timestamped into a history** — auditability is a first-class requirement, not an afterthought.

> This mirrors real AI/OCR delivery work: extraction requirements, confidence-threshold logic, human-in-the-loop verification, and exception handling.

## How the "AI" works here

The extractor is a **deterministic on-device heuristic parser** (no API key, nothing leaves the browser) that simulates an LLM/OCR extraction service. That keeps the repo free to clone and run, while the *product logic being demonstrated* — confidence scoring, thresholds, review routing, exception creation — is exactly what a production extraction pipeline would need. The [API contract](api-contract.yaml) specifies the real service this would call.

## Run it

Just open `index.html` in a browser — no build step, no dependencies. To host: enable **GitHub Pages** on the `main` branch (root) and share the link.

## Repo map

```
├── index.html              # Working prototype (self-contained)
├── README.md               # This case study
└── docs/
    ├── FRD.md              # Functional Requirements Document
    ├── user-stories.md     # Stories + Gherkin acceptance criteria
    ├── process-flows.md    # Intake + lifecycle flows (Mermaid)
    ├── prioritization-moscow.md  # MoSCoW scope + Value/Effort
    ├── api-contract.yaml   # OpenAPI 3.0 spec for the extraction + orders API
    └── test-scenarios.md   # Test scenario library incl. edge cases
```

## What this demonstrates about how I work

1. **I start from the operational reality**, not the feature list — the design comes from how orders actually arrive.
2. **I make requirements testable** — every story has Gherkin criteria that map to the [test scenarios](test-scenarios.md).
3. **I can speak the technical layer** — the [OpenAPI contract](api-contract.yaml) defines payloads, confidence fields, and error handling engineers can build against.
4. **I design for edge cases up front** — missing values, ambiguous customers, and partial orders are handled by design, not patched later.

---

*Prototype for demonstration. Sample data is fictional.*
