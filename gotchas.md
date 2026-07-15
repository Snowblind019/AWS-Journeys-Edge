# gotchas.md

Running list of exam traps and lab surprises, by domain. Each line is the sharp edge that separates a plausible answer from the correct one.

---

## Domain 4 — Identity and Access Management

- The trust policy is the gatekeeper. Identity permissions never punch through it, so when an assume fails, read the trust policy first.
- `:root` in a trust policy delegates to IAM (you also need an identity policy). Naming a principal directly is sufficient on its own.
- Same-account is a union (an allow on either side is enough). Cross-account is an intersection (both sides must allow).
- Explicit deny beats any allow, anywhere, and is scoped to the actions and resources it names.
- A permissions boundary is a ceiling, never a grant. Its implicit deny does not limit a same-account resource grant to a directly-named user ARN, but an explicit deny in it always wins, and the named-ARN bypass does not apply to a plain role ARN.
- S3 object vs bucket actions: `GetObject` needs `bucket/*`, `ListBucket` needs `bucket` (no `/*`). The wrong ARN shape is a silent deny.
- SCP never grants, caps member-account principals, and never restricts the management account or its root.
- SCP is principal-side, RCP is resource-side. An external threat, including a bucket policy granting an external account, needs an RCP, not an SCP.
- Cross-account assume needs both the trust naming you and your identity policy granting `sts:AssumeRole`.
- `ExternalId` is the third-party/SaaS confused deputy. `aws:SourceArn`/`aws:SourceAccount` is the AWS-service confused deputy. Do not swap them.
- OIDC trust: `aud` proves the token came from GitHub, `sub` proves it came from your repo. `aud`-only with no `sub` lets anyone's repo assume the role. The thumbprint is theater (validated against the CA since 2023).
- Cognito: the user pool authenticates, the identity pool authorizes to AWS.
- Identity Center is console-enable only, and it is the answer for many-account workforce access, not per-account SAML.
- ABAC principal tags come from user tags or session tags, never a role's own resource tags. Tagging the role does nothing.
- ABAC collapses if the authorization tag is editable, so pair the allow with a deny on `aws:TagKeys`.
- Tags are case-sensitive in conditions, and not every service supports `aws:ResourceTag` (S3 objects use `s3:ExistingObjectTag`).

---

## Domain 5 — Data Protection

