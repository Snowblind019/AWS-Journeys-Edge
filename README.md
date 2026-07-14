# AWS-Journeys-Edge

Study log and notes for the **AWS Certified Security – Specialty (SCS-C03)** exam. Worked through domain by domain, each subdomain backed by hands-on labs in a real AWS account. This is a working repository, not a polished course. Notes are written to be technically correct and useful for review, not to look good.

## What this repo is

This is in relation to my main repo **[AWS Security Specialty - The Long Road](https://github.com/Snowblind019/AWS-Security-Specialty-The-Long-Road)**. This specific repo is where the hands-on SCS work is getting documented. After my third attempt on the SCS, I came up about five questions short, and the reason was clear. I know the theory, but I don't yet have the low-level, built-it-myself knowledge to confidently pick the "more correct" answer when two or three options all look right. Qualifiers like *most cost efficient*, *least overhead*, and *most secure* are exactly where that gap shows up.

I created a plan to fix this, and the CCNA, which I had to get for work, helped me iron it out. Studying it subdomain by subdomain, building each objective out by hand before drilling questions on it is what helped me out the most. So I'm pointing that same method straight at the SCS, one subdomain at a time:

- Build the subdomain out in AWS by hand instead of just reading about it.
- Generate practice questions for it, then build out every answer to see which work and which fail.
- Document why each correct answer is correct and each wrong answer is wrong.

To help with this, I had Claude break each domain down in a way that fits my learning style best and so 4.1 in my lab doesn't correlate with 4.1 on the official AWS exam.

The goal isn't only to pass. It's to become a Cloud Security Engineer who can actually create and break things, not just recite them. The CCNA proved the method works. Now I run it back for the SCS.

## How this repo is organized

```
AWS-Journeys-Edge/
├── README.md
├── gotchas.md   # running list of exam traps and lab surprises
├── cheatsheets/ # one-page mental models per service
└── domains/
    ├── 01-detection/
    ├── 02-incident-response/
    ├── 03-infrastructure-security/
    ├── 04-iam/
    ├── 05-data-protection/
    └── 06-security-foundations-governance/
```

Each domain folder holds one markdown file per subdomain, plus a short README listing that domain's subdomain checklist.

## Exam domains and progress

SCS-C03 has six scored domains, listed here by exam weight.

- [x] **04 — Identity and Access Management** (20%)
- [x] **03 — Infrastructure Security** (18%)
- [x] **05 — Data Protection** (18%) — in progress (KMS)
- [x] **01 — Detection** (16%)
- [x] **02 — Incident Response** (14%)
- [ ] **06 — Security Foundations and Governance** (14%)

Subdomain-level checklists live in each domain folder.

## Repository hygiene

This is a public security repo, so it contains no real account data. All example commands use placeholder values:

- Account IDs shown as `111122223333`
- ARNs, key IDs, IAM user names, and IPs are dummies
- Real secret keys never appear here and are rotated immediately if ever exposed

Screenshots, if added, have the console account ID cropped out of frame.

## Journal

Reverse chronological. Short entries, what was covered, what broke, what to revisit.

### 2026-06-10
Repo set up. Starting with Domain 4 (IAM).

### 2026-06-12
4.1 done — IAM roles, trust policies, STS AssumeRole. Built a role and broke the assume three ways to prove the trust policy, not identity permissions, is the gatekeeper.

### 2026-06-17
4.2 done — policy evaluation logic. Ran the same calls against one S3 bucket to walk the identity/resource/boundary/explicit-deny decision tree end to end.

### 2026-06-18
4.3 done — SCPs, RCPs, and Organizations. Stood up a real org and member account to prove SCPs cap above identity and never restrict the management account.

### 2026-06-19
4.4 done — cross-account access and the confused deputy. Assumed a role across accounts (both sides required) and gated it with `sts:ExternalId` and `aws:PrincipalOrgID`.

### 2026-06-23

4.5 done — federation and IAM Identity Center. Set up GitHub Actions OIDC via AssumeRoleWithWebIdentity, scoping the trust policy's sub claim to one repo so the role can't be assumed from anyone else's.

### 2026-06-23

4.6 done — attribute-based access control (ABAC). Wrote one tag-match policy (aws:PrincipalTag/project matches aws:ResourceTag/project) that isolates teams without naming resources, fed by user tags and by session tags for the federation path.

### 2026-06-23

5.1 done — KMS key policies vs IAM policies. Rewrote one key's policy three ways to prove the key policy is primary: :root delegates to IAM, dropping it kills an unchanged IAM allow, and naming a principal directly grants access with no IAM policy at all.

### 2026-06-24

5.2 done — cross-account KMS and encryption context. Proved cross-account needs both the key policy and the caller's IAM (the opposite of 5.1's key-policy-only), and that encryption context binds to the ciphertext as AAD so decrypt requires the exact same context and can be scoped by condition.

