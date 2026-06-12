# Domain 4 — Identity and Access Management (20%)

The highest-weighted domain on SCS-C03. Covers authentication and authorization across human, application, and system identities: roles and trust, policy evaluation, Organizations-level guardrails, cross-account access, federation, and attribute-based access control.

Lab numbering here is my own breakdown for build order and does not map to the official exam task statements (official 4.1 is authentication strategies; my 4.1 is role assumption mechanics).

## Subdomains

- [x] **4.1 — IAM roles, trust policies, STS AssumeRole** - Proves the trust policy is the gatekeeper by breaking same-account assume three ways.
- [ ] **4.2 — Policy evaluation logic** - identity vs resource policies, explicit deny, and permissions boundaries, tested against a live S3 bucket.
- [ ] **4.3 — SCPs, RCPs, and Organizations** - builds a real org and member account to prove the management-account exemption. Only lab in the domain that leaves infrastructure residue.
- [ ] **4.4 — Cross-account access and the confused deputy** - ExternalId, `aws:PrincipalOrgID`, and the both-sides requirement for cross-account assume.
- [ ] **4.5 — Federation and IAM Identity Center** - hands-on GitHub Actions OIDC, plus decision drills on the STS API family, Cognito user pools vs identity pools, and Identity Center vs direct SAML.
- [ ] **4.6 — ABAC** - principal tags and session tags via SSM Parameter Store, including the gotcha that role resource tags do not populate `aws:PrincipalTag`.
