# Cheatsheet — Domain 6: Security Foundations and Governance

One-page mental models per topic. The recurring spine: direction of control decides SCP vs RCP, tools govern either shape or presence, and a tag is only a control when a policy reads it.

---

**Organizations policy types:** deny-lists over an implicit allow, each a distinct job.
- SCP caps your principals (principal-side). RCP caps access to your resources regardless of principal (resource-side).
- An external threat, including a bucket policy granting an external account, is an RCP. Org-perimeter RCPs must except `aws:PrincipalIsAWSService` or they break service-linked roles.
- Declarative policy is a durable config baseline that survives new APIs. AI opt-out is content governance and does not block usage.
- Enabling any type attaches a FullAWSAccess default, so none of them grant.

**Centralized root and break-glass:** remove standing root credentials, run privileged work through scoped sessions.
- A root-deny SCP breaks `sts:AssumeRoot`. Delegated admin is the reflex for every security service, never from the management account.
- Task-scoped root sessions cover the common tasks (unlock S3/SQS, manage root creds).
- The root-only list (account name, email, password, restore IAM after lockout, Support plan, close account, RI Marketplace, GovCloud, MFA Delete on S3) needs an actual root sign-in, which is what break-glass exists for.

**Control Tower:** two independent axes.
- Behavior: preventive (SCP, before), detective (Config, after), proactive (CloudFormation hook, before provisioning).
- Guidance: mandatory, strongly recommended, elective. Guidance never implies behavior.
- Proactive controls only see CloudFormation. Preventive controls skip the management account. An OU allows at most five SCPs, so nest OUs.

**IaC security and rollout:** gates in order, then deploy the validated baseline.
- cfn-lint checks validity (will it deploy), cfn-guard checks security (should it). A Guard-backed Hook is a proactive control (CloudFormation-only).
- StackSets self-managed (roles you create, account-ID targets) vs service-managed (CFN-managed roles, OU targets, auto-deployment to new accounts).
- StackSets deploy resources into accounts, Account Factory creates accounts.

**Cross-account sharing:** decide by what you share and to whom.
- RAM shares infrastructure (subnets, transit gateways). In-org sharing skips invitations, external accounts must accept.
- A resource-based policy grants one data resource to a specific principal (two-sided cross-account, KMS strictest).
- Service Catalog is governed self-service, where a launch constraint lets users deploy approved products without the underlying permissions.
- Firewall Manager is the org-wide orchestrator (auto-applies WAF, Shield, Network Firewall to new resources), not the firewall. Config is its silent prerequisite.

**Compliance services:** five services, five questions.
- Config: is a resource compliant, and can you fix it?
- Security Hub: prioritized findings and standard scores, and it depends on Config.
- Audit Manager: your evidence against a framework (customer side).
- Artifact: AWS's own SOC/PCI/ISO reports (provider side), free.
- Well-Architected Tool: is the architecture sound? It reviews design, not live resources.

**Tagging strategy:** tags are the substrate, and shape is not presence.
- A tag policy governs shape (value and case, never presence). An SCP with `aws:RequestTag` and a `Null` condition governs presence.
- `aws:RequestTag` is the tag in the request, `aws:ResourceTag` is the tag on the resource, `aws:PrincipalTag` is on the caller (ABAC), `aws:TagKeys` restricts which keys.
- Deny-based require-tag SCPs break create-then-tag services (Secrets Manager). A tag is only a control when a policy reads it.