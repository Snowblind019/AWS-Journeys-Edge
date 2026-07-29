# Cheatsheet — Confusion Pairs

Every pair here has appeared as two options in the same question. The middle column is the one sentence that separates them. Read the discriminator, not the service name.

---

## Detection and investigation

| A vs B | Discriminator | A when | B when |
|---|---|---|---|
| GuardDuty vs Config | behavior vs configuration state | malicious or suspicious activity | drift, misconfiguration, compliance |
| GuardDuty vs Inspector | threat detection vs vulnerability scanning | compromised credentials, crypto mining, exfiltration | CVEs, patch level, network reachability |
| GuardDuty vs Detective | detect vs investigate | "identify suspicious activity" | "root cause", "indicators of compromise", "investigate this principal" |
| Macie vs Inspector | data content vs software vulnerabilities | PII inventory in S3 | CVEs on EC2, ECR, Lambda |
| Security Hub vs Detective | aggregate and correlate findings vs deep-dive one entity | multi-stage attack across services | one role or instance, full activity graph |
| CloudTrail vs Config | every API call vs latest resource state | who called what, when | cumulative result of rapid changes |
| CloudTrail Insights vs metric filter | learned baseline vs fixed threshold | "unusually high volume" | "more than 50 in 1 hour" |
| CloudTrail data events vs S3 server access logging | structured, near real time vs flat text, hours late | identity plus timestamp per object call | legacy or cost-driven bulk request logs |
| Athena vs Logs Insights vs CloudTrail Lake | S3 vs CloudWatch Logs vs event data store | trail delivered to a bucket | trail delivered to a log group | an event data store exists |
| Cost Anomaly Detection vs Cost Explorer | push alert on ML baseline vs pull analysis | "earliest detection", "automatically" | "analyze", "trends", "forecast", "why did it rise" |

## Access and identity

| A vs B | Discriminator | A when | B when |
|---|---|---|---|
| Cognito vs IAM Identity Center | external app users vs internal workforce | customer-facing sign-up and sign-in | employees reaching AWS accounts |
| Cognito user pool vs identity pool | authenticate vs authorize to AWS | sign-in, tokens, password policy | temporary AWS credentials for app users |
| IAM Identity Center vs IAM Roles Anywhere | humans via SAML vs machines via X.509 | workforce SSO, session duration | on-premises servers, certificate auth |
| STS AssumeRole vs KMS grant | whole role for a session vs specific key operations until revoked | broad temporary access | one key, one purpose, revoke later |
| `GetSessionToken` vs `AssumeRole` | MFA-flag an existing user's own session vs take on a different identity | MFA-conditioned policy blocking CLI | cross-account or elevated role |
| SCP vs RCP | principal-side cap vs resource-side cap | "our identities must not reach untrusted resources" | "untrusted identities must not reach our resources" |
| Access Analyzer external vs internal | outside the zone of trust vs inside the account or org | accidental public or cross-account exposure | which of our principals can reach this resource |
| Permission set vs IAM role | Identity Center provisioning construct vs the account-level artifact | assigning workforce access | trust policy, cross-account assume |

## Protection and network

| A vs B | Discriminator | A when | B when |
|---|---|---|---|
| Security group vs NACL | stateful, instance, allow-only vs stateless, subnet, allow and deny | normal access control | explicit deny, sever an established session |
| Gateway vs interface endpoint | route table, free, S3 and DynamoDB vs ENI, hourly, everything else, on-prem reachable | private S3 access from a VPC | private access to KMS, SSM, Secrets Manager, or from on-premises |
| VPC endpoint vs peering or TGW | AWS public service vs another VPC's resources | S3, DynamoDB, KMS | EC2, RDS in another VPC |
| Rate-based rule vs geo match | behavior vs origin country | legitimate users exist in the same countries | no legitimate users there |
| Shield Advanced vs Shield Advanced with automatic L7 mitigation | visibility, cost protection, SRT vs automated rule creation during an attack | "detect and engage" | "automated, no manual effort" |
| SSE-S3 vs SSE-KMS customer managed | no key policy vs a key policy you own | simplicity | separation of duties, revocation, audit |
| Governance vs compliance mode | has a bypass vs no bypass, not even root | "most users cannot delete" | "administrators cannot delete" |
| Object Lock vs versioning | WORM on a version vs keep old versions | permanent-delete protection | accidental overwrite recovery |
| Multi-Region key vs two AWS managed keys | shared material and key ID vs unrelated keys | replicated encrypted data | Region-local encryption only |

## Governance and compliance

| A vs B | Discriminator | A when | B when |
|---|---|---|---|
| Audit Manager vs Artifact | your evidence vs AWS's reports | assessment report from CloudTrail, Config, Security Hub | download SOC, PCI, ISO on demand |
| Audit Manager vs Well-Architected Tool | evidence collection vs architecture review | prove controls to an auditor | improve resilience, document and mitigate risks |
| Conformance pack vs aggregator | deploy rules vs view results | one baseline across the org | unified compliance visibility |
| Tag policy vs SCP with `Null` | value shape vs tag presence | "must be one of three approved values" | "must have the tag" |
| Firewall Manager vs Config remediation | native org-wide enforcement vs detect-then-fix | least operational overhead, future resources auto-covered | anything Firewall Manager does not cover |
| Preventive vs detective vs proactive control | SCP before the call, Config after the fact, CloudFormation hook before provisioning | simple condition key | drift and existing resources | IaC-only guardrails |