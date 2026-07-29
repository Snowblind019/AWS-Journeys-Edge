# Cheatsheet — Domain 2: Incident Response

One-page mental models per topic. The recurring spine: contain without destroying evidence, revoked credentials can outlive the role, and one finding is not one incident.

---

**EC2 containment:** contain in the right order or you destroy the evidence.
- Terminate and stop both destroy RAM (and the EBS volume if `DeleteOnTermination`), so capture memory from the running instance first.
- An SG swap does not contain a live intrusion (stateful flows survive) and still allows egress. Correct: empty SG plus revoked egress plus a stateless NACL deny, in a quarantine subnet.
- Revoke live role credentials with `aws:TokenIssueTime` ("Revoke active sessions"). The break-glass role is pre-staged.

**EC2 forensics:** memory first, disk via a CMK-keyed snapshot, read-only mount.
- Memory capture needs SSM reachability, so no-egress isolation needs the `ssm`, `ssmmessages`, `ec2messages` endpoints.
- You cannot share an `aws/ebs`-encrypted snapshot cross-account. Re-copy under a CMK (grant `Decrypt`, `DescribeKey`, `CreateGrant`, `ReEncrypt*`), then double-copy under a forensic-owned CMK.
- Mount `-o ro` and store in COMPLIANCE Object Lock.

**Compromised credentials:** the session-kill tool matches the principal.
- Role session → `aws:TokenIssueTime`. IAM user `GetSessionToken` → `aws:userid` (no console button). Root → credential rotation only.
- AWS's quarantine policy is damage-limiting, not remediation. Deactivating an AKIA key does not kill live ASIA sessions.
- Containment is not eradication, so hunt persistence (extra keys, backdoor users, new roles, altered trust). The real fix is temporary credentials via roles.

**Detective (scoping):** correlate before you act.
- One finding is not one incident (Extended Threat Detection plus finding groups).
- Detective is fast scoping, Athena is ad-hoc deep-dives.
- Severity is priority, not truth, so validate against the entity's baseline before destructive action.

**Automated remediation:** a pipeline with guardrails.
- Automation rules change the finding, EventBridge changes the resource.
- Fully automatic destructive action on unfiltered findings is a self-inflicted outage: filter `Sample` false, scope by tag and account, gate destructive steps behind a custom action.
- Cross-account via `AssumeRole`, never hardcoded creds. ASR is the named org-wide solution.

**IR plan validation:** demonstrated, not documented.
- FIS stop conditions (CloudWatch alarms) are mandatory. Resilience Hub finds gaps, FIS runs experiments, ARC proves recovery.

**Compromised root:** root sits above IAM.
- No identity policy, boundary, or `TokenIssueTime` touches root. An SCP reaches member-account root only, never the management account.
- Centralized root removes standing credentials, a root-deny SCP breaks `sts:AssumeRoot`, and task-scoped sessions handle the common tasks.

**Malware and ransomware:** rebuild clean, and immutability is survival.
- Malware Protection for EC2 is agentless and not continuous. The latest backup may be infected, so recover from a verified-clean point and rebuild from a golden AMI.
- S3 ransomware: encryption at rest is not a defense (SSE-C is the weapon). Object Lock COMPLIANCE, versioning, and MFA Delete are survival, and you revoke access before restoring.
---

## What each control cannot do

- **Security group changes** cannot terminate a live, established connection. Connection tracking keeps the existing flow alive, so an active SSH session survives the rule removal. Only a stateless control (NACL deny) drops packets belonging to an already-tracked session.
- **Network isolation** cannot stop exfiltrated credentials. Once temporary keys leave the instance, they work from the attacker's own infrastructure. Quarantining the instance does nothing to them.
- **Deactivating an access key** does not kill sessions minted from it. **Deleting a role** does not invalidate issued tokens. Only `aws:TokenIssueTime` (role sessions) or `aws:userid` (user session tokens) does.
- **Session revocation** is not a one-time check at issuance. The deny policy is evaluated on every API call, so cached or exfiltrated credentials fail on next use. The real residual risk is re-assumption, not caching.
- **Host-level firewall rules** on a compromised instance cannot be trusted. The attacker owns that OS.
- **An SCP** cannot restrict an external account's principals. Stolen credentials belonging to an outside account are out of its jurisdiction entirely.
- **GuardDuty** cannot invoke Lambda directly. Every automated response is EventBridge in the middle.
- **A bucket policy denying all principals** stops the attacker and everyone else. Correct only when the requirement accepts a full outage.

## Containment decision table

| Situation | Right lever | Wrong-but-tempting |
|---|---|---|
| Exfiltration using temporary creds from a compromised instance profile | Revoke sessions (`aws:TokenIssueTime`) | Quarantine SG, blanket bucket deny |
| Live interactive session must be severed now | Stateless NACL deny on the subnet | Strip the security group rules |
| Exfiltration to a bucket outside the org, instances in a private subnet | S3 gateway **endpoint policy** with `aws:PrincipalOrgID` and `aws:ResourceOrgID` | Instance-profile IAM policy, SCP |
| Compromised IAM Identity Center user, all accounts | Disable access in Identity Center | Disable an IAM user in the management account, remove permission sets one by one |
| Exposed long-term access key in a public repo | Deactivate the key, then read `access_key_last_used_*` in the credential report (or CloudTrail) | Access Analyzer, "GuardDuty blocking rule" |

## Security Hub custom action wiring (order matters)

1. Create the custom action in Security Hub, which mints its ARN.
2. Create the EventBridge rule matching `aws.securityhub` plus that action ARN, targeting the Lambda function.
3. Select findings in Security Hub and invoke the action, choosing the finding type that matches what the function acts on (an EC2 instance finding for instance quarantine, not a security group finding).

## DR and recovery arithmetic

- Check backup or snapshot cadence against the stated RPO before anything else. Daily backups fail a 1-hour RPO, 4-hour snapshots fail a 1-hour RPO, and no amount of good architecture in the rest of the option rescues it.
- **Continuous replication of physical and virtual servers with a tight RTO** → AWS Elastic Disaster Recovery. Staging area keeps ongoing cost low, full instances launch only at failover or drill.
- AMIs are point-in-time and never satisfy "continuously replicate". AWS Backup is scheduled recovery points, not continuous.
- Recovery needs both data (backups at RPO cadence) and infrastructure (CloudFormation templates in source control). An option with only one of the two is incomplete.

## Question triggers

- "Immediately prevent further use of credentials already in an attacker's hands" → revoke the sessions, not the network path.
- "Preserve evidence" anywhere in the stem → memory before disk, never terminate first, read-only mount, COMPLIANCE Object Lock.
- "Collect everything this principal did across all accounts in the last N days" → CloudTrail Lake query if an event data store exists, otherwise Athena over the org trail bucket.
- "Automatically isolate an instance on a confirmed malware finding" → EventBridge rule on the GuardDuty finding type invoking Lambda that swaps to an isolate security group.