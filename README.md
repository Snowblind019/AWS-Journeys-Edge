# AWS Journeys Edge

![Coverage](https://img.shields.io/badge/Coverage-All%206%20Domains-brightgreen)
![Exam](https://img.shields.io/badge/SCS--C03-Passed%20Aug%202026-success)

Hands-on lab series for the **AWS Certified Security Specialty (SCS-C03)**. Every domain and subdomain of the blueprint built out by hand in a real AWS account, then documented. This is a working repository, not a polished course. Notes are written to be technically correct and useful for review, not to look good.

**I passed the SCS-C03 on August 4, 2026, on the fourth attempt.** The first three were studied rather than built. This repo is what the fourth one was built on, so if you are deciding whether the method below is worth the time: it is what worked, and nothing before it did.

Companion to my main repo, **[AWS Security Specialty: The Long Road](https://github.com/Snowblind019/AWS-Security-Specialty-The-Long-Road)**, which covers the full path and what each failed attempt taught me.

## The method

Take one subdomain at a time and:

1. Build it out in AWS by hand instead of reading about it.
2. Generate practice questions for it, then build out every answer option to see which work and which fail.
3. Document why each correct answer is correct and each wrong answer is wrong.

That last step is the point of the whole exercise. The SCS routinely offers two or three options that are all technically correct, where only one wins on the qualifier: most cost efficient, least overhead, fastest, most secure. Discriminating between them takes low-level, built-it-myself knowledge rather than theory. This repo is where I built it.

The method came from the CCNA, which I took for work. Going objective by objective, building each one out before drilling questions on it, is what carried me over the line there. This is the same method pointed at the SCS.

Domains here are broken into subdomains that suit how I learn, so lab 4.1 does not correspond to objective 4.1 on the official AWS blueprint. Coverage is complete either way.

## Layout

```
AWS-Journeys-Edge/
├── README.md
├── gotchas.md    # running list of exam traps and lab surprises
├── cheatsheets/  # one-page mental model per domain
├── 01-detection/
├── 02-incident-response/
├── 03-infrastructure-security/
├── 04-iam/
├── 05-data-protection/
└── 06-security-foundations-governance/
```

Each lab carries diagrams, CLI walkthroughs, the gotchas that only surface on deployment, and checkpoint questions for recall.

## Coverage

All six scored domains are complete, listed by exam weight.

| Domain | Weight | Labs | Status |
|--------|--------|------|--------|
| 04 Identity and Access Management | 20% | 6 | Complete |
| 03 Infrastructure Security | 18% | 8 | Complete |
| 05 Data Protection | 18% | 8 | Complete |
| 01 Detection | 16% | 7 | Complete |
| 02 Incident Response | 14% | 8 | Complete |
| 06 Security Foundations and Governance | 14% | 7 | Complete |

Subdomain-level checklists live in each domain folder.

## Repository hygiene

Public security repo, so it contains no real account data. All example commands use placeholder values:

- Account IDs shown as `111122223333`
- ARNs, key IDs, IAM user names, and IPs are dummies
- Real secret keys never appear here, and are rotated immediately if ever exposed

## Domain notes

Multiple entries per domain, in completion order. Per-subdomain detail lives in each domain folder's README.

### Domain 4, Identity and Access Management
Six labs from role trust and STS AssumeRole through policy evaluation, SCPs and RCPs, cross-account access and the confused deputy, federation and Identity Center, to ABAC. The spine: the trust policy is the gatekeeper, same-account is a union while cross-account is an intersection, and an explicit deny always wins.

### Domain 5, Data Protection
Eight labs across KMS key policies, cross-account KMS and encryption context, envelope encryption, S3 encryption at rest and in transit, Secrets Manager vs Parameter Store, Macie and ACM, rotation and BYOK and multi-Region keys, and Object Lock. The spine: a key policy behaves like a trust policy, the CMK only ever wraps the data key, and immutability plus versioning is what makes data survivable.

### Domain 3, Infrastructure Security
Eight labs on security groups vs NACLs, VPC endpoints, connectivity (peering vs Transit Gateway), CloudFront with WAF and Shield, IMDSv2 and SSM, network inspection, hybrid encryption, and DNSSEC with load balancer TLS. The spine: stateful vs stateless decides everything, keep origins private, and private is not encrypted.

### Domain 1, Detection
Seven labs on CloudTrail, GuardDuty, Security Hub, Config, CloudWatch Logs, Security Lake, and the per-service logging failure modes with Macie. The spine: every default is a trap (multi-region, data events, log validation), Security Hub runs on Config, and the service-confusion grid (GuardDuty vs Macie vs Inspector vs Config vs Detective) is the real battleground.

### Domain 2, Incident Response
Eight labs across EC2 containment and forensics, compromised IAM credentials and root, Detective scoping, automated remediation, IR plan validation, and malware and ransomware. The spine: contain without destroying evidence, revoked credentials can outlive the role, and one finding is not one incident.

### Domain 6, Security Foundations and Governance
Seven labs on the org policy family beyond SCPs, centralized root and break-glass, Control Tower, IaC guardrails, sharing and Firewall Manager, the compliance capstone, and tagging strategy. The spine: direction of control decides SCP vs RCP, tools govern either shape or presence, and a tag is only a control when a policy reads it.