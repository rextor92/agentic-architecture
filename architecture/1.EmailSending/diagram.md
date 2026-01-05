# Email Sending Flow (Mermaid)

Below is a mermaid diagram that illustrates the transactional email flow using Azure + SendGrid. Save or preview this file in a mermaid-capable renderer (VS Code Mermaid preview, MkDocs plugin, or GitHub rendered where supported).

```mermaid
flowchart LR
  %% Producer -> Queue -> Worker -> Provider -> Webhook -> Processor -> Stores

  subgraph AZ[Azure Environment]
    direction TB
    A[Producer App / API] -->|Enqueue `SendRequest`| B[Azure Service Bus Queue]
    B -->|Deliver message| C[Email Worker(s)]
    C -->|Get API key via Managed Identity| KV[Azure Key Vault]
    C -->|Render template (repo or provider)| T[Template Store]
    C -->|POST /mail/send (TLS)| SG[SendGrid API]
    SG -->|Event webhook (bounce/complaint/delivered)| WH[SendGrid Webhook Delivery]
  end

  WH -->|Signed webhook, verify| WP[Webhook Processor]
  WP -->|Persist immutable event| AS[Audit Store (append-only)]
  WP -->|Update| SL[Suppression / Bounce DB]
  C -->|Check & update| SL

  C -->|Emit metrics/traces| OM[Observability (Metrics/Logs/Tracing)]
  AS -->|Access controls & retention| Sec[Security & Compliance]

  %% Optional fallback provider
  C -->|Failover adapter| ALT[Secondary Provider (e.g., SES)]
  ALT -.-> SG

  %% Network hints
  KV -. Private Endpoint .-> C
  B -. Private Endpoint .-> C
  AS -. Private Endpoint .-> WP

  classDef azure fill:#f0f8ff,stroke:#0366d6,stroke-width:1px;
  class AZ,KV,B,AS azure
```
