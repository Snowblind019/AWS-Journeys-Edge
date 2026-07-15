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