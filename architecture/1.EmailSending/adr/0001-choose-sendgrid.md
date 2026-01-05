---
title: "0001 Choose SendGrid for Transactional Email"
status: Accepted
date: 2026-01-05
owners: Email Platform Team
feature: EmailSending
related-docs:
  - `architecture/1.EmailSending/overview.md`
---

## Status
**Status:** Accepted  
**Date:** 2026-01-05  
**Owners:** Email Platform Team  
**Feature/Area:** EmailSending  
**Related Docs:**
- `architecture/1.EmailSending/overview.md`

## Context
We require a reliable, auditable, and secure transactional email service for user-triggered messages (registration confirmations, password resets, receipts, critical alerts). The environment is primarily Azure-hosted and some messages may contain PII/PHI. We must ensure deliverability, bounce handling, and observability while minimizing operational burden.

Constraints and assumptions:
- Emails may include PII/PHI; sending PHI requires a signed BAA with the provider.  
- Primary platform is Azure; prefer Azure-friendly integration patterns.  
- Volume is currently unknown; solution must scale from small to large and support provider failover.  

## Decision
We will use Twilio SendGrid as the primary transactional email provider.  

We will not:
- Send PHI through SendGrid until a Business Associate Agreement (BAA) is reviewed and signed by legal.  

Key parameters:
- Use a dedicated sending subdomain (e.g., `email.example.com`).  
- Store SendGrid API keys in Azure Key Vault and use Managed Identity for worker access.  
- Use Azure Service Bus as the canonical queue for send requests and DLQ processing.  
- Maintain an internal suppression list and audit store; synchronize SendGrid events to these internal stores.  

## Rationale
Primary drivers:
- Offloads deliverability, reputation, DKIM/SPF management, and provides mature webhook/event handling.  
- Offers templating and management features that can reduce developer overhead for copy changes.  
- SDKs and community support for integration with Azure-hosted workers.

Supporting evidence:
- SendGrid supports transactional use cases, dedicated IPs, and event webhooks.  
- Commonly used in the industry for similar workloads, reducing operational risk relative to self-hosted MTAs.

Why now:
- We need a production-ready transactional email capability with minimal operational overhead and strong deliverability.

## Options Considered

### Option A: Twilio SendGrid (Chosen)
**Summary:** Use SendGrid for API-based transactional email, webhook events, and optional provider-side templates.
**Pros:**
- Mature transactional tooling, templates, event webhooks, suppression management.  
- Dedicated IP options and deliverability consultancy available.  
- Lower operational burden than self-hosted solutions.  
**Cons / Risks:**
- Requires contractual review for PHI handling (BAA).  
- Public API endpoints (no private link) — requires controlled egress and payload minimization.  
**Operational impact:**
- Implement provider adapter, webhook processor, and synchronization to internal suppression list and audit store.  
**Security/compliance impact:**
- Must obtain BAA for PHI; redact sensitive data in logs; store secrets securely.  
**Cost drivers:**
- Per-email pricing, dedicated IP costs, potential deliverability services.  
**Notes / unknowns:**
- Confirm current BAA terms with Twilio/Sales.

### Option B: Amazon SES
**Summary:** Use Amazon SES (simple email service) as an alternate provider, optionally via VPC endpoints.
**Pros:**
- Cost-effective at scale; good deliverability with proper setup; VPC endpoints can limit egress exposure.  
**Cons / Risks:**
- AWS-centered provider; may add multi-cloud complexity if we stay primarily on Azure.  
**Notes:**
- Consider SES as a fallback provider if contractual or deliverability issues arise with SendGrid.

### Option C: Postmark / Sendinblue / Mailgun
**Summary:** Other transactional-focused providers offering high deliverability; evaluated as backups.
**Pros:**
- Some providers (Postmark) are highly focused on transactional deliverability and simpler operational models.  
**Cons / Risks:**
- Different feature sets and potential limits on templates or webhooks; must validate BAA availability.

## Consequences
Positive outcomes:
- Faster time-to-market for transactional email with good deliverability.  
- Reduced operational overhead for maintaining sending infrastructure.  

Trade-offs accepted:
- Dependence on a third-party provider for delivery and the need to negotiate BAA for PHI.  

New risks introduced:
- Potential exposure of sensitive data to an external provider (mitigated by BAA, payload minimization, and encryption).  

Migration / adoption steps required:
- Configure SendGrid domain authentication (SPF/DKIM/DMARC) on a dedicated subdomain.  
- Provision Azure Service Bus, workers (Functions/App Service/Container Apps), Key Vault secrets.  
- Implement provider adapter, webhook endpoint, and synchronization to internal stores.  

Impact:
- Teams: Email Platform, Security/Compliance, DevOps, Support.

## Security & Compliance Considerations (Healthcare)
- Data classification: likely PII/PHI for some emails; treat as sensitive until confirmed.  
- BAA: Required before sending PHI via SendGrid. Obtain legal approval and signed BAA.  
- Encryption: TLS in transit for all API calls and webhooks; storage encryption for audit logs.  
- Identity & Access: Use Azure Managed Identity for workers and grant least privilege to Key Vault and Service Bus.  
- Audit/logging: Persist immutable send/audit records in an access-controlled store; redact PII/PHI from central logs.  
- Retention: Apply retention and deletion policies to audit and suppression data per policy.  
- Break-glass: Document emergency procedures for disabling provider usage or revoking keys.

## Observability & Operations
- Monitor and alert on: send rates, bounce/complaint rates, queue depth, webhook failures, and provider latency.  
- Maintain runbooks for: high bounce/complaint spikes, webhook signature failures, provider outage failover.  
- SLOs: Establish delivery/retry targets and queue processing latency targets during implementation.

## Cost & FinOps Notes
- Major drivers: monthly send volume, dedicated IP cost, audit storage retention.  
- Controls: budget alerts, sampling logs to limit billable telemetry and audit cost, and periodic review of dedicated IP utility.

## Open Questions
- [ ] Confirm expected daily and peak email volumes.  
- [ ] Does our legal/compliance team require PHI to be excluded until BAA signed?  
- [ ] Which Azure worker platform is preferred for implementation?  
- [ ] Do we require dedicated IP(s) at launch for deliverability?

## Decision Validation
- Success metrics: <50% delivery rate for transactional email is unacceptable; target >95% delivery for non-bounced messages.  
- Validate: end-to-end test sends, webhook event delivery, suppression synchronization, and operational runbooks.  
- Review date / revisit trigger: Review in 6 months or on major deliverability incident.
