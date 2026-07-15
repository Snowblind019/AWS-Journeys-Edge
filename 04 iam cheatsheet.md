# Cheatsheet — Domain 4: Identity and Access Management

One-page mental models per service. The recurring spine: gather all policies, an explicit deny wins, and same-account is a union while cross-account is an intersection.

---

**IAM policy evaluation:** collect every applicable policy and take the intersection, with any explicit deny short-circuiting.
- Order: explicit deny anywhere → boundary ceiling → allow from identity OR resource (same account) → else implicit deny.
- Same-account is a union (allow on either side). Cross-account is an intersection (both sides).
- A permissions boundary and an SCP are ceilings, they never grant.
- Decision rule: "has an Allow but the call fails" is an explicit deny, a boundary, or an SCP missing the action. "No identity policy but it works" is a resource policy granting it.

**STS and trust policies:** the trust policy is the door, identity permissions are the key, and cross-account needs both.
- `AssumeRole` for IAM principals, `AssumeRoleWithWebIdentity` for OIDC/Cognito/mobile, `AssumeRoleWithSAML` for enterprise SAML, `GetSessionToken` for an existing user wanting MFA temp creds.
- `:root` in trust delegates to IAM; a named principal is sufficient alone.
- Confused deputy: `ExternalId` for a third-party/SaaS principal, `aws:SourceArn`/`aws:SourceAccount` for an AWS service principal.
- Decision rule: cross-account assume fails despite the trust naming you means your identity policy lacks `sts:AssumeRole`.

**SCP and RCP:** org guardrails that cap, never grant. SCP caps your principals, RCP caps your resources.
- SCP is principal-side ("stop my identities reaching untrusted resources"). RCP is resource-side ("stop untrusted identities reaching my resources").
- Neither restricts the management account. RCPs must except `aws:PrincipalIsAWSService` for service-linked roles.
- Decision rule: an external threat, including a bucket policy granting an external account, is an RCP.

**Federation and Identity Center:** federation changes the principal, not the model; the trust condition on token claims is the real fence.
- GitHub OIDC: `aud` proves GitHub, `sub` proves your repo. No `sub` means anyone's repo can assume.
- Cognito: user pool authenticates, identity pool authorizes to AWS.
- Identity Center is the many-account workforce answer, console-enabled, with permission sets provisioning `AWSReservedSSO_` roles.

**ABAC:** one policy that compares a principal tag to a resource tag, so access scales by tagging.
- `${aws:PrincipalTag/x}` equals `aws:ResourceTag/x` is the signature.
- Principal tags come from user tags or session tags, never a role's own resource tags.
- Always deny editing the authorization tag key (`aws:TagKeys`) or the model collapses.
- Decision rule: "permissions should scale as we add teams without new policies" is ABAC, a fixed set of job functions is RBAC.