### 2026-06-25

5.3 done — envelope encryption. Hit the 4 KB CMK wall, then used GenerateDataKey to encrypt a big file locally and recover it, proving the CMK only ever wraps the data key while the bulk AES happens off-KMS.

### 2026-06-25

5.4 done — S3 encryption at rest and in transit. Confirmed SSE-S3 is the automatic baseline and SSE-KMS adds the audited CMK path, then forced KMS-only uploads with a bucket policy (an absent encryption header counts as not aws:kms, since the deny runs before S3's default applies) and required TLS via an aws:SecureTransport deny.

### 2026-06-25

5.5 done — Secrets Manager vs Parameter Store. Stored a CMK-encrypted secret and the SecureString equivalent, walked the four-step rotation contract and version stages, and proved a cross-account secret read takes three grants: the secret resource policy, the caller's GetSecretValue IAM, and kms:Decrypt on a customer-managed CMK.

### 2026-06-26

5.6 done — Macie, ACM, and in-transit enforcement. Covered Macie classification (S3-only, sampled automated vs full targeted discovery), the ACM cert lifecycle (public vs Private CA, DNS validation, us-east-1 for CloudFront, imported certs don't auto-renew), and the per-service points that force TLS rather than just allow it.

### 2026-06-26

5.7 done — KMS rotation, BYOK, and multi-Region keys. Proved rotation is transparent and non-destructive but does nothing for a leaked data key, imported my own EXTERNAL key material, replicated a multi-Region key, and slotted CloudHSM and XKS on the spectrum from AWS-managed to never-in-AWS.

### 2026-06-26

5.8 done — S3 Object Lock, legal hold, MFA Delete, and checksums. Proved governance retention has an admin bypass while compliance can't be overridden by anyone (not even root) or shortened, legal hold blocks deletion with no end date, and checksums verify integrity where ETag can't.

### 2026-07-01

3.1 done — VPC security groups vs NACLs (stateful vs stateless). Removed outbound from each layer and watched opposite results: dropping the NACL's ephemeral outbound rule broke the SSH return packet, while stripping the SG's egress didn't, since the SG auto-allows the reply to an inbound connection it already permitted.

### 2026-07-02

3.2 done — VPC endpoints (gateway vs interface). Swapped a NAT-to-public-S3 path for a free S3 gateway endpoint, then pinned the bucket to the VPC with two locks (an endpoint policy plus a bucket policy on aws:sourceVpce), and contrasted with an interface endpoint (ENI with a private IP, SG on 443, private DNS) for Secrets Manager.

### 2026-07-02
3.3 done — VPC connectivity (IGW/NAT, peering, Transit Gateway). Proved a subnet is public only via an IGW route and that NAT is outbound-only, that peering is non-transitive (A can't reach C through B), and that a Transit Gateway is the transitive hub replacing the n(n-1)/2 peering mesh.

### 2026-07-02

3.4 done — CloudFront private origin, WAF, and Shield. Locked S3 to one distribution with OAC (SigV4 plus AWS:SourceArn, after showing a public bucket and legacy OAI both fall short), added a WAF web ACL (managed, rate-based, and geo rules) in us-east-1, and placed Shield Standard against Advanced.

### 2026-07-02

3.5 done — IMDSv2 and SSM management. Reproduced the IMDSv1 SSRF that leaks role credentials on an unauthenticated GET, closed it by requiring IMDSv2 (PUT-for-token plus a low hop limit) and stacking real enforcement since an account default isn't enforcement, then swapped the SSH bastion for Session Manager: zero inbound ports, IAM-authenticated, fully logged.

### 2026-07-02

3.6 done — DNS Firewall, Network Firewall, and Gateway Load Balancer. Built a Route 53 DNS Firewall block rule (bad domain returns NXDOMAIN on the resolver path) and mapped the inspection tiers: Network Firewall as the AWS-managed L3-L7 backstop for what bypasses DNS, and GWLB for inserting a third-party vendor appliance inline via GENEVE.

### 2026-07-02

3.7 done — hybrid connectivity encryption (MACsec, IPsec over DX, VPN). Started from the wrong assumption that Direct Connect is encrypted (it is private, not encrypted), then compared MACsec at L2 (near line rate, dedicated-only, link not journey) against IPsec VPN over DX at L3 (end-to-end, ~1.25 Gbps per tunnel), with VPN over the internet as the encrypted DX failover.

### 2026-07-02

3.8 done — DNSSEC and load balancer TLS posture. Signed a hosted zone with a KSK backed by an asymmetric ECC_NIST_P256 KMS key (signing does nothing until the DS record lands at the parent, and KMS key loss means SERVFAIL), then set a TLS 1.3 listener policy and configured ALB mTLS verify against passthrough, noting NLB has no LB-managed mTLS.

### 2026-07-03

1.1 done — CloudTrail. Built the bucket policy CloudTrail requires (ACL check plus write, with bucket-owner-full-control for cross-account and aws:SourceArn as the confused-deputy guard), created a multi-region trail with log file validation, proved data events are off by default until you add a selector, then validated the digest chain and watched it flag a deleted log object.

### 2026-07-03

1.2 done — GuardDuty. Enabled a detector and generated sample findings with no flow logs or trail present, proving GuardDuty consumes its inputs independently, then set the invertible trusted (suppress) versus threat (alert) IP lists, a suppression rule that archives without deleting, org-wide auto-enable, and drilled the GuardDuty vs Macie/Inspector/Config/Detective discriminators.

### 2026-07-03

1.3 done — Security Hub. Enabled CSPM with default standards while Config wasn't recording and watched controls go NOT_AVAILABLE (a wrong score is a Config problem first), then turned on the recorder to fix it, and drilled the tested distinctions: disable vs suppress, automation rules (act inside on the finding) vs EventBridge (act outside), ASFF ingest, the cross-region aggregator, and org-wide central configuration.

### 2026-07-08

1.4 done — AWS Config. Turned on the recorder for all supported plus global types (the foundation that fed Security Hub in 1.3), added a change-triggered managed rule that flipped a 0.0.0.0/0 SSH group to NON_COMPLIANT, then broke auto-remediation with an under-permissioned AutomationAssumeRole before fixing it, and drilled managed vs custom, change-triggered vs periodic, org conformance packs, and the aggregator as visibility not enforcement.

### 2026-07-08

1.5 done — CloudWatch Logs. Wired CloudTrail into CloudWatch Logs (a separate path from the S3 copy), built a root-usage metric filter and alarm (notify), contrasted it with Logs Insights (investigate) and a subscription filter (forward events to a SIEM), then applied a data protection policy that masks PII at ingest, revealed only with logs:Unmask.

### 2026-07-08

1.6 done — Security Lake. Enabled the lake with a VPC Flow source normalized to OCSF, queried the tables with Athena (partitioned on time to control cost), and drilled the right-tool decisions: query vs data access subscribers, Athena (one-off) vs OpenSearch (frequent, low-latency), Security Lake vs a plain org trail, and OCSF vs ASFF.

### 2026-07-08

1.7 done — service logging failure modes and Macie. Worked the silent-failure cause per service (API Gateway account-level role, Lambda execution role, CloudFront latency, Route 53 us-east-1 and resource policy, Flow Logs delivery role plus the DNS and metadata coverage blind spot), then revisited Macie as the at-rest PII detector that pairs with 1.5's in-transit masking, breadth via automated discovery against depth via jobs.

### 2026-07-09

2.1 done — EC2 incident response. Built the wrong answers first (terminate destroys RAM and the EBS volume, an SG swap leaves the live SSH session up because SGs are stateful and still allow egress), then contained correctly with an empty SG plus a stateless NACL deny, revoked the role's live creds by aws:TokenIssueTime, and snapshotted for clean-room analysis, with the break-glass role pre-staged.

### 2026-07-10

2.2 done — EC2 forensics. Captured memory first from the running instance via SSM, which meant adding PrivateLink endpoints so 2.1's no-egress isolation didn't block it, then hit the cross-account trap that a default aws/ebs-encrypted snapshot can't be shared, so re-keyed under a CMK and double-copied under a forensic-owned key, mounted read-only, and stored in a COMPLIANCE Object Lock bucket.

### 2026-07-10

2.3 done — compromised IAM credentials. Proved AWS's auto-quarantine is damage-limiting not remediation (S3 reads still worked), then ran the real sequence: contain the key, kill live sessions with an explicit deny (aws:userid for user sessions, aws:TokenIssueTime for role sessions, since deactivating AKIA doesn't touch live ASIA), investigate via CloudTrail, hunt persistence, and recover by migrating off long-term keys to roles.

### 2026-07-10

2.4 done — Detective and scoping an investigation. Drove the workflow off a sample GuardDuty finding, built the wrong answers (scoping off one finding, hand-querying Athena, acting on severity alone), then ran the correct path: pivot into Detective, validate the entity against its baseline, follow the finding group to the full multi-stage sequence, pull IoCs and MITRE TTPs, and hand the scoped set to response, since one finding isn't one incident and severity is priority not truth.

### 2026-07-11

2.5 done — automated remediation. Learned the three event paths (GuardDuty Finding, Security Hub Imported for automatic, Custom Action for human-in-the-loop), built the wrong answers (auto-terminate everything is a self-DoS, an automation rule can't touch the resource, one admin Lambda with hardcoded creds), then built the correct pipeline: precise EventBridge match (Sample false, Workflow NEW), an SSM runbook under least privilege, close the loop, and a human gate for destructive actions, and placed ASR as the prebuilt org-wide answer.

### 2026-07-11

2.6 done — validating the IR plan. Framed validation as demonstrated not documented, built the wrong answers (a runbook is paper, FIS in prod with no stop conditions is a sev-1, Resilience Hub doesn't inject faults), then ran the workflow: steady state and hypothesis, sample findings for detection, scoped FIS with stop conditions for the fault side, a GameDay with success criteria, Step Functions to orchestrate, and ARC to prove recovery, keeping the Resilience Hub / FIS / ARC trio straight.

### 2026-07-11

2.7 done — compromised root user. Proved root sits above IAM (a deny policy or boundary does nothing, and neither TokenIssueTime nor aws:userid touches it, so containment is credential rotation only), that SCPs reach member-account root but never the management account, then ran the response (reset password, delete keys, reset MFA, hunt persistence and lockout moves) and set up centralized root credentials management so member accounts have no root credentials to steal.

### 2026-07-11

2.8 done — malware and ransomware. EC2 side: confirmed with an agentless on-demand scan (Malware Protection is agentless and not continuous, once per 24h, so it complements AV), then isolate, preserve, rebuild from a golden AMI, and recover from a verified-clean backup since the latest backup may be infected. S3 side: proved encryption is the ransomware weapon not the defense (SSE-C) and that immutability (Object Lock COMPLIANCE) plus versioning and MFA Delete, not replication, make it survivable, revoking credentials before restoring.

### 2026-07-12

6.1 done — Organizations policy types beyond SCPs. Proved an RCP is a resource-side upper bound by denying my own GET on a bucket I owned with admin, then built an org-perimeter RCP (and saw why it needs the aws:PrincipalIsAWSService exception or it breaks SLR access), showed why an SCP can't stop an external account a bucket policy grants (direction of control: SCP principal-side, RCP resource-side), and separated declarative policies (durable config baseline) and AI opt-out (content governance, not access) from both.

### 2026-07-12

6.2 done — centralized root, delegated admin, and break-glass. Enabled centralized root access (and saw the trap that a root-deny SCP silently blocks sts:AssumeRoot), moved root administration to a delegated security account (the same reflex that runs every security service off the management account), deleted a member's root credentials and audited zero, unlocked a deny-all bucket policy through a task-scoped root session that could do nothing else, and designed break-glass for the root-only tasks centralized access can't cover.

### 2026-07-12

6.3 done — Control Tower. Mapped the two independent axes (behavior: preventive SCP, detective Config, proactive CFN hook; guidance: mandatory, strongly recommended, elective, which never implies behavior), and drilled the traps: proactive controls only see CloudFormation (miss console, CLI, Terraform), preventive controls skip the management account (the 4.3 SCP exemption), the five-SCP-per-OU budget, and Account Factory/AFT vending accounts versus StackSets deploying resources.

### 2026-07-14

6.4 done — IaC security and StackSets. Ran the two gates (cfn-lint for validity, cfn-guard for security, since a valid template can still be insecure), unit-tested Guard rules and trimmed rulegen scaffolding, bridged to server-side CloudFormation Hooks (a Guard-backed Hook is a proactive control, with the same CloudFormation-only limit as 6.3), and rolled a baseline to an OU with a service-managed auto-deploying StackSet, keeping StackSets (deploy resources) distinct from Account Factory (create accounts).

### 2026-07-14

6.5 done — cross-account sharing and Firewall Manager. Shared an infrastructure resource into the org with RAM (in-org sharing skips invitations, external accounts must accept), drilled RAM (infra) vs resource-based policy (one data resource, two-sided handshake, KMS strictest) vs Service Catalog (self-service with a launch constraint so users deploy without underlying permissions), and mapped Firewall Manager as the org-wide network enforcement orchestrator (four prerequisites, Config the silent one), not the firewall itself.