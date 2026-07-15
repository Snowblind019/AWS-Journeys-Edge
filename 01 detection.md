# Cheatsheet — Domain 1: Detection

One-page mental models per service. The recurring spine: every default is a trap, Security Hub runs on Config, and telling the services apart is the whole game.

---

**The service grid (the real battleground):**
- Malicious behavior → GuardDuty.
- Sensitive data location → Macie.
- CVEs and reachability → Inspector.
- Config drift and compliance → Config.
- Post-finding investigation → Detective.

**CloudTrail:** the API audit backbone, and its defaults betray you.
- Bucket policy needs an ACL-check plus write to `AWSLogs/account`, with `aws:SourceArn` and, cross-account, `bucket-owner-full-control`.
- Global services (IAM, STS, CloudFront, Route 53) log in us-east-1, so use a multi-region trail.
- Data events and log file validation are opt-in. Detection is validation, prevention is Object Lock plus a separate logging account.

**GuardDuty:** managed threat detection, independent of your log config.
- Regional, so enable everywhere and use org auto-enable.
- Trusted IP list suppresses, threat IP list alerts (one trusted list per account). Suppression rules archive, not delete.
- Findings publish to EventBridge (`aws.guardduty`) for SNS, Lambda, or Security Hub.

**Security Hub:** aggregates findings and scores standards, on top of Config.
- CSPM controls depend on Config recording, so `NOT_AVAILABLE` or a wrong score is a Config problem first.
- Disable a control (stop checking and paying) vs suppress via an automation rule (mute, kept for audit).
- Automation rules change the finding, EventBridge changes the resource. FSBP is the recommended baseline.

**Config:** the configuration-state and compliance-as-code backbone.
- The recorder must be continuous and record all plus global types (feeds Security Hub and Firewall Manager).
- Remediation runs under a separate `AutomationAssumeRole`, so silent failure is that role.
- Change-triggered vs periodic rules, an aggregator is visibility not enforcement, org conformance packs enforce.

**CloudWatch Logs:** the log pipeline, three ways to act.
- CloudTrail to CloudWatch Logs is a separate path from S3.
- Metric filter plus alarm notifies (forward only), Logs Insights investigates (historical), a subscription filter forwards events to a SIEM.
- Data protection masks PII at ingest, revealed only with `logs:Unmask`, which is mask-at-ingest vs Macie's scan-at-rest.

**Security Lake:** normalizes many sources into OCSF in S3.
- Query access (SQL) vs data access (SQS plus your own code).
- Athena (one-off, partition on time) vs OpenSearch (frequent, low-latency).
- OCSF is Security Lake and the new Security Hub, ASFF is Security Hub CSPM.

**Service logging failures:** the resource exists but cannot write.
- API Gateway account-level role, Lambda execution role `logs:` perms, CloudFront latency and destination, Route 53 us-east-1 plus resource policy, Flow Logs delivery role or bucket policy.
- Flow Logs never capture Amazon DNS resolver, DHCP, metadata, or Windows activation.