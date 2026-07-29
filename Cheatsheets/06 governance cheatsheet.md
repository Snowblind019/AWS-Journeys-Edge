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
---

## What an SCP can and cannot see

An SCP is an IAM policy, so it can only evaluate what appears in the request context as a condition key. It cannot reach inside nested request parameters.

- **Can**: `aws:RequestTag/x` with a `Null` check (tag presence), `s3:x-amz-server-side-encryption`, Region, service, action, principal ARN, resource ARN patterns, instance types via `ec2:InstanceType`.
- **Cannot**: the CIDR and port pair nested inside `IpPermissions` on `AuthorizeSecurityGroupIngress`. There is no condition key for "this specific rule allows 0.0.0.0/0 on 22", which is why "prevent open SSH security group rules" is a **detect and remediate** question (Config or Security Hub finding, EventBridge, Lambda) and not an SCP question, despite the word "prevent" in the stem.
- **Cannot**: an arbitrary Config rule outcome, drift, or configuration state. If compliance is defined by what Config evaluates, an SCP cannot express it.
- **Cannot**: act on anything that already exists. "Existing and future resources" always needs a remediation path alongside any preventive control.

Rule of thumb: "prevent" plus a simple condition key is an SCP. "Prevent" plus nested rule detail, or "existing and future", or "remediate", is Config or Security Hub with automated remediation.

## Config solution shapes (these five map one-to-one)

- Collection of rules deployable org-wide → **conformance packs**.
- Compliance data from many accounts and Regions into one account → **aggregator** (organization-wide scope, not "only the accounts and Regions we currently use", which silently stops covering anything added later).
- Evaluate resource configuration against desired settings → **Config rules**.
- Automatically fix noncompliant resources → **Config with Systems Manager** (remediation runs an SSM Automation document).
- Notify on configuration change or compliance violation → **Config with AWS User Notifications**.

## Compliance and audit services, five questions

- **Config**: is a resource compliant, and can you fix it?
- **Security Hub**: prioritized findings and standard scores, on top of Config.
- **Audit Manager**: automatically collects evidence from CloudTrail, Config, and Security Hub, mapped to a framework, for an assessment report. Customer-side evidence.
- **Artifact**: download AWS's own SOC, PCI, and ISO reports on demand. Provider-side documents, nothing to configure.
- **Well-Architected Tool**: review the architecture against the pillars, document risks, implement mitigations. Reviews design, not live resources, and is the answer when the requirement is to **improve and evidence resilience** (Reliability pillar) rather than scan for vulnerabilities or gather compliance evidence.

Also watch the environment scope: FIS is the right category for testing resilience, but running experiments in the development account fails a requirement scoped to production workloads.

## Firewall Manager vs the alternatives

- Org-wide WAF, Shield, or Network Firewall policy that auto-applies to existing **and future** resources across accounts → **Firewall Manager**, one policy, self-enforcing, least operational overhead. Config-plus-remediation, Service Catalog products, and custom Security Hub to EventBridge to Lambda pipelines are all detect-then-fix loops with more moving parts and a window where a resource exists unprotected.
- Config is Firewall Manager's silent prerequisite.

## Troubleshooting order when an SCP is suspected

A template that deploys in Development and Testing but fails in Production with an IAM permissions error, where the OUs have different SCPs, is an SCP question. The **first** step is still evidence: read CloudTrail in the failing account for the exact denied action and denial source, which distinguishes an SCP explicit deny from a plain IAM gap. Removing SCPs to test the theory, or copying another OU's SCPs over, are both live security-lowering changes made before the cause is known.

## Question triggers

- "Centrally give users access to an AWS managed application (Q Developer, Q Business, QuickSight, SageMaker Studio) across the organization" → IAM Identity Center organization instance. "Identity pool" in an option is a Cognito distractor.
- "Tag must exist **and** the value must be one of N approved values" → tag policy in enforcement mode for the values, plus an SCP with a `Null` condition on `aws:RequestTag/Key` for presence. Tag policies never require presence.
- "Immutable evidence of a KMS key's creation, origin, and use, for auditors" → CloudTrail with log file validation delivering to an S3 bucket with Object Lock, trail configured before the key is created.