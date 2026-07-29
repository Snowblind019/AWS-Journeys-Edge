# Cheatsheet — Domain 5: Data Protection

One-page mental models per service. The recurring spine: the key policy behaves like a trust policy, the CMK only wraps a data key, and immutability is what makes data survivable.

---

**KMS key policy:** the key policy is the primary control and must open the door before IAM can walk through.
- Same-account: an IAM allow works only if the key policy enables IAM (`:root`) or names the principal. `:root` means the account, not the root user.
- Cross-account: both the key policy allowing the external account and the caller's IAM. Use the key ARN, not the alias.
- An explicit deny beats any key-policy allow (the Deny asymmetry).
- One policy named `default`, so always keep a controllable admin statement.

**Envelope encryption:** `GenerateDataKey`, encrypt bulk data locally, store the wrapped key, destroy the plaintext.
- Direct `kms:Encrypt` caps at 4 KB, which is why every service uses envelope encryption.
- Encryptor needs `GenerateDataKey`, decryptor needs `Decrypt`, so you can split write-only and read-only roles.
- Encryption context is authenticated (AAD), not secret, must match exactly, and AWS services set it (S3 = object ARN).

**KMS rotation, BYOK, MRK:** rotation swaps material transparently but fixes no past compromise.
- Rotating a CMK does not re-encrypt data or protect a leaked data key.
- Only symmetric AWS-origin keys auto-rotate. Imported (BYOK) keys are manual, keep a copy outside AWS, and can be crypto-shredded.
- Multi-Region keys share material and key ID but have independent policies and grants. CloudHSM key store is dedicated HSMs you control, XKS is material never in AWS.

**S3 encryption:** at-rest is automatic, the choice of encryption is a bucket policy, and in-transit is a separate control.
- New objects are always SSE-S3 encrypted. SSE-KMS adds a customer key and CloudTrail audit.
- Enforce KMS by denying `PutObject` when `x-amz-server-side-encryption` is not `aws:kms` (an absent header counts as not aws:kms).
- Require TLS with a deny on `aws:SecureTransport` false, covering both the bucket and object ARNs.

**Object Lock and integrity:** WORM and checksums, a different axis from encryption.
- Object Lock requires versioning and protects versions. Governance has an admin bypass, Compliance blocks everyone including root and cannot be shortened.
- MFA Delete is root-only and CLI-only. ETag is not an integrity hash for multipart or SSE-KMS objects, use the checksum fields.

**Secrets Manager vs Parameter Store:** choose on rotation, cross-account, and CMK.
- Secrets Manager has built-in four-step rotation and resource policies; Parameter Store Standard has neither.
- Cross-account secret read needs three: the secret resource policy, the caller's `GetSecretValue`, and `kms:Decrypt` on a customer CMK.

**Macie and ACM:** Macie finds PII in S3, ACM manages certs.
- Macie is S3-only, automated discovery samples (cheap), jobs scan exhaustively (audit).
- CloudFront cert must be in us-east-1, ALB/NLB in the LB's Region. DNS validation auto-renews, imported and email-validated certs do not.
---

## The SSE grid (separation of duties lives here)

| Mode | Who holds the key | Customer-controlled key policy | Use when |
|---|---|---|---|
| SSE-S3 | AWS, invisible | No | Simplicity, no key-access control needed |
| SSE-KMS, AWS managed (`aws/s3`) | AWS, in KMS | No custom policy | KMS audit trail without key ownership |
| SSE-KMS, customer managed | You | Yes | **Separation of duties**, rotation control, revocation by disabling the key |
| DSSE-KMS | You | Yes | Regulatory dual-layer encryption |
| SSE-C | You, never stored by AWS | N/A | Zero AWS key storage; lose it and the object is unreadable |

- **Separation of duties is the signature phrase for customer managed SSE-KMS.** Operations owns the bucket policy, security owns the key policy, and neither alone can hand out plaintext because decrypt requires permission on both sides.
- SSE-S3 has no key policy, so it can never satisfy a two-team split.
- SSE-C keys are supplied per request over HTTPS and are not stored in KMS. "Store the customer-provided keys in KMS" is a fabricated workflow.
- Cost and throughput caveat for SSE-KMS: every encrypt and decrypt is a KMS API call with request quotas and per-request cost.

## Cross-Region and immutability

- **AWS managed keys cannot be shared or replicated across Regions.** Only customer managed keys support multi-Region keys (shared material and key ID, independent policies). Secrets Manager cross-Region secret replication pairs with a multi-Region CMK.
- "Must work if only one Region is available" rules out any design where the second Region calls back to the first Region's endpoint. Each Region needs a locally usable copy.
- **Compliance mode** blocks deletion by everyone including the account root, cannot be shortened, and has no bypass. **Governance mode** always has an override for a principal with the right permission (`s3:BypassGovernanceRetention`, or removing an AWS Backup vault lock). Any requirement phrased as "administrators cannot delete this" means compliance mode.
- Object Lock configuration and retention metadata **replicate with S3 Replication**, so locking the source protects the destination copies too.
- S3 versioning alone does not stop a deliberate permanent delete of a specific version, and a bucket policy is not a hard immutability guarantee because a sufficiently privileged principal can edit it.
- **Immutable compliance evidence** always means S3 Object Lock. CloudWatch Logs and DynamoDB have no WORM equivalent. Sequencing matters too: to capture a resource's creation and origin, the trail and event selector must exist **before** the resource is created.

## Question triggers

- "Two teams, one owns buckets, one owns keys, a mistake by either must not expose plaintext" → SSE-KMS with a customer managed key, bucket policy from ops, key policy from security.
- "Temporary, occasional access to a key for a software process, least overhead" → KMS grant, revoked afterward. Editing the key policy each cycle is the more manual distractor.
- "Object-level access detail with identity and timestamp, structured, near real time" → CloudTrail data events, not S3 server access logging.
- "Replicate secrets to a second Region, minimize latency, survive one Region being down" → customer managed multi-Region KMS key plus Secrets Manager replication.
- "Protect from permanent deletion even by admins, plus cross-Region DR" → Object Lock in compliance mode on the source, replication to the second Region.
- "Presigned URLs must be valid for the full hour" → sign with credentials from an explicit `AssumeRole` call, because a presigned URL dies with the session that signed it, and instance profile credentials rotate on AWS's schedule, not yours.