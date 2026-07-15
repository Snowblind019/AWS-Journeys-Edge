# AWS-Journeys-Edge

Study log and notes for the **AWS Certified Security – Specialty (SCS-C03)** exam. Worked through domain by domain, each subdomain backed by hands-on labs in a real AWS account. This is a working repository, not a polished course. Notes are written to be technically correct and useful for review, not to look good.

## What this repo is

This is in relation to my main repo **[AWS Security Specialty - The Long Road](https://github.com/Snowblind019/AWS-Security-Specialty-The-Long-Road)**. This specific repo is where the hands-on SCS work is getting documented. After my third attempt on the SCS, I came up about five questions short, and the reason was clear. I know the theory, but I don't yet have the low-level, built-it-myself knowledge to confidently pick the "more correct" answer when two or three options all look right. Qualifiers like *most cost efficient*, *least overhead*, and *most secure* are exactly where that gap shows up.

I created a plan to fix this, and the CCNA, which I had to get for work, helped me iron it out. Studying it subdomain by subdomain, building each objective out by hand before drilling questions on it is what helped me out the most. So I'm pointing that same method straight at the SCS, one subdomain at a time:

- Build the subdomain out in AWS by hand instead of just reading about it.
- Generate practice questions for it, then build out every answer to see which work and which fail.
- Document why each correct answer is correct and each wrong answer is wrong.

To help with this, I had Claude break each domain down in a way that fits my learning style best and so 4.1 in my lab doesn't correlate with 4.1 on the official AWS exam.

The goal isn't only to pass. It's to become a Cloud Security Engineer who can actually create and break things, not just recite them. The CCNA proved the method works. Now I'm running it back for the SCS.

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
- [x] **06 — Security Foundations and Governance** (14%)

Subdomain-level checklists live in each domain folder.

## Repository hygiene

This is a public security repo, so it contains no real account data. All example commands use placeholder values:

- Account IDs shown as `111122223333`
- ARNs, key IDs, IAM user names, and IPs are dummies
- Real secret keys never appear here and are rotated immediately if ever exposed

Screenshots, if added, have the console account ID cropped out of frame.

## Journal

Chronological, one entry per domain in completion order. Per-subdomain detail lives in each domain folder's README.

### 2026-06-23
Domain 4 (IAM) done. Six labs from role trust and STS AssumeRole through policy evaluation, SCPs and RCPs, cross-account access and the confused deputy, federation and Identity Center, to ABAC. The spine: the trust policy is the gatekeeper, same-account is a union while cross-account is an intersection, and an explicit deny always wins.

### 2026-06-26
Domain 5 (Data Protection) done. Eight labs across KMS key policies, cross-account KMS and encryption context, envelope encryption, S3 encryption at rest and in transit, Secrets Manager vs Parameter Store, Macie and ACM, rotation and BYOK and multi-Region keys, and Object Lock. The spine: a key policy behaves like a trust policy, the CMK only ever wraps the data key, and immutability plus versioning is what makes data survivable.

### 2026-07-02
Domain 3 (Infrastructure Security) done. Eight labs on security groups vs NACLs, VPC endpoints, connectivity (peering vs Transit Gateway), CloudFront with WAF and Shield, IMDSv2 and SSM, network inspection, hybrid encryption, and DNSSEC with load balancer TLS. The spine: stateful vs stateless decides everything, keep origins private, and private is not encrypted.

### 2026-07-08
Domain 1 (Detection) done. Seven labs on CloudTrail, GuardDuty, Security Hub, Config, CloudWatch Logs, Security Lake, and the per-service logging failure modes with Macie. The spine: every default is a trap (multi-region, data events, log validation), Security Hub runs on Config, and the service-confusion grid (GuardDuty vs Macie vs Inspector vs Config vs Detective) is the real battleground.

### 2026-07-11
Domain 2 (Incident Response) done. Eight labs across EC2 containment and forensics, compromised IAM credentials and root, Detective scoping, automated remediation, IR plan validation, and malware and ransomware. The spine: contain without destroying evidence, revoked credentials can outlive the role, and one finding is not one incident.

### 2026-07-14
Domain 6 (Security Foundations and Governance) done. Seven labs on the org policy family beyond SCPs, centralized root and break-glass, Control Tower, IaC guardrails, sharing and Firewall Manager, the compliance capstone, and tagging strategy. The spine: direction of control decides SCP vs RCP, tools govern either shape or presence, and a tag is only a control when a policy reads it.