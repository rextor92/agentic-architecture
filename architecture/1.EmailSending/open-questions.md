# Open Questions & Next Steps

**Date:** 2026-01-05
**Owners:** Email Platform Team

## Open Questions
- Expected daily and peak email volumes (baseline and burst patterns): **~10,000 emails/day** (confirmed).
- Will transactional emails ever contain PHI in body or attachments, or only references/links to protected data? **Current scope: no PHI in emails.**
- Does Legal/Compliance require us to exclude PHI until a signed BAA is in place with SendGrid/Twilio? **Legal confirmed (2026-01-05) that a BAA is required for PHI; current sends contain no PHI so a BAA is not required at this time.**
- Which Azure worker platform is preferred for the first implementation (Azure Functions, App Service/WebJobs, Container Apps, AKS)?
- Do we require dedicated sending IP(s) at launch, or can we start on shared IPs and request dedicated IPs later?
- Who will own the sending domain/subdomain DNS changes and DKIM/SPF/DMARC setup (DevOps/Platform/Security)?
- Template ownership and editing workflow: will non-developers update templates directly in SendGrid, or do we require repo-managed templates with CI validation?
- Retention and deletion policies for audit logs and suppression lists (how long to keep immutable audit events)?
- Required SLOs for send latency and delivery (e.g., queue processing time, delivery confirmation windows).
- Are there any pre-approved providers (legal or procurement preferences) or restrictions outside SendGrid?
- Do we need per-tenant dedicated IPs or per-customer sending isolation for deliverability or compliance?

## Next Steps (Concrete)
1. Confirm volume estimates and PHI scope with Product/Analytics and Compliance (owner: Product, due: 2026-01-12).
2. Only start BAA negotiation if PHI scope changes. Legal has confirmed BAAs are required for PHI (2026-01-05); trigger: PHI scope change (owner: Legal/Compliance).
3. Choose initial worker platform and create a small spike/prototype (owner: Email Platform Team, due: 2026-01-15):
   - Minimal prototype: Service Bus trigger -> template render -> SendGrid send -> persist message id.
   - Include local tests and a README.
4. Provision test environment infra (Service Bus namespace, Key Vault, storage for audit logs) via IaC (owner: DevOps, due: 2026-01-22).
5. Configure sending subdomain DNS records (SPF/DKIM) in a test DNS zone and verify with SendGrid (owner: DevOps/Platform, due: 2026-01-22).
6. Implement webhook endpoint for event processing, with signature verification and replay protection; wire to audit store and suppression sync (owner: Email Platform Team, due: 2026-01-29).
7. Define retention and access controls for audit logs and suppression list; ensure logs redaction rules are documented (owner: Security/Compliance, due: 2026-01-29).
8. Create runbooks for bounce/complaint spikes and provider outage failover (owner: Email Platform Team + SRE, due: 2026-02-05).
9. Pilot low-volume sends to verify deliverability and webhook processing; evaluate need for dedicated IP(s) (owner: Email Platform Team, due: 2026-02-12).
10. Document ADR (done), update `overview.md` and `considered-approaches.md` with lessons from prototype and pilot (owner: Email Platform Team, due: 2026-02-19).

## Acceptance Criteria for Next Steps
- Signed BAA (if PHI in scope) or documented decision to exclude PHI from provider payloads.
- Test infra and prototype successfully send and receive webhook events end-to-end in test environment.
- Audit store persists immutable events and suppression list sync works for bounce/complaint events.
- Runbooks and alerts created for the primary operational failure modes.

