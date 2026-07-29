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
---

## Which mechanism for which identity

| Identity | Mechanism | Tell in the stem |
|---|---|---|
| Workforce users, many accounts, SAML IdP | IAM Identity Center permission sets, session duration limit | "SAML", "temporary credentials", "organization" |
| External app end users | Cognito user pool (authn) plus identity pool (authz to AWS) | "external users", "minimize user database management" |
| On-premises or non-AWS servers | IAM Roles Anywhere with X.509 from a trust anchor | "certificate-based", "on-premises", "remove the bastion" |
| CI/CD in GitHub | OIDC provider plus role trust on `aud` and `sub` | repo name in the stem |
| An existing IAM user needing MFA-conditioned CLI | `sts get-session-token --serial-number --token-code` | policy uses `aws:MultiFactorAuthPresent` |
| Cross-account human or app access | Role in the **resource-owning** account, trust naming the caller's account | "cross-account" |
| Temporary use of one KMS key only | KMS grant, revoked when done | "occasionally", "temporary access to the key" |

## What each mechanism cannot do

- **IAM password policy** applies only to native IAM users. It cannot touch federated Active Directory users (fix in AD Group Policy) or Cognito user pool users (fix in the user pool's own password policy). Neither an SCP nor an IAM policy can set password length anywhere.
- **`aws:MultiFactorAuthPresent`** is a property of how the session's credentials were obtained, not of whether the user owns an MFA device. Console MFA sign-in does not flag a CLI session made with long-term keys.
- **A role in the requesting account** cannot grant access to another account's resources. The role always lives where the resources live.
- **IAM Identity Center does not have identity pools.** That word is Cognito. It also supports one directory per organization and a single Region for the instance.
- **A customer managed policy referenced by a permission set is not replicated.** It must already exist, with the identical name and permissions, in every target account, or the assignment fails at provisioning. AWS managed policies never have this problem.
- **Access Analyzer** does not trace what a credential did. External access analyzers find resources shared outside the zone of trust; internal access analyzers answer "which principals inside my account or org can reach this resource". Forensic "who used this key and when" is CloudTrail or the credential report.
- **STS `AssumeRole`** gives a whole role's permission set for a bounded session. A **KMS grant** delegates specific key operations, persists until revoked, and is what AWS services themselves use to act on your key. They layer rather than compete.

## Ordered setups worth memorizing

**IAM Identity Center with an external SAML IdP** (two-way metadata exchange):
1. Obtain the SAML metadata from IAM Identity Center (ACS URL, issuer, sign-in URL).
2. Obtain the SAML metadata from the external IdP, after configuring the app there with the values from step 1.
3. Configure the external IdP as the identity source in IAM Identity Center by uploading its metadata.

SCIM auto-provisioning is a separate optional follow-on, not part of establishing the trust. An option describing "an IAM role whose trust policy specifies the IdP's API endpoint" is OIDC workload federation, a different mechanism.

**Delegating IAM Identity Center administration**, prep work in the management account:
1. Grant least privilege access to the management account.
2. Create permission sets for use only in the management account.
3. Create user assignments only in the management account.

The through-line for every delegated-admin question: the management account gets locked down and stops being where day-to-day administration happens.

**External app users across ECS microservices**:
1. Configure a Cognito user pool.
2. Create a Cognito app client for the web application.
3. Put API Gateway with a Lambda authorizer in front, validating the JWT before forwarding.

A DynamoDB table storing user credentials is the trap; it rebuilds exactly the overhead Cognito removes.

## Cross-account assume failure checklist

Three checkpoints, and password or secret-key options are always distractors because `AssumeRole` runs on top of already-valid credentials:
- Trust policy names the right account or principal, and the `ExternalId` matches if one is required (very common for third-party auditors, and a classic confused-deputy control).
- The caller's identity policy grants `sts:AssumeRole` for that role.
- The role ARN is exactly right.