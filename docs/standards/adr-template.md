# docs/standards/adr-template.md
# Architecture Decision Record (ADR) Template
# File name convention: architecture/<feature>/adr/NNNN-short-title.md  (e.g., 0003-api-gateway-choice.md)
# Status values: Proposed | Accepted | Deprecated | Superseded

# ADR NNNN: <Decision Title>

## Status
**Status:** Proposed  
**Date:** YYYY-MM-DD  
**Owners:** <name/team>  
**Feature/Area:** <feature name or domain>  
**Related Docs:**  
- `architecture/<feature>/overview.md`  
- `architecture/<feature>/considered-approaches.md`  
- Links to related ADRs: <ADR links>

## Context
Describe the background and why this decision is needed.
- Business goals / user needs:
- Scope (in-scope / out-of-scope):
- Constraints (regulatory, security, budget, timeline, skills):
- Current state (what exists today):
- Assumptions (explicitly list anything not confirmed):
- Stakeholders consulted (and dates if known):

## Decision
State the decision clearly and unambiguously.
- We will:
- We will not:
- Key parameters (regions, sizing class, HA/DR stance, tenancy, etc.):

## Rationale
Explain *why* this option was chosen.
- Primary drivers:
- Supporting evidence / references:
- Why now (timing considerations):

## Options Considered
List the viable alternatives that were seriously considered.

### Option A: <Name>
**Summary:**  
**Pros:**  
-  
**Cons / Risks:**  
-  
**Operational impact:**  
-  
**Security/compliance impact:**  
-  
**Cost drivers:**  
-  
**Notes / unknowns:**  

### Option B: <Name>
**Summary:**  
**Pros:**  
-  
**Cons / Risks:**  
-  
**Operational impact:**  
-  
**Security/compliance impact:**  
-  
**Cost drivers:**  
-  
**Notes / unknowns:**  

### Option C: <Name> (optional)
...

## Consequences
What changes because of this decision (good and bad)?
- Positive outcomes:
- Trade-offs accepted:
- New risks introduced:
- Migration / adoption steps required:
- Impacted systems / teams:

## Security & Compliance Considerations (Healthcare)
(Keep this even if “N/A” and explain why.)
- Data classification involved (PHI/PII/Confidential/Internal/Public):
- Data residency/sovereignty constraints:
- Encryption (at rest/in transit/in use) expectations:
- Identity & access (least privilege, managed identity, RBAC):
- Audit/logging requirements (and redaction rules for PHI):
- Retention & deletion considerations:
- Break-glass / emergency access:
- Shared responsibility notes:

## Observability & Operations
- Monitoring/alerts/SLOs:
- Logging (what we log; what we must NOT log):
- Runbooks / on-call expectations:
- Backup/restore approach:
- RTO/RPO assumptions:
- DR approach:

## Cost & FinOps Notes
- Major cost drivers:
- Expected scaling characteristics:
- Cost controls (budgets, alerts, reservations/savings plans if applicable):

## Open Questions
List unresolved items that block implementation or require follow-up.
- [ ] Question 1
- [ ] Question 2

## Decision Validation
How will we know this decision is working?
- Success metrics:
- Load/perf tests:
- Security testing:
- Audit evidence expectations:
- Review date / revisit triggers:

## Supersession (if applicable)
If this ADR replaces another:
- Supersedes: ADR NNNN
- Superseded by: ADR NNNN (if later)
- Reason for supersession:
