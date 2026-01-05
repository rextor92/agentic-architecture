# Handoff: EmailSending (SendGrid)

**Purpose**
This document summarizes the EmailSending work so another engineer can continue implementation and run the next spikes/pilots.

**Current Decision**
- **Decision:** Use Twilio SendGrid as the primary transactional email provider (see ADR: `architecture/1.EmailSending/adr/0001-choose-sendgrid.md`).
 - **Important constraint & legal status:** Legal confirmed on 2026-01-05 that a Business Associate Agreement (BAA) is required if emails contain PHI. Current scope: transactional emails do NOT contain PHI, so a BAA is not required for initial sends.

**Key Artifacts**
- **Overview:** `architecture/1.EmailSending/overview.md`
- **ADR:** `architecture/1.EmailSending/adr/0001-choose-sendgrid.md`
- **Considered approaches:** `architecture/1.EmailSending/considered-approaches.md`
- **Open questions & next steps:** `architecture/1.EmailSending/open-questions.md`

**Architecture Summary (high level)**
- **Producer:** App/API pushes canonical `SendRequest` messages to an Azure Service Bus queue.
- **Queue:** `Azure Service Bus` (durable, DLQ, ordered processing where needed).
- **Worker:** Stateless worker (App Service / Function / Container) reads queue, renders template (repo or provider), calls SendGrid API via adapter, persists provider message id and status.
- **Webhook Processor:** HTTPS endpoint that verifies SendGrid signatures, persists raw events to the audit store, and updates the suppression list.
- **Stores:** Internal audit store (append-only), suppression/bounce DB, and template store (repo-managed or SendGrid-managed).
- **Secrets & Identity:** SendGrid API keys in `Azure Key Vault`; workers use Managed Identity for Key Vault + Service Bus access.
- **Networking:** Private Endpoints for Key Vault, Service Bus, Storage; restricted egress NAT/firewall allowing TLS to SendGrid endpoints.

**Canonical SendRequest (example)**
```json
{
  "send_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "tenant_id": "tenant-123",
  "template_id": "welcome_user_v1",
  "to": [{ "email": "user@example.com", "name": "Jane User" }],
  "from": { "email": "no-reply@email.example.com", "name": "Example" },
  "data": { "displayName": "Jane", "link": "https://example.com/confirm?token=..." },
  "priority": "normal",
  "trace_id": "req-abc-123",
  "sensitive": false
}
```
 - **Notes:** `send_id` is required for idempotency. Set `sensitive=true` only if the payload may contain PHI. Current scope: `sensitive` should be `false` for initial sends; do not set `sensitive=true` unless Legal confirms BAA or scope changes.

**Expected Volume**
- Baseline: ~10,000 emails/day (confirmed). Design for bursts above baseline and provider rate limits.

**Operational patterns & requirements**
- **Idempotency:** Workers must deduplicate on `send_id` to avoid duplicate sends during retries or failover.
- **Retries & DLQ:** Exponential backoff on transient SendGrid errors; message move to DLQ after retry limit.
- **Suppression:** Sync SendGrid suppression/bounce/complaint events to internal suppression list and never re-send to hardened addresses.
- **Signing & validation:** Implement webhook signature/timestamp verification and replay protection.
- **Telemetry:** Metrics for sent/delivered/bounced/complaint rates, queue depth, worker errors, and webhook failures. Correlate via `trace_id`.

**Immediate next steps** (see `open-questions.md` for owners/due dates)
1. Confirm expected daily/peak volumes and PHI scope with Product & Compliance.
2. If PHI is in scope, request Legal to start BAA negotiations with Twilio/SendGrid.
3. Choose worker platform (Functions, App Service, Container Apps, AKS) and create a small spike:
   - Service Bus queue -> worker spike -> SendGrid send -> persist provider id.
4. Provision test infra (Service Bus namespace, Key Vault, storage for audit logs) using IaC.
5. Configure a sending subdomain (`email.example.com`) and add DKIM/SPF/DMARC per SendGrid verification.
6. Implement webhook endpoint with signature verification and persistence to audit store.

**How to pick up (quick guide)**
- Read the ADR and `overview.md` first to understand contractual and compliance constraints.
- Check `open-questions.md` and resolve any assigned items before sending PHI.
- For the spike, you can run a simple worker locally that reads a Service Bus queue (use the Azure Service Bus SDK) and calls SendGrid's `mail/send` API with an API key from Key Vault (or environment variable for local testing).

**Contacts / Owners**
- **Primary owners:** Email Platform Team (see repo owners/TEAM document for contacts)
- **Legal/Compliance:** responsible for BAA and retention policy approvals
- **DevOps/SRE:** responsible for DNS changes, Key Vault, Service Bus provisioning, and network egress rules

**Acceptance criteria for initial pilot**
- Prototype successfully sends transactional emails in a test environment and processes SendGrid webhook events end-to-end.
- Audit store contains immutable send events; suppression sync prevents re-sends to bounced/complained addresses.
- Runbooks exist for bounce/complaint spikes and provider failover.

---

If you'd like, I can now scaffold the prototype worker (choose language: Node/Python/C#), or generate an IaC skeleton to provision Service Bus + Key Vault + worker identity.