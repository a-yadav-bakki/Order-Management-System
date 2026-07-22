# Test Scenario Library — Cargoflow OMS

Traceable test scenarios covering happy paths, business rules, and edge cases. IDs map back to the [user stories](user-stories.md) and [FRD](FRD.md). This is the kind of scenario library used to standardise UAT and catch regressions.

Legend: **P** = positive · **N** = negative/edge

| ID | Type | Scenario | Steps | Expected result | Traces |
|----|------|----------|-------|-----------------|--------|
| TS-01 | P | Extract from forwarded email | Load "Forwarded email" sample → Extract | All six fields populated; each shows a confidence score | US-A1, FR-2.1 |
| TS-02 | P | Low-confidence fields flagged | Load "Messy WhatsApp" sample → Extract | Value & delivery date flagged (below 80%); flagged count shown | US-A2, FR-3.2 |
| TS-03 | P | Edit clears flag | On a flagged field, type a corrected value | Field highlight clears; flagged count decreases by 1 | US-A3, FR-3.4 |
| TS-04 | N | No date present → not fabricated | Extract text with no delivery date | Delivery date empty, low confidence, flagged (not guessed) | US-A4, BR-4 |
| TS-05 | P | Create clean order | Resolve all fields → Create order | Order created as **New**; appears atop list; history has creation entry | US-B1, FR-4.3 |
| TS-06 | N | Create with unresolved fields | Leave ≥1 field below threshold → Create | Order created as **Exception** and flagged in list | US-B2, FR-4.2 |
| TS-07 | N | Missing customer blocks creation | Clear customer → Create | Creation blocked; "Customer is required" message | US-B3, FR-4.1 |
| TS-08 | P | Advance one stage | Open a Confirmed order → Advance | Status → Picking; timestamped history entry added | US-C1, FR-5.2 |
| TS-09 | P | Resolve an exception | Open an Exception order → Resolve | Status → Confirmed; resolution recorded in history | US-C2, FR-5.3 |
| TS-10 | P | Filter by status | Select "Exception" filter | Only exception orders listed; count updates | US-C3, FR-6.2 |
| TS-11 | P | KPIs recalculate | Create/advance orders | Open, needs-review, open-value, delivered KPIs update | US-C4, FR-6.4 |
| TS-12 | N | Empty input guarded | Click Extract with empty box | No extraction; prompt to paste order text | FR-1.1 |
| TS-13 | N | Line-item notation variety | Extract text mixing "3 x item", "25  item", "a + b" | All notations parsed into discrete line items | FR-2.4 |
| TS-14 | N | Relative date normalisation | Extract text with "Friday" / "by 29 Jul" | Date normalised to a concrete YYYY-MM-DD | FR-2.3 |
| TS-15 | N | Ambiguous customer from email only | Extract text where sender is only an email address | Customer inferred from domain **with reduced confidence** and flagged | FR-2.5 |
| TS-16 | P | Search finds order | Type a PO/reference into search | List narrows to matching orders | FR-6.2 |
| TS-17 | N | Flag then resolve round-trip | Flag a Picking order → then resolve | Exception recorded, then Confirmed; both in history | FR-5.3, FR-5.4 |
| TS-18 | P | Delivered is terminal | Advance an order to Delivered | No further advance offered; shown as completed | FR-5.1 |

## Edge cases explicitly designed for
- **Partial data** (missing value/date) — surfaced as review flags and forced Exceptions rather than silent bad records *(TS-04, TS-06)*.
- **Format variety** — bulleted, inline "qty x item", column-style, and "a + b" notations all parse *(TS-13)*.
- **Weak signals** — customer derivable only from an email domain returns a *low-confidence* guess, never a confident wrong answer *(TS-15)*.
- **Human override** — any auto value can be corrected, and correction is treated as ground truth *(TS-03)*.

## Suggested regression pack
Before any release touching intake or lifecycle, run: **TS-02, TS-04, TS-06, TS-07, TS-13, TS-14, TS-15** — these guard the data-quality rules that protect downstream dispatch.
