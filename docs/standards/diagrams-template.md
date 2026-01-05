# docs/standards/diagrams-template.mmd
# Mermaid diagram templates for healthcare-grade architecture docs
# Usage:
# - Copy the relevant diagram blocks into:
#   - architecture/<feature>/diagrams/*.mmd  (recommended), or
#   - architecture/<feature>/overview.md / threat-model.md (inline mermaid blocks)
#
# Conventions:
# - Treat PHI/PII flows explicitly and label them.
# - Mark trust boundaries clearly.
# - Identify ingress/egress points.
# - Prefer private endpoints where applicable.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% 1) C4 Context Diagram (System Context)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% Paste into a Mermaid block:
%% ```mermaid
%% (content)
%% ```
%%
%% Notes:
%% - C4-style diagrams in Mermaid are often approximations; keep them simple.
%% - Use boundaries to show hospital network, cloud tenant, partners.
%% - Label PHI flows.
%%
flowchart LR
  %% Actors
  Patient([Patient])
  Clinician([Clinician])
  Admin([IT Admin])

  %% Trust boundaries
  subgraph TB1["Trust Boundary: Public Internet"]
  end

  subgraph TB2["Trust Boundary: Healthcare Org (On-Prem / Hospital Network)"]
    EHR[EHR/EMR System]
    AD[On-prem Directory / Legacy IAM]
    Devices[Medical Devices / IoT (optional)]
  end

  subgraph TB3["Trust Boundary: Azure Tenant (Production)"]
    Ingress[Ingress (WAF / App Gateway / Front Door)]
    App[Core Application / APIs]
    Data[(Primary Data Store)]
    FHIR[(FHIR Store / Health Data Platform)]
    Msg[Messaging/Eventing]
    KV[Key Vault]
    Obs[Logging/Monitoring/SIEM]
  end

  subgraph TB4["Trust Boundary: External Partners"]
    Lab[Lab Partner]
    Payer[Payer/Insurer]
    HIE[HIE / National Exchange]
  end

  %% Interactions
  Patient -->|Uses| Ingress
  Clinician -->|Uses| Ingress
  Admin -->|Administers| App

  EHR <-->|HL7/FHIR (PHI)| App
  Devices -->|Telemetry (may include PHI)| App

  App -->|Reads/Writes (PHI)| Data
  App -->|FHIR resources (PHI)| FHIR
  App --> Msg
  App --> KV
  App -->|Metrics/Logs (NO PHI)| Obs

  App <-->|Orders/Results (PHI)| Lab
  App <-->|Eligibility/Claims (PHI)| Payer
  App <-->|Exchange (PHI)| HIE


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% 2) C4 Container Diagram (Containers/Services inside the system)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
flowchart TB
  subgraph TB_A["Azure Tenant: Prod Subscription"]
    subgraph TB_NET["VNet + Private Networking"]
      PE1[Private Endpoint: Data]
      PE2[Private Endpoint: Key Vault]
      PE3[Private Endpoint: FHIR (if applicable)]
      Egress[Controlled Egress (Firewall/NVA)]
    end

    subgraph TB_EDGE["Edge/Ingress"]
      WAF[WAF / App Gateway / Front Door]
      APIM[API Management (optional)]
    end

    subgraph TB_APP["App Layer"]
      FE[Web/UI (optional)]
      API[API Service]
      Worker[Background Worker]
      Integrations[Integration Service (HL7/FHIR/partner)]
    end

    subgraph TB_DATA["Data Layer"]
      DB[(DB: SQL/Postgres/Cosmos - per stack)]
      Cache[(Cache)]
      Bus[(Service Bus / Eventing)]
      FHIRS[(FHIR Store)]
      Blob[(Object Storage)]
    end

    subgraph TB_SEC["Security & Ops"]
      Entra[Entra ID]
      MI[Managed Identities / Workload Identity]
      KV2[Key Vault]
      Policy[Azure Policy/Blueprints]
      Monitor[Monitor/App Insights/Log Analytics]
      SIEM[Sentinel/SIEM]
    end
  end

  %% Flows
  WAF --> APIM --> API
  FE --> WAF
  API --> DB
  API --> Cache
  API --> Bus
  Worker --> Bus
  Worker --> DB
  Integrations --> Bus
  Integrations --> FHIRS
  API --> FHIRS
  API --> Blob

  API --> KV2
  Worker --> KV2
  API --> Monitor
  Worker --> Monitor
  Monitor --> SIEM

  Entra --> APIM
  Entra --> API
  MI --> API
  MI --> Worker


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% 3) Data Flow Diagram (DFD) with PHI labeling (Level 1)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% DFD tips:
%% - Label data types (PHI/PII) on edges.
%% - Identify stores and processes.
%% - Note redaction rules for logs.
%%
flowchart LR
  %% External entities
  User([Clinician/Patient])
  Partner([External Partner])

  %% Processes
  P1((P1: Authenticate))
  P2((P2: API Request Handling))
  P3((P3: Business Processing))
  P4((P4: Integration/Exchange))
  P5((P5: Telemetry/Observability))

  %% Data stores
  D1[(D1: Patient DB - PHI)]
  D2[(D2: FHIR Store - PHI)]
  D3[(D3: Queue/Event Bus - may contain PHI)]
  D4[(D4: Audit Log - NO PHI)]
  D5[(D5: Secrets Vault)]

  %% Flows
  User -->|Credentials/Tokens| P1
  P1 -->|Access Token| P2

  User -->|Request (may contain PHI)| P2
  P2 -->|Validated Request| P3

  P3 -->|Read/Write PHI| D1
  P3 -->|FHIR Resources (PHI)| D2
  P3 -->|Event (PHI? minimize)| D3

  P3 -->|Outbound (PHI)| P4
  Partner <-->|Exchange (PHI)| P4

  P2 -->|Audit events (NO PHI)| D4
  P3 -->|Audit events (NO PHI)| D4

  P2 -->|Secrets/Keys| D5
  P3 -->|Secrets/Keys| D5

  P2 -->|Metrics/Traces (NO PHI)| P5
  P3 -->|Metrics/Traces (NO PHI)| P5


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% 4) Trust Boundary Diagram (simple + explicit)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
flowchart LR
  subgraph Z0["Trust Boundary 0: Untrusted (Internet)"]
    U([User Devices])
  end

  subgraph Z1["Trust Boundary 1: Edge (Protected)"]
    Edge[WAF / Ingress]
  end

  subgraph Z2["Trust Boundary 2: Application Network (Private)"]
    App[Services / APIs]
    Bus[Messaging]
  end

  subgraph Z3["Trust Boundary 3: Data Network (Highly Restricted)"]
    DB[(DB / FHIR / Storage)]
    KV[(Key Vault / HSM)]
  end

  subgraph Z4["Trust Boundary 4: Operations & Security"]
    Mon[Monitoring]
    SIEM[SIEM/Sentinel]
    Admin([Admin Workstations/PIM])
  end

  %% Connections
  U -->|TLS| Edge
  Edge -->|mTLS/TLS + AuthZ| App
  App -->|Private Link| DB
  App -->|Private Link| KV
  App -->|Telemetry (NO PHI)| Mon
  Mon --> SIEM
  Admin -->|Privileged access| App
  Admin -->|Privileged access| DB


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% 5) Sequence Diagram Template (Auth + API + Data) with PHI note
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
sequenceDiagram
  autonumber
  participant User as User (Clinician/Patient)
  participant IdP as Entra ID (IdP)
  participant Edge as Edge/WAF/APIM
  participant API as API Service
  participant DB as Data Store (PHI)
  participant Audit as Audit Log (NO PHI)

  User->>IdP: Authenticate (MFA/CA)
  IdP-->>User: Access token
  User->>Edge: Request + token (may include PHI)
  Edge->>API: Forward request (validated headers)
  API->>Audit: Write audit event (NO PHI)
  API->>DB: Read/Write PHI (encrypted)
  DB-->>API: Result
  API-->>User: Response (min PHI; least data)


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%% 6) Threat Model Checklist Diagram (optional visual)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
flowchart TD
  A[Define Scope] --> B[Identify Assets & Actors]
  B --> C[Draw Trust Boundaries]
  C --> D[Enumerate Entry Points]
  D --> E[STRIDE Threats]
  E --> F[Rank Risks]
  F --> G[Mitigations + Backlog]
  G --> H[Validation Plan]
  H --> I[Residual Risk + Sign-off]
