# Process Flows — Cargoflow OMS

Mermaid diagrams (render natively on GitHub).

## 1. Order lifecycle state machine

```mermaid
stateDiagram-v2
    [*] --> New: created (all essentials present)
    [*] --> Exception: created (missing date/value)
    New --> Confirmed: partial approval (>=1 item)
    New --> NotAvailable: confirm with no items selected
    Confirmed --> Picking: advance
    Picking --> Shipped: advance (tracking created)
    Shipped --> Arrived: advance (reached destination)
    Arrived --> Delivered: record POD (upload + received qty)
    Delivered --> [*]
    NotAvailable --> [*]

    Confirmed --> Exception: flag (+remarks)
    Picking --> Exception: flag (+remarks)
    Shipped --> Exception: flag (+remarks)
    Arrived --> Exception: flag (+remarks)
    New --> Exception: flag (+remarks)
    Exception --> Confirmed: resolve (returns to prior stage)
    note right of Exception
      Resolve returns the order to the
      stage it was flagged from, not a fixed stage.
    end note
```

## 2. Add order — form + optional AI extract

```mermaid
flowchart TD
    A([Open Add order]) --> B{Paste order text?}
    B -- Yes --> C[Extract fields] --> D[Pre-fill form + flag low-confidence]
    B -- No --> E[Fill form manually]
    D --> F[Item rows]
    E --> F
    F --> G{Item in master?}
    G -- Yes --> H[Auto-fill value/unit]
    G -- No --> I[Flag row + 'Add to master' -> needs value/unit]
    H --> J[Live order value = Σ unit × qty]
    I --> J
    J --> K{Customer + >=1 item?}
    K -- No --> L[Block create]
    K -- Yes --> M{Date & value present?}
    M -- No --> N[Create as Exception]
    M -- Yes --> O[Create as New]
```

## 3. Partial approval (New order)

```mermaid
flowchart TD
    A([New order]) --> B[Per item: available? ordered qty / available qty]
    B --> C{Available <= Ordered for all ticked?}
    C -- No --> D[Block: 'Available Qty cannot be more than Ordered Qty']
    C -- Yes --> E{Any item selected?}
    E -- No --> F[Warn: will move to Not Available]
    F --> G{Confirm again?}
    G -- Yes --> H[Not Available: all rejected]
    E -- Yes --> I[Unticked -> Rejected; ticked keep available qty]
    I --> J[Confirmed]
```

## 4. Shipping → arrival → delivery (POD gated)

```mermaid
flowchart LR
    S[Shipped<br/>in transit] --> A[Arrived<br/>at destination]
    A --> P{POD recorded?}
    P -- Upload doc + received qty --> V{Received <= Available?<br/>Discrepancies have remarks?}
    V -- No --> P
    V -- Yes --> D[Delivered<br/>trip ended]
    D --> E[Edit POD?<br/>logs 'POD corrected']
    note1[POD capture is unavailable while Shipped/in-transit] -.-> S
```

## 5. Value tracking across stages

```mermaid
flowchart LR
    O[Ordered value<br/>Σ unit × ordered] --> Av[Available value<br/>Σ unit × available<br/>after approval]
    Av --> De[Delivered value<br/>Σ unit × received<br/>after POD]
    note[Effective value shown per stage;<br/>console strikes through the original when reduced] -.-> Av
```

## Notes
- The **Arrived** stage exists specifically so proof of delivery is only capturable once the shipment has physically reached the destination (Flow 4).
- **Exception** is reachable from every active stage and always returns to the stage it came from (Flow 1), so flagging never loses an order's place.
- Prices are snapshotted at creation; value then only *narrows* through approval and delivery (Flow 5).
