# Test Scenario Library — Cargoflow OMS

Traceable scenarios covering happy paths, business rules and edge cases. IDs map to the [user stories](user-stories.md) and [FRD](FRD.md).

Legend: **P** = positive · **N** = negative/edge

| ID | Type | Scenario | Steps | Expected result | Traces |
|----|------|----------|-------|-----------------|--------|
| TS-01 | P | AI extract pre-fills form | Add order → paste sample → Extract | Customer, reference, ship-to, date, items filled; low-confidence fields highlighted | US-A1, FR-1.2 |
| TS-02 | P | Manual add | Add order → fill fields + items by hand → Create | Order created without using extraction | US-A1, FR-1.1 |
| TS-03 | P | Item smart search | Focus item field → type part of a name → pick a result | List filters to master items; name + value/unit fill | US-A2, FR-2.1 |
| TS-04 | N | Item not in master | Extract/enter an item absent from master | Row flagged "not in master"; "Add to master" needs a value/unit; adding without one is blocked | US-A3, FR-2.3 |
| TS-05 | P | Item-wise value | Set quantities and values per unit | Order value = Σ (unit × qty); updates live | US-A4, FR-3.1 |
| TS-06 | P | Clean create | Customer + items + date + value → Create | Order created as **New** | US-B1, FR-4.1 |
| TS-07 | N | Missing essentials | Omit delivery date or value → Create | Order created as **Exception**, flagged | US-B2, FR-4.2 |
| TS-08 | P | Partial approval | New order → untick one item, set available on another → Confirm | Unticked → Rejected; ticked keep available qty; status **Confirmed** | US-C1, FR-5.3 |
| TS-09 | N | Available > Ordered | Set available qty above ordered → Confirm | Blocked: "Available Qty cannot be more than Ordered Qty." | US-C2, FR-5.2 |
| TS-10 | N | No items selected | Untick all → Confirm | Warned it will move to **Not Available**; second confirm applies it | US-C3, FR-5.4 |
| TS-11 | P | Edit New order | Open New order → edit → confirm | Fields saved; **"Edited"** event in timeline | US-D1, FR-9.3 |
| TS-12 | P | Flag & resolve exception | Flag a Picking order (remarks) → Resolve | Status Exception with remarks; resolve returns to **Picking** | US-D2, FR-L4 |
| TS-13 | P | Value narrows | 10 ordered → 9 available → 8 received | Ordered/Available/Delivered values shown; console strikes through original | US-D3, FR-6 |
| TS-14 | P | In-transit tracking | Open a Shipped order | Map shows route + in-transit position; carrier/tracking/ETA shown | US-E1, FR-7 |
| TS-15 | N | POD hidden in transit | Open a Shipped order | **No POD upload** is available | US-E2, FR-8.1 |
| TS-16 | P | POD appears on arrival | Advance Shipped → Arrived | Map shows shipment at destination; **POD capture appears** | US-E2, FR-L2 |
| TS-17 | P | Record delivery | Arrived → upload POD + received qty → Mark delivered | Status **Delivered**; POD stored and shown | US-E3, FR-8.5 |
| TS-18 | N | Received > Available / no remarks | Received qty above available, or short with no remarks | Blocked: "Received Qty cannot be more than Available Qty." / "Add remarks for rejected or short items." | US-E3, FR-8.3, FR-8.4 |
| TS-19 | P | Edit submitted POD | Delivered order → edit POD → change received/details → Save | Same validation; **"POD corrected"** event; delivered value recalculates | US-E4, FR-8.6 |
| TS-20 | P | Filter, search, KPIs | Apply a status filter / search | List narrows; KPIs reflect current state | US-F1, FR-9.1 |
| TS-21 | P | Full-page detail | Open an order | Opens full page with Order details / Status timeline tabs; back returns to console | US-F2, FR-9.3 |
| TS-22 | N | Add-to-master reuse | Add an unknown item to master, then start another order | The item is now searchable in the master | US-A3, FR-2.4 |

## Edge cases explicitly designed for
- **Two rejection points** — approval (what we agreed to supply) and delivery (what actually arrived); both require remarks *(TS-08, TS-18)*.
- **Value integrity** — reductions at approval and delivery flow through to the effective value *(TS-13)*.
- **POD timing** — proof of delivery cannot be recorded before the shipment arrives *(TS-15, TS-16)*.
- **No fabricated data** — unknown extracted fields stay empty; unknown items are flagged, not silently priced at zero *(TS-04)*.
- **Auditable corrections** — POD edits are permitted but logged *(TS-19)*.

## Suggested regression pack
Before any release touching intake, approval, delivery or value: **TS-04, TS-07, TS-09, TS-10, TS-15, TS-16, TS-18, TS-19** — these guard the data-quality, POD-timing and value rules that protect downstream dispatch and invoicing.
