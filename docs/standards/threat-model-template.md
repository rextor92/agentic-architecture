# docs/standards/threat-model-template.md
# Threat Model Template (Healthcare-oriented)
# Recommended: one per feature or system boundary
# File name convention: architecture/<feature>/threat-model.md (or threat-model-YYYY-MM-DD.md)

# Threat Model: <System / Feature Name>

## Status & Ownership
**Status:** Draft | Reviewed | Approved  
**Date:** YYYY-MM-DD  
**Owners:** <name/team>  
**Reviewers:** <security, compliance, engineering>  
**Related Docs:**  
- `architecture/<feature>/overview.md`
- Relevant ADRs: <links>
- Data classification policy: <link if exists>

## 1. Executive Summary
Briefly describe:
- What we are protecting
- Major risks identified
- Top mitigations / design decisions

## 2. Scope
### In Scope
- Components:
- Environments (dev/test/prod):
- Identities (users/services/admins):
- Data stores and queues:
- Network boundaries:

### Out of Scope
-  
(Example: “client endpoint security,” “third-party SaaS internals,” etc.)

## 3. System Overview
### Architecture Description
- What the system does:
- Primary user journeys / workflows:
- Critical dependencies:

### Key Assumptions
-  
(Example: “All traffic enters via WAF,” “private endpoints required,” etc.)

## 4. Data Classification & Compliance
### Data Types
List data types processed/stored/transmitted:
- PHI:
- PII:
- Operational metadata:
- Logs/telemetry:
- Secrets/keys/certificates:

### Regulatory / Policy Requirements
- HIPAA/HITECH (if applicable):
- GDPR (if applicable):
- Local healthcare regulations (if applicable):
- Audit evidence expectations:
- Data residency/sovereignty requirements:

### Data Retention & Deletion
- Retention periods:
- Deletion requirements:
- Archival approach:
- Legal hold considerations (if applicable):

## 5. Assets, Actors, and Trust Boundaries
### Assets (What we protect)
- Patient data (PHI/PII):
- Credentials/secrets:
- Clinical workflows availability:
- Integrity of orders/results/records:
- Audit logs:

### Actors
- End users (clinical staff, patients, admins):
- Service identities (managed identities, workload identities):
- External partners (HL7/FHIR, labs, insurers):
- Attackers (external, insider, supply chain):

### Trust Boundaries
Describe boundaries where trust changes:
- Internet ↔ Ingress/WAF
- App ↔ Data stores
- Tenant ↔ Tenant (if multi-tenant)
- On-prem ↔ Cloud
- Partner networks ↔ Cloud

## 6. Entry Points & Attack Surface
List entry points and interfaces:
- Public endpoints (if any):
- Private endpoints:
- APIs (REST/GraphQL/gRPC):
- Messaging/eventing:
- Admin planes (Azure portal, kubectl, CI/CD):
- Identity providers (Entra ID, B2C, etc.):

## 7. Security Controls Baseline
Document current/required baseline controls:
- Identity: MFA, Conditional Access, PIM, RBAC, least privilege
- Secrets: Key Vault/HSM, rotation, no secrets in code
- Network: private endpoints, NSGs, firewall rules, egress control
- Compute hardening: container image scanning, runtime policies
- Data: encryption at rest/in transit, key management
- Monitoring: SIEM integration, alerting, anomaly detection
- Change control: IaC, approvals, policy enforcement (Azure Policy)
- Vulnerability management: SAST/DAST, dependency scanning

## 8. Threat Analysis (STRIDE)
For each category, list threats and mitigations. Add rows as needed.

### Threat Register
| ID | STRIDE | Threat Scenario | Impact (H/M/L) | Likelihood (H/M/L) | Risk | Affected Assets | Existing Controls | Mitigations / Actions | Owner | Status |
|---:|:------:|-----------------|:--------------:|:------------------:|:----:|-----------------|------------------|-----------------------|-------|--------|
| T01 | S | <spoofing scenario> | H | M | H | <assets> | <controls> | <mitigations> | <owner> | Open |
| T02 | T | | | | | | | | | |
| T03 | R | | | | | | | | | |
| T04 | I | | | | | | | | | |
| T05 | D | | | | | | | | | |
| T06 | E | | | | | | | | | |

### Notes for Healthcare
Explicitly consider:
- PHI leakage via logs/telemetry
- Misconfigured access leading to broad PHI exposure
- Partner integration trust failures (HL7/FHIR)
- Ransomware and availability attacks impacting patient care
- Insider access misuse
- Data integrity risks (tampering with results/orders)

## 9. Abuse Cases / Misuse Scenarios
List realistic “how it could be abused” stories:
- UC01:
- UC02:
- UC03:

## 10. Mitigation Plan & Prioritization
### Top Risks (Priority List)
- P1:
- P2:
- P3:

### Security Backlog (Actionable)
- [ ] Task, owner, due date (if known)
- [ ] Task, owner, due date (if known)

## 11. Validation & Testing
How mitigations will be validated:
- Pen test / red team:
- Threat-informed test cases:
- Config compliance checks (Azure Policy):
- Logging/audit verification:
- DR/BCP tests (availability requirements):

## 12. Residual Risk & Sign-off
### Residual Risk Summary
- Remaining high risks and why accepted:

### Approvals
- Security:
- Compliance/Privacy:
- Engineering:
- Product/Business owner:

## Appendix A: Diagram Links
- Context diagram:
- Data flow diagram:
- Trust boundary diagram:

## Appendix B: References
- Policies/standards:
- Relevant ADRs:
- External requirements:
