# Considered Approaches: EmailSending

This document records the approaches considered for implementing transactional email delivery and the rationale for the recommended pattern (SendGrid + queued worker).

## Goals
- Deliver reliable, secure, and auditable transactional email (registration, password resets, receipts, alerts).
- Minimize operational burden while maintaining deliverability and compliance controls (PHI/PII handling).
- Support scale and provider failover with clear operational runbooks.

## Approach A — Provider-managed Transactional (SendGrid)

Summary
- Use SendGrid as primary provider for API-based transactional sends, provider templates (optional), event webhooks, and suppression management.

Pros
- Offloads deliverability, DKIM/SPF, reputation management.
- Rich event webhooks for bounces/complaints/deliveries; suppression lists and monitoring tools.
- Provider-side templates enable non-dev copy changes and versioning.

Cons / Risks
- Provider contract and BAA required before sending PHI; public API endpoints (no Azure Private Link).
- Some lock-in to provider template formats if heavily used.

Operational notes
- Keep internal canonical suppression list & audit store; sync SendGrid events into internal stores.
- Store API keys in `Key Vault`; workers use Managed Identity.
- Implement webhook signature verification and replay protection.

When to choose
- Preferred when deliverability and low operational overhead are priorities and a BAA can be obtained for PHI.

## Approach B — Cloud SMTP / SES (Self-managed via Cloud Provider)

Summary
- Use a cloud provider SMTP API (e.g., Amazon SES) or SMTP relay. Optionally use VPC endpoints (SES) to reduce egress exposure.

Pros
- Cost-effective at high volume; SES can be regionally constrained and supports BAAs via AWS agreements.
- VPC endpoints reduce public egress surface.

Cons / Risks
- Adds multi-cloud complexity if primary infra remains in Azure.
- SES management still requires handling DKIM/SPF, IP warming and reputation concerns.

Operational notes
- Provide adapter layer to abstract SES vs SendGrid APIs.
- Maintain internal suppression & audit stores, similar to Approach A.

When to choose
- Preferred if cost at scale is the primary driver and multi-cloud operating model is acceptable.

## Approach C — Self-Hosted MTA (Postfix/Haraka/Postal)

Summary
- Run and operate your own mail transfer agent or transactional mail platform in your environment (AKS/VMs).

Pros
- Full control over message content, IPs, and reputation handling.
- No third-party contract exposures for PHI (but still consider network egress if sending externally).

Cons / Risks
- High operational burden: deliverability, blocklist monitoring, TLS, DKIM, SPF, IP warm-up, monitoring, and maintenance.
- Poorer deliverability compared to established providers without significant investment.

When to choose
- Only when regulatory or policy prohibits third-party providers and the organisation has staffing to run a robust MTA and deliverability team.

## Approach D — Hybrid: Provider + Repo Templates + Adapter

Summary
- Render templates in-app (repo-managed) and use providers only for sending. Keep templates and localization in source control; provider templates used only for marketing/non-sensitive messages.

Pros
- Full control over templates, CI-driven changes, easier localization and testing.
- Lower risk of leaking PHI into provider-managed templates.

Cons
- Non-dev editors lose direct control over copy updates unless a CMS/workflow is provided.

When to choose
- Preferred when PHI is possible and template changes are primarily developer-driven or when strict CI and auditability for templates is required.

## Template Strategy Considerations
- Provider-managed templates (SendGrid): good for marketing and simple transactional copy; faster for content teams.
- Repo-managed templates: recommended for any template that may include sensitive content or requires strict CI/testing/localization. Use Handlebars or Liquid for templating.
- Hybrid: allow marketing templates in SendGrid and keep sensitive or complex templates in repo.

## Webhook & Event Handling Options
- Push model: provider sends webhook events to a validated HTTPS endpoint; endpoint verifies signatures and persists raw events to an immutable audit store.
- Pull model: periodically fetch event logs from provider (less real-time, less reliable for immediate suppression updates).

Recommendation: prefer push with strict signature/timestamp verification and idempotent processing.

## Security & Compliance Patterns
- Obtain signed BAA from provider before sending PHI. If BAA is not acceptable, use hybrid approach or exclude PHI from emails.
- Payload minimization: prefer links/tokens to PHI; surfacing PHI only behind authenticated portals.
- Secrets: store provider API keys in `Azure Key Vault` and grant minimal access via Managed Identities.
- Network: use Private Endpoints for Key Vault, Service Bus, and Storage; restrict outbound egress to known provider endpoints via NAT/firewall rules.
- Logging: redact personal data in logs; save hashed identifiers for traceability in audit store.

## Scalability & Reliability Patterns
- Use Azure Service Bus for durable queuing, ordering, and DLQ support.
- Workers must implement idempotency with a `send_id` and handle retries with exponential backoff.
- Consider provider failover (SendGrid primary, SES secondary) implemented in the adapter layer with deduplication checks.

## Cost Considerations
- Provider costs: per-message pricing, dedicated IP costs, deliverability/consulting services.
- Azure costs: Service Bus, worker compute, Key Vault, storage for audit logs.
- Optimize telemetry sampling and retention to control audit storage costs.

## Migration & Adoption Steps (high-level)
1. Validate requirements: daily/peak volumes, PHI presence, legal/BAA constraints.
2. Select primary provider (SendGrid) and negotiate necessary agreements (BAA if required).
3. Provision infra: Service Bus, Key Vault, worker platform, audit storage.
4. Implement provider adapter, templating strategy, webhook processor, and suppression sync.
5. Pilot with low-volume sends; monitor deliverability and adjust DKIM/SPF/DMARC, request dedicated IPs if needed.

## Recommendation
- Adopt Approach A (SendGrid) as primary for transactional email, with Approach D (Hybrid templates) for any content that may include sensitive information.
- Maintain a clear fallback plan to SES and keep an adapter layer to minimize lock-in.

