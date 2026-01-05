As a software architect, I want to develop a solution to reliably and securely send transactional emails in our system (for example: registration confirmations, password resets, receipts, and critical alerts).

Goal
 - Deliver reliable, auditable, and compliant transactional email delivery for users, with strong deliverability, bounce handling, and audit trails.

Assumptions
 - Emails may contain PII/PHI. Treat content as sensitive until confirmed otherwise.
 - Primary hosting is in Azure; prefer Azure-native services where they match requirements.
 - Expected volume is not yet provided; design for configurable scale (small → large) and provider fallback.

Recommendation (SendGrid-focused)
 - Use a queued, worker-based send pipeline integrated with Twilio SendGrid for transactional delivery.
 - Offload deliverability, DKIM/SPF management and webhook events to SendGrid while keeping an internal canonical suppression list and audit store.
 - Require a BAA before sending PHI; if a BAA cannot be obtained, avoid sending PHI through the provider.

High-level Architecture
 - Producer (App/API) -> enqueue `SendRequest` -> `Azure Service Bus` -> Email Worker (App Service / Function / Container) -> SendGrid API -> SendGrid Event Webhook -> Webhook Processor -> Audit Store & Suppression List.
 - Key components: canonical `SendRequest` format, provider adapter layer, templating (repo-managed or SendGrid templates), webhook verification & processing, audit & suppression stores, telemetry and alerting.

Azure Integration Notes
 - Queue: use `Azure Service Bus` for durability, ordering and DLQ support.
 - Workers: run in a private subnet (App Service with VNet integration, Container Apps, or AKS) using Managed Identity to access `Key Vault`, Service Bus and Storage.
 - Networking: use Private Endpoints for Key Vault, Storage and Service Bus; permit restricted outbound TLS egress to SendGrid endpoints.

Security & Compliance
 - BAA: do not send PHI until a BAA is in place with SendGrid/Twilio.
 - Secrets: store API keys in `Key Vault`; use Managed Identities for access.
 - Payload minimization: prefer references/tokens to PHI, not raw PHI in email bodies or provider payloads.
 - Webhooks: verify SendGrid signatures and timestamps; protect against replay attacks.
 - Logging: redact PII/PHI from logs; persist immutable audit records in an access-controlled store.

Deliverability
 - Use a dedicated sending subdomain (e.g., `email.example.com`) and configure SPF, DKIM and DMARC per SendGrid instructions.
 - For high volume, request dedicated IP(s) and implement warm-up schedules.

Operational Patterns
 - Idempotency: include `send_id` per logical send to avoid duplicates across retries or failover.
 - Retries & DLQ: implement exponential backoff for API errors and DLQ for poison messages.
 - Suppression: maintain internal suppression list synchronized with SendGrid events; never re-send to hard-bounced or complained addresses.
 - Fallback: design a small provider-adapter to allow swapping/failing over to an alternate provider (e.g., SES) while preserving idempotency.

Observability
 - Capture metrics: sent, delivered, bounced, complaint rates, queue depth, worker failures, webhook failures.
 - Tracing: propagate correlation IDs through queue → worker → provider → webhook events.
 - Alerts: bounce/complaint spikes, webhook signature failures, queue backlog thresholds.

Next Steps
 - Confirm open questions: volume estimates, PHI presence, approved providers, template ownership, and whether a dedicated IP is required.
 - Create an ADR capturing the SendGrid decision and compliance prerequisites (BAA requirement).
 - Optionally prototype a minimal worker (Service Bus trigger) + SendGrid adapter and webhook processor.

Open Questions
 - What are expected daily and peak email volumes?
 - Will email content include PHI/PII or only references to protected data?
 - Which Azure worker platform is preferred (Functions, App Service, Container Apps, AKS)?
 - Do we require provider-dedicated IPs at launch?

References
 - SendGrid documentation for domain authentication, event webhooks, and API usage.
