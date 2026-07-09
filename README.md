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
- [ ] **01 — Detection** (16%)
- [ ] **02 — Incident Response** (14%)
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

### 2026-07-14

1.6 done — Security Lake. Enabled the lake with a VPC Flow source normalized to OCSF, queried the tables with Athena (partitioned on time to control cost), and drilled the right-tool decisions: query vs data access subscribers, Athena (one-off) vs OpenSearch (frequent, low-latency), Security Lake vs a plain org trail, and OCSF vs ASFF.