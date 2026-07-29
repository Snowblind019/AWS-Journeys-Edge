# Trigger Phrase Index

The stem's wording is the answer key. Scan for these before reading the options, because the phrase usually eliminates three of them.

---

## Scope and scale words

| Phrase | What it forces |
|---|---|
| "existing and future resources" | a remediation path, not a preventive-only control. Kills SCP-alone answers |
| "hundreds of accounts", "across the organization" | delegated administrator, organization-wide aggregator, or Firewall Manager. Kills per-account setups |
| "accounts and Regions we currently use" | this is the wrong scope; the correct option says organization-wide |
| "new accounts added later" | auto-enable, service-managed StackSets, organization instance |
| "LEAST operational overhead" | the native managed service, not a Lambda pipeline you build |
| "MOST efficient", "least effort" | fewest moving parts that still meets every stated requirement |
| "within 24 hours" | rules out patching source code and any redesign |

## Time and freshness words

| Phrase | What it forces |
|---|---|
| "near real time" | CloudTrail data events, EventBridge, streaming to CloudWatch Logs. Kills S3 server access logs and scheduled jobs |
| "earliest detection" | a push-based ML or event service, never a console someone reviews |
| "continuously replicate" | Elastic Disaster Recovery. Kills AMIs, snapshots, AWS Backup |
| "RPO of N" | compare N against every option's backup cadence first |
| "must work if only one Region is available" | each Region needs a local, independent copy. Kills call-back-to-the-other-Region designs |
| "cumulative impact of several rapid changes" | Config configuration items, not per-call CloudTrail |

## Authority and immutability words

| Phrase | What it forces |
|---|---|
| "even administrators cannot delete" | compliance mode (S3 Object Lock or Backup Vault Lock). Governance mode always has a bypass |
| "immutable evidence", "tamper-proof" | S3 Object Lock, plus CloudTrail log file validation |
| "separation of duties" | customer managed SSE-KMS, ops owns the bucket policy, security owns the key policy |
| "configuration errors by one team must not expose data" | two independent policy layers, so a customer managed key |
| "only temporary credentials" | roles, STS, Identity Center, Roles Anywhere. Kills embedded access keys |
| "no manual effort", "automatically" | kills alarms-to-email, human review, and DRT engagement |

## Identity words

| Phrase | What it forces |
|---|---|
| "external users" | Cognito user pool |
| "minimize user database management" | Cognito, and the DynamoDB credential table is the trap |
| "SAML-based IdP" plus "multiple AWS accounts" | IAM Identity Center permission sets with a session duration |
| "certificate-based authentication" from on-premises | IAM Roles Anywhere |
| "custom validation at sign-up, accept or deny" | Cognito pre sign-up Lambda trigger |
| "temporary access to the key, occasionally" | KMS grant |
| "third party" plus a role | `ExternalId` in the trust policy |
| "which principals in our account can access this resource" | Access Analyzer internal access analyzer |

## Network words

| Phrase | What it forces |
|---|---|
| "cannot send traffic over the public internet" plus S3 or DynamoDB | gateway VPC endpoint |
| "reachable from on-premises" | interface endpoint, not gateway |
| "instances are regularly created and terminated" plus "only some need access" | security group referencing |
| "sever the active session" | stateless NACL deny, because a security group change will not do it |
| "SQL injection" plus EC2 behind Route 53 | insert an ALB, then attach the WAF web ACL |
| "block the country" but the app is global | wrong; use a rate-based rule instead |
| "layer 7 DDoS" plus CloudFront plus automated | Shield Advanced automatic application layer DDoS mitigation |

## Troubleshooting stems

| Symptom | Read this first |
|---|---|
| Works in dev and test, fails in prod, OUs have different SCPs | CloudTrail in the failing account for the denied action and its source |
| Cross-account assume fails | trust policy and `ExternalId`, caller's `sts:AssumeRole`, exact role ARN |
| Access denied on an encrypted object though IAM looks correct | the KMS key policy, which may no longer delegate to IAM |
| GuardDuty control test produced no DNS finding | the VPC DHCP option set pointing at a custom resolver |
| Permission set assignment fails in some accounts | a customer managed policy missing from those accounts |
| MFA policy blocks all CLI calls | users need `get-session-token` with serial number and token code |
| EC2 Instance Connect host key validation error after rotation | the new host key was never published |
| Presigned URL expires early | it was signed with instance profile credentials that rotated |
| Logs disappear on scale-in | nothing streams them off the instance |

## Fabricated-option tells

If an option says any of these, it is wrong by construction:

- "Apply an AWS WAF web ACL to the EC2 instances"
- "Configure a geographic restriction setting in the Cognito user pool"
- "Create a rule in GuardDuty to block the access key"
- "Configure Amazon Inspector to scan the S3 buckets for sensitive data"
- "Publish findings to AWS Trusted Advisor"
- "Create an identity pool in IAM Identity Center"
- "Store the customer-provided SSE-C keys in AWS KMS"
- "Share the HSM ID with AWS RAM"
- "Configure an OpenSearch table with a partition projection"
- "Create a CloudWatch alarm with a `StopLogging` event name"
- "Export the KMS key material to an on-premises HSM"
- "Configure GuardDuty to directly invoke the Lambda function"