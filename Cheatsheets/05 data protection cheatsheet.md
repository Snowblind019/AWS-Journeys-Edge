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