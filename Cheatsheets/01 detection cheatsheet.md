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
---

## What each service cannot do

- **GuardDuty** cannot block, quarantine, or remediate anything. It emits findings only, and every automated response runs through EventBridge. There is no "GuardDuty rule" that blocks an IP or an access key.
- **GuardDuty DNS findings** cannot fire if the VPC DHCP option set points at a custom resolver (OpenDNS, Google, on-prem). Those queries never touch the Amazon-provided resolver, so there is no log to analyze, and the control test silently produces nothing.
- **GuardDuty IP lists** are not JSON. Plaintext (one IPv4 or CIDR per line) or a threat-intel format (STIX, OTX_CSV, ALIEN_VAULT, PROOF_POINT, FIRE_EYE). The file must live in S3 first; there is no direct upload and no paste-in-console path. The API request body is JSON, the list file is not.
- **Inspector** cannot scan S3 buckets, IAM roles, or data. It covers EC2, container images in ECR, and Lambda, for CVEs and network reachability. "Run an Inspector assessment on an IAM role" is not a real operation.
- **Macie** cannot log or track object access. It classifies content at rest in S3 only, and it has no Trusted Advisor integration.
- **Config** cannot detect behavior. It sees configuration state, so "unusual API volume", "malicious activity", or "who accessed this object" are all outside it.
- **Detective** cannot prevent or remediate. It correlates and visualizes for scoping, and it depends on GuardDuty being enabled.
- **Trusted Advisor** is not a findings destination. Nothing publishes into it.
- **CloudWatch alarms** cannot pattern-match a named API event. Alarms fire on metric thresholds; matching `StopLogging` or a finding type by name is EventBridge's job.
- **CloudTrail** does not carry WAF traffic logs, S3 object access by default, or any service's data-plane records unless data events are explicitly enabled.

## Log source to query engine (the pairing that gets swapped)

- Trail to **S3** → Athena (partition projection for date-partitioned prefixes, no crawler needed).
- Trail to **CloudWatch Logs** → Logs Insights, metric filters, subscription filters.
- Organizational **event data store** (CloudTrail Lake) → CloudTrail's own SQL query. This is the tell: if the scenario says an event data store exists, Athena and Logs Insights are both distractors.
- **WAF logs** go to S3, CloudWatch Logs, or Firehose, chosen on the web ACL. Never through CloudTrail.
- **S3 access detail** with identity plus timestamp, structured and near real time → CloudTrail data events. S3 server access logging is best-effort, delayed by hours, and flat text.
- **API Gateway access patterns**, least effort → stage access logging plus Logs Insights. S3 plus Athena is the higher-setup answer, correct only when retention or SQL joins are emphasized.
- **API call rate anomalies** (unusual volume of deletes, spike in a principal's activity) → CloudTrail Insights, which baselines automatically. A metric filter is fixed-threshold counting, not baselining.
- **Detection rules over ingested logs with alerting to SNS** → OpenSearch Service Security Analytics.

## Question triggers

- "Suspicious or malicious activity, across the org, centrally" → GuardDuty with a delegated administrator in the security account, findings to EventBridge to SNS.
- "Inventory of sensitive data across accounts, visible in one place" → Macie delegated admin publishing to Security Hub.
- "Turn CloudTrail back on automatically if it is disabled" → Config managed rule plus `AWS-EnableCloudTrail` remediation.
- "Record only the latest configuration after several rapid changes" → Config (configuration items consolidate), not CloudTrail's per-call log.
- "Correlate findings from multiple detection services to see a multi-stage attack" → Security Hub custom insights.
- "Investigate one principal or resource, root cause, indicators of compromise" → Detective.
- "Earliest detection of a cost increase" → Cost Anomaly Detection (ML baseline, pushes to SNS). Cost Explorer is investigation and forecasting, Budgets is a planned-threshold alert, and any "review daily" option loses on speed.
- "Logs lost when the Auto Scaling group scales in" → CloudWatch agent in the AMI streaming continuously. Any periodic copy job leaves a loss window.