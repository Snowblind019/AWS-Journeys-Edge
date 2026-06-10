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

- [ ] **04 — Identity and Access Management** (20%)
- [ ] **03 — Infrastructure Security** (18%)
- [ ] **05 — Data Protection** (18%) — in progress (KMS)
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
Repo set up. Going to begin with Domain 4 (IAM) labs.