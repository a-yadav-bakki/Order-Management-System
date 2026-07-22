# User Stories & Acceptance Criteria — Cargoflow OMS

Stories are grouped by epic. Each has Gherkin-style acceptance criteria that map directly to the [test scenarios](test-scenarios.md) and the [FRD](FRD.md).

---

## Epic A — AI-assisted intake

### US-A1 · Extract a structured order from raw text
**As an** order desk operator
**I want** to paste raw order text and have the fields extracted automatically
**So that** I don't have to re-key every order by hand.

```gherkin
Scenario: Extract from a forwarded email
  Given I have pasted a forwarded order email into the intake box
  When I click "Extract order fields"
  Then I see customer, PO/reference, ship-to, delivery date, value and line items populated
  And each field shows a confidence score
```
*Maps to FR-2.1, FR-2.2 · TS-01*

### US-A2 · See which fields to trust
**As an** operator
**I want** low-confidence fields clearly flagged
**So that** I know exactly what to check instead of re-reading everything.

```gherkin
Scenario: Low-confidence fields are flagged
  Given an extraction has completed
  When any field's confidence is below the 80% threshold
  Then that field is visually highlighted and marked "review"
  And the number of flagged fields is shown to me
```
*Maps to FR-3.1, FR-3.2, FR-3.5 · TS-02*

### US-A3 · Correct before creating
**As an** operator
**I want** to edit any extracted field
**So that** I can fix or confirm values before the order is created.

```gherkin
Scenario: Editing a flagged field clears the flag
  Given a field is flagged for review
  When I edit that field's value
  Then the field is treated as human-verified
  And its review flag is removed
  And the flagged-field count decreases
```
*Maps to FR-3.3, FR-3.4 · TS-03*

### US-A4 · Don't fabricate data
**As an** operations lead
**I want** the system to leave a field blank when it isn't sure
**So that** we never ship on invented data.

```gherkin
Scenario: Unknown field is left empty, not guessed
  Given the raw text contains no delivery date
  When extraction runs
  Then the delivery date is empty
  And it carries a low confidence score and a review flag
```
*Maps to FR-2.5, BR-4 · TS-04*

---

## Epic B — Order creation & data quality

### US-B1 · Create a clean order
**As an** operator
**I want** to create an order once fields are confirmed
**So that** it enters the workflow ready to fulfil.

```gherkin
Scenario: Create with all fields resolved
  Given all extracted fields are at or above threshold or human-verified
  When I click "Create order"
  Then a new order is created in "New" status
  And it appears at the top of the orders list
```
*Maps to FR-4.3, FR-4.4 · TS-05*

### US-B2 · Partial orders are made visible
**As an** operations lead
**I want** orders created with unresolved fields to be flagged
**So that** risky orders are caught before dispatch, not after.

```gherkin
Scenario: Create with unresolved fields
  Given at least one field is still below threshold
  When I click "Create order"
  Then the order is created in "Exception" status
  And it is marked as needing review in the list
```
*Maps to FR-4.2, BR-2 · TS-06*

### US-B3 · Customer is mandatory
**As an** operator
**I want** to be stopped if there's no customer
**So that** we never create an unattributable order.

```gherkin
Scenario: Block creation without a customer
  Given the customer field is empty
  When I click "Create order"
  Then creation is blocked
  And I am told the customer is required
```
*Maps to FR-4.1 · TS-07*

---

## Epic C — Lifecycle & console

### US-C1 · Move an order forward
**As an** operator
**I want** to advance an order through its stages
**So that** its status reflects reality.

```gherkin
Scenario: Advance one stage
  Given an order is in "Confirmed"
  When I advance it
  Then its status becomes "Picking"
  And a timestamped entry is added to its history
```
*Maps to FR-5.2, FR-5.4 · TS-08*

### US-C2 · Handle and resolve exceptions
**As an** operations lead
**I want** to flag and later resolve exceptions
**So that** problem orders are tracked to closure.

```gherkin
Scenario: Resolve an exception
  Given an order is in "Exception"
  When I resolve it
  Then its status becomes "Confirmed"
  And the resolution is recorded in history
```
*Maps to FR-5.3, FR-5.4 · TS-09*

### US-C3 · Find orders fast
**As an** operator
**I want** to filter and search the order list
**So that** I can find what I need across many active orders.

```gherkin
Scenario: Filter by status
  Given orders exist in several statuses
  When I select the "Exception" filter
  Then only exception orders are listed
  And the count updates accordingly
```
*Maps to FR-6.2 · TS-10*

### US-C4 · See the whole picture at a glance
**As an** operations lead
**I want** KPIs and a status timeline per order
**So that** I can gauge load and audit history quickly.

```gherkin
Scenario: KPIs reflect current state
  Given orders change status or are created
  When the board updates
  Then the open, needs-review, open-value and delivered KPIs recalculate
```
*Maps to FR-6.3, FR-6.4 · TS-11*