- `:root` in a KMS key policy means the account delegating to IAM, not the root user, and not a standalone grant.
- A same-account KMS allow needs the key policy to permit it (unlike S3's plain union). An IAM allow alone is insufficient unless the key policy enables IAM or names the principal directly.
- Explicit deny in IAM, an SCP, or a boundary beats any key-policy allow (the Deny asymmetry).
- A KMS key has one policy named `default`. Lock yourself out and Support cannot fix it, so always keep a controllable admin statement.
- Cross-account KMS needs both the key policy allowing the external account and the caller's IAM. Use the key ARN, not the alias.
- Only crypto plus `DescribeKey`/`GenerateDataKey` work cross-account. Management actions do not.
- Encryption context is authenticated (AAD), not secret (it is in CloudTrail). It must match exactly to decrypt, scopes decrypt by condition, and AWS services set it (S3 = object ARN, Secrets Manager = secret ARN).
- Direct `kms:Encrypt` caps at 4 KB. Bigger data means envelope encryption, and no flag raises the limit.
- Destroy the plaintext data key after use, store the wrapped one. `Decrypt` needs no `--key-id` for symmetric keys (embedded), required for asymmetric.
- New S3 objects are always encrypted (SSE-S3 minimum since 2023). "Uploaded unencrypted" is impossible for new objects.
- Enforce-KMS bucket policy: an absent `x-amz-server-side-encryption` header counts as "not aws:kms", because policy runs before S3 applies its default.
- Require TLS: deny `aws:SecureTransport` false on both the bucket and object ARNs.
- Cross-account secret read needs three: the secret resource policy, the caller's `GetSecretValue` IAM, and `kms:Decrypt` on a customer CMK. The `aws/secretsmanager` managed key cannot cross accounts.
- Rotating a CMK does not re-encrypt data or protect a leaked data key. Only new data keys use new material, so exposed data must be re-encrypted.
- Only symmetric AWS-origin keys auto-rotate. Imported, asymmetric, HMAC, and custom-store keys are manual.
- Imported (EXTERNAL) keys: keep a copy outside AWS (AWS does not retain it), and you can delete the material on demand (crypto-shred).
- Multi-Region keys share material and key ID but have independent policies, grants, and aliases. A primary cannot be deleted while replicas exist.
- CloudHSM key store is dedicated HSMs you control (still in AWS). XKS is material that never resides in AWS.
- Object Lock requires versioning and protects versions (a plain delete just adds a marker). Governance has an admin bypass, Compliance blocks everyone including root and cannot be shortened.
- MFA Delete is root-only, CLI-only, and requires versioning.
- ETag is not an integrity hash for multipart or SSE-KMS objects. Use the checksum fields.
- Macie is S3-only. Automated discovery samples (cheap, continuous), jobs scan exhaustively (audit), and a score of 0 is not "clean".
- ACM: the CloudFront cert must be in us-east-1, the ALB/NLB cert in the LB's Region. Imported and email-validated certs do not auto-renew, DNS validation does.

---

## Domain 3 — Infrastructure Security

- Security groups are stateful (return auto-allowed, allow-only, on the ENI). NACLs are stateless (each direction, allow and deny, numbered, on the subnet).
- A custom NACL denies everything until you add rules and needs an outbound ephemeral allow (1024 to 65535) for return traffic. The missing ephemeral rule is the classic self-inflicted outage.
- Stripping SG egress does not break the reply to an allowed inbound connection, but it does break instance-initiated outbound.
- A NACL is the only place for an explicit deny (block a specific bad IP).
- Gateway endpoint (S3 and DynamoDB, free, route-table-based, no on-prem or cross-Region) vs interface endpoint (an ENI, per-hour, SG on 443, on-prem reachable).
- Pin a bucket to a VPC with an endpoint policy (the doorway) plus a bucket policy on `aws:sourceVpce` (require the doorway). `aws:sourceVpce` is a hard lock that also blocks the console.
- A subnet is public only via a route to an IGW, not a toggle and not the public IP. NAT is outbound-only IPv4, egress-only IGW is outbound-only IPv6, and NAT lives in the public subnet.
- Peering is non-transitive, forbids edge-to-edge routing and overlapping CIDRs, and needs routes on both sides. Transit Gateway is the transitive hub, not open by default.
- Public IPv4 addresses bill hourly since Feb 2024.
- CloudFront private origin uses OAC, not OAI, granting the CloudFront service principal scoped by `AWS:SourceArn`. SSE-KMS objects also need the CloudFront principal on the KMS key policy, and an S3 website endpoint cannot use OAC or OAI.
- CloudFront WAF is scope `CLOUDFRONT` in us-east-1, set on the distribution. Roll rules out in Count mode first. WAF inspects 8 KB of body on CloudFront (64 KB on ALB and API Gateway).
- Shield Standard is free and automatic. Advanced ($3,000/month) adds L7 mitigation and the SRT, but the SRT also needs a Business or Enterprise support plan.
- IMDSv1 plus an SSRF bug leaks role credentials with an unauthenticated GET. IMDSv2 needs a PUT-obtained token and a low hop limit (1 for a host, 2 for containers).
- An account or Region IMDS default is not enforcement (overridable, does not touch existing instances). Enforce with a per-AMI setting, an org declarative policy, and `ec2:MetadataHttpTokens`/`ec2:RoleDelivery` conditions.
- Session Manager needs no inbound ports, and a private-subnet instance needs the `ssm`, `ssmmessages`, and `ec2messages` interface endpoints.
- DNS Firewall only sees Route 53 Resolver queries (an external resolver or raw IP bypasses it). Network Firewall is the AWS-managed engine, and GWLB inserts a third-party appliance via GENEVE. "Whose engine": AWS-managed is Network Firewall, a named vendor is GWLB.
- Direct Connect is private but not encrypted. MACsec is L2 link encryption (dedicated-only at 10/100/400G, link not journey), IPsec VPN over DX is L3 end-to-end (~1.25 Gbps per tunnel), and every Site-to-Site VPN has two tunnels.
- DNSSEC signing is authoritative-side, validation is resolver-side. The KSK is backed by an asymmetric ECC_NIST_P256 CMK in us-east-1, KMS key loss yields SERVFAIL, and you disable by removing the DS record first.
- ALB mTLS verify authenticates client certs at the load balancer. NLB has no LB-managed mTLS.

---

## Domain 1 — Detection

- CloudTrail's bucket policy needs an ACL-check and a write grant scoped to `AWSLogs/account`, with `aws:SourceArn` and, cross-account, `bucket-owner-full-control`. A missing or mis-scoped policy is why a trail fails to create or stops delivering.
- Global services (IAM, STS, CloudFront, Route 53) log in us-east-1, so missing IAM activity means a single-region trail. Use a multi-region trail.
- Data events and log file validation are opt-in and off by default.
- Detection is not prevention: validation catches tampering, Object Lock plus a separate logging account prevents it.
- GuardDuty is regional (enable everywhere) and consumes its inputs independently (no log config needed).
- A trusted IP list suppresses, a threat IP list alerts, and there is one trusted list per account. Suppression rules archive (still exported), they do not delete.
- Service grid: malicious behavior is GuardDuty, sensitive data is Macie, CVEs and reachability are Inspector, config drift is Config, post-finding investigation is Detective.
- Security Hub CSPM depends on Config recording, so `NOT_AVAILABLE` controls or a wrong score is a Config problem first.
- Disable a control (stop checking and paying) vs suppress via an automation rule (keep checking, mute, kept for audit).
- Automation rules change the finding (ASFF). EventBridge changes the resource.
- The Config recorder must be continuous and record all plus global types (feeds Security Hub and Firewall Manager). Remediation runs under a separate `AutomationAssumeRole`, so silent failure is that role.
- Config change-triggered vs periodic rules, an aggregator is visibility not enforcement, and org conformance packs enforce a baseline member accounts cannot weaken.
- CloudTrail to CloudWatch Logs is a separate path from S3, so "trail exists but no alarms fire" means it is not wired.
- Metric filter plus alarm notifies (counts forward only), Logs Insights investigates (historical), a subscription filter forwards events (SIEM).
- CloudWatch data protection masks PII at ingest (before storage), revealed only with `logs:Unmask`, account vs log-group scope, and it is mask-at-ingest vs Macie's scan-at-rest.
- Security Lake normalizes to OCSF. Query access (SQL) vs data access (SQS plus your own code), Athena (one-off, partition on time) vs OpenSearch (frequent, low-latency). OCSF is Security Lake, ASFF is Security Hub CSPM.
- Logging silent failures: API Gateway's account-level role, Lambda's execution role `logs:` perms, CloudFront latency and destination, Route 53's us-east-1 and resource policy, Flow Logs' delivery role or bucket policy. Flow Logs never capture Amazon DNS resolver, DHCP, metadata, or Windows activation.

---

## Domain 2 — Incident Response

- Terminating or stopping destroys volatile memory (and the EBS volume if `DeleteOnTermination`). Capture memory from the running instance first, and treat terminate-first as always wrong.
- An SG swap does not contain a live intrusion (stateful, established flows survive), and a new SG still allows egress. Correct containment is an empty SG plus revoked egress plus a stateless NACL deny, ideally in a dedicated quarantine subnet.
- Revoke live role credentials with `aws:TokenIssueTime` (the console "Revoke active sessions"). Deleting the role does not invalidate issued tokens.
- The break-glass role is pre-staged, and blast radius is set at architecture time.
- Memory capture needs SSM reachability, and no-egress isolation cuts SSM, so add the `ssm`, `ssmmessages`, and `ec2messages` endpoints.
- You cannot share an `aws/ebs`-encrypted snapshot cross-account. Re-copy under a CMK and grant `kms:Decrypt`, `DescribeKey`, `CreateGrant`, `ReEncrypt*`, then double-copy under a forensic-owned CMK.
- Mount evidence read-only (`-o ro`) and use COMPLIANCE Object Lock for retention.
- AWS's `AWSCompromisedKeyQuarantineV2` is damage-limiting (a subset deny), not remediation.
- Kill sessions by principal: `aws:TokenIssueTime` (role sessions), `aws:userid` (IAM user `GetSessionToken`, no console button), credential rotation (root). Deactivating an AKIA key does not kill live ASIA sessions.
- Containment is not eradication, so hunt persistence (extra keys, backdoor users, new roles, altered trust, a changed account email). The real fix is temporary credentials via roles.
- Detective: one finding is not one incident (Extended Threat Detection plus finding groups), Detective vs Athena (fast scoping vs ad-hoc query), and severity is priority not truth (validate against baseline).
- Automated remediation: fully automatic destructive action on unfiltered findings is a self-inflicted outage, so filter `Sample` false, scope by tag and account, and gate destructive steps behind a custom action. Cross-account via `AssumeRole`, never hardcoded creds. ASR is the named org-wide remediation solution.
- Validation is demonstrated, not documented. FIS stop conditions (CloudWatch alarms) are mandatory, Resilience Hub finds gaps, FIS runs experiments, ARC proves recovery.
- Root sits above IAM: no identity policy, boundary, or `TokenIssueTime` touches it. An SCP only reaches member-account root, never the management account. Root containment is credential rotation.
- A root-deny SCP breaks `sts:AssumeRoot`. Task-scoped root sessions use the five task policies, and the management account root cannot be centralized.
- Malware Protection for EC2 is agentless and not continuous (once per 24 hours). The latest backup may be infected, so recover from a verified-clean point (Malware Protection for AWS Backup), and rebuild from a golden AMI.
- S3 ransomware: encryption at rest is not a defense (SSE-C is the weapon). Immutability (Object Lock COMPLIANCE), versioning, and MFA Delete are the survival controls, and you revoke access before restoring.

---

## Domain 6 — Security Foundations and Governance

- Enabling any Organizations policy type attaches a FullAWSAccess default, so RCPs and SCPs are deny-lists over an implicit allow and never grant.
- An RCP is a resource-side upper bound, an SCP is principal-side. An org-perimeter RCP must except `aws:PrincipalIsAWSService` or it breaks service-linked-role access.
- A declarative policy is a durable config baseline (survives new APIs). An AI opt-out is content governance and does not block usage.
- A root-deny SCP breaks `sts:AssumeRoot`. Delegated admin is the reflex for every security service (not from the management account), and secure-by-default means new accounts ship with no root credentials.
- Centralized root sessions cover common tasks (unlock S3/SQS, manage root creds). The root-only list (account name, email, or password, restore IAM after lockout, Support plan, close account, RI Marketplace, GovCloud, MFA Delete on S3) needs an actual root sign-in and is what break-glass exists for.
- Control Tower has two independent axes: behavior (preventive SCP, detective Config, proactive CFN hook) and guidance (mandatory, strongly recommended, elective). Proactive controls only see CloudFormation, preventive controls skip the management account, and an OU allows at most five SCPs (nest OUs).
- cfn-lint checks validity (will it deploy), cfn-guard checks security (should it). A Guard-backed Hook is a proactive control (CloudFormation-only). StackSets self-managed (roles you create, account-ID targets) vs service-managed (CFN-managed roles, OU targets, auto-deployment). StackSets deploy resources, Account Factory creates accounts.
- RAM shares infrastructure (in-org sharing skips invitations, external accounts must accept). A resource-based policy grants one data resource (two-sided). Service Catalog is self-service where a launch constraint lets users deploy without underlying permissions. Firewall Manager is the orchestrator not the firewall, and Config is its silent prerequisite.
- Config vs Security Hub (Security Hub depends on Config). Audit Manager is your evidence (customer side), Artifact is AWS's reports (provider side). The Well-Architected Tool reviews design, not live resources.
- A tag policy governs shape (value and case, never presence). An SCP with `aws:RequestTag` and a `Null` condition governs presence. `aws:RequestTag` is the tag in the request, `aws:ResourceTag` is the tag on the resource. Deny-based require-tag SCPs break create-then-tag services (Secrets Manager). A tag is only a control when a policy reads it.