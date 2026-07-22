# Process Flows — Cargoflow OMS

Diagrams use Mermaid, which renders natively on GitHub.

## 1. AI-assisted intake (happy path + review loop)

```mermaid
flowchart TD
    A([Operator pastes raw order text]) --> B[Click 'Extract order fields']
    B --> C[Extraction service parses text]
    C --> D[Return fields + confidence per field]
    D --> E{Any field below<br/>80% threshold?}
    E -- No --> F[All fields shown as confident]
    E -- Yes --> G[Flag low-confidence fields for review]
    G --> H[Operator reviews / edits flagged fields]
    H --> I{Customer present?}
    F --> I
    I -- No --> J[Block creation:<br/>'Customer required']
    J --> H
    I -- Yes --> K{Any field still<br/>below threshold?}
    K -- Yes --> L[Create order as EXCEPTION + flag]
    K -- No --> M[Create order as NEW]
    L --> N([Order in console])
    M --> N
```

## 2. Order lifecycle state machine

```mermaid
stateDiagram-v2
    [*] --> New: created (all fields ok)
    [*] --> Exception: created (unresolved fields)
    New --> Confirmed: advance
    Confirmed --> Picking: advance
    Picking --> Shipped: advance
    Shipped --> Delivered: record POD (upload + receive)
    Delivered --> [*]

    New --> Exception: flag
    Confirmed --> Exception: flag
    Picking --> Exception: flag
    Shipped --> Exception: flag
    Exception --> Confirmed: resolve
```

## 3. Confidence-threshold decision (the core control)

```mermaid
flowchart LR
    F[Field extracted<br/>with confidence c] --> T{c >= threshold?}
    T -- Yes --> P[Pass through:<br/>no action needed]
    T -- No --> R[Route to human:<br/>highlight + 'review']
    R --> E[Operator edits] --> V[Verified:<br/>c set to 1.0]
    R --> A[Operator accepts as-is] --> X[Remains sub-threshold]
    V --> OK[Eligible for clean create]
    X --> EXC[Forces Exception on create]
```

## 4. Roles & ownership (swimlane view)

```mermaid
flowchart TD
    subgraph Operator
      O1[Paste + extract] --> O2[Review flagged fields] --> O3[Create order] --> O4[Advance status]
    end
    subgraph System
      S1[Extract + score fields] --> S2[Apply threshold + flag] --> S3[Enforce create rules] --> S4[Record history + KPIs]
    end
    subgraph OpsLead
      L1[Monitor 'needs review' KPI] --> L2[Work exceptions to closure]
    end
    O1 --> S1
    O2 --> S2
    O3 --> S3
    O4 --> S4
    S4 --> L1
```

## Notes on the design
- The **threshold decision (Flow 3)** is the heart of the product: it's where AI speed and human accountability meet. Documented as a configurable business rule (BR-1) so operations can tune the speed/quality trade-off.
- The lifecycle (**Flow 2**) keeps `Exception` reachable from every active state, because problems can surface at any point, and always recoverable via `resolve`.
