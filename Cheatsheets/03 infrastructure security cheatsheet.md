# Cheatsheet — Domain 3: Infrastructure Security

One-page mental models per service. The recurring spine: stateful vs stateless decides everything, keep origins private, and private is not encrypted.

---

**Security groups vs NACLs:** a packet crosses both, and both must allow it.
- SG is stateful (return auto-allowed, allow-only, on the ENI). NACL is stateless (each direction, allow and deny, numbered, on the subnet).
- A custom NACL needs an outbound ephemeral allow (1024 to 65535) or return traffic dies.
- NACL is the only place for an explicit deny (block a bad IP). Stripping SG egress breaks instance-initiated outbound, not the reply to an inbound connection.

**VPC endpoints:** private access to AWS services, two shapes.
- Gateway endpoint (S3, DynamoDB): free, route-table-based, no on-prem or cross-Region.
- Interface endpoint (everything else): an ENI, per-hour, SG on 443, reachable from on-prem.
- Pin a bucket to a VPC with an endpoint policy (the doorway) plus a bucket policy on `aws:sourceVpce` (require the doorway).

**VPC connectivity:** a subnet is public only via an IGW route.
- IGW bidirectional, NAT outbound-only IPv4 (lives in the public subnet), egress-only IGW outbound-only IPv6.
- Peering is one-to-one and non-transitive (no edge-to-edge routing). Transit Gateway is the transitive hub, not open by default.

**CloudFront, WAF, Shield:** private origin, L7 filtering, L3/L4 DDoS.
- OAC (not OAI) locks S3 to one distribution via `AWS:SourceArn`. SSE-KMS objects also need the CloudFront principal on the KMS key policy.
- CloudFront WAF is scope `CLOUDFRONT` in us-east-1, roll rules out in Count mode, inspects 8 KB of body.
- Shield Standard is free/automatic. Advanced adds L7 and the SRT, but the SRT needs Business/Enterprise support.

**IMDSv2 and SSM:** kill the credential-theft path and remove inbound management.
- IMDSv1 plus SSRF leaks role creds. IMDSv2 needs a PUT-obtained token and a low hop limit (2 for containers).
- An account default is not enforcement; enforce with per-AMI, org declarative policy, and `ec2:MetadataHttpTokens`/`RoleDelivery`.
- Session Manager needs no inbound ports, and private-subnet use needs the `ssm`, `ssmmessages`, `ec2messages` endpoints.

**Network inspection:** three tools at three layers.
- DNS Firewall (resolver path, blocks by domain, bypassed by external resolvers or raw IPs).
- Network Firewall (AWS-managed L3-L7 engine, the backstop).
- Gateway Load Balancer (inserts a third-party appliance via GENEVE). "Whose engine": AWS-managed is Network Firewall, a named vendor is GWLB.

**Hybrid encryption:** Direct Connect is private but not encrypted.
- MACsec is L2 link encryption (dedicated-only, link not journey). IPsec VPN over DX is L3 end-to-end.
- Every Site-to-Site VPN has two tunnels.

**DNSSEC and LB TLS:** signing vs validation, security policy vs mTLS.
- DNSSEC signing is authoritative-side, validation is resolver-side. KSK is an asymmetric ECC_NIST_P256 CMK in us-east-1, and key loss yields SERVFAIL.
- ALB mTLS verify authenticates client certs at the LB. NLB has no LB-managed mTLS.
---

## AWS WAF attachment matrix (the single most useful fact here)

Attachable: **CloudFront, Application Load Balancer, API Gateway (REST), AppSync, Cognito user pool, App Runner, Verified Access**.

Not attachable: **EC2 instances, Network Load Balancer, S3 buckets, Route 53**.

Consequences that show up as whole exam questions:
- Route 53 weighted routing straight to EC2 with an SQLi problem means you must insert an ALB first, then attach the web ACL, then cut the DNS records over, then lock the instance security groups to the ALB.
- Cognito user pools have **no native geo-blocking setting**. Country filtering for sign-up comes from a WAF web ACL with a geographic match rule associated to the user pool.
- "Apply a web ACL to the EC2 instances" is always a fabricated step.

## What cannot reach what

- **VPC peering and Transit Gateway cannot reach S3, DynamoDB, or any AWS public service.** They connect VPC to VPC. Private access to a public service is a gateway or interface endpoint, full stop.
- **A gateway endpoint cannot be reached from on-premises or across Regions.** That is the interface endpoint's job.
- **Security group referencing works across peered VPCs in the same Region**, and is the correct answer whenever "only some instances need access" plus "instances are created and terminated regularly" appear together. CIDR-based rules always over-grant to the whole range.
- **CloudHSM is not governed by IAM for crypto operations.** Sharing an HSM cross-account is RAM sharing the **VPC subnet** where the cluster's ENIs live, plus a security group rule for the client IPs. There is no shareable "HSM ID" resource, and IAM roles or STS tokens are distractors.
- Same shape for **RDS/Aurora, ElastiCache, EFS, Amazon MQ, and Managed Microsoft AD**: the data plane speaks a traditional protocol (SQL, NFS, AMQP, LDAP/Kerberos, PKCS#11), so access is security groups plus the engine's own auth. IAM wraps only the management API. The tell is a non-AWS-native protocol.

## Rate-based vs geographic vs static IP blocking

- Malicious traffic described by **volume or behavior**, and legitimate users must keep working → rate-based rule (per-IP threshold over a rolling window, self-adjusting as attacker IPs rotate).
- Geo match is correct only when the stem says there are **no legitimate users** in that country, or the company wants the country blocked outright. If the app is global, geo-blocking fails the "do not block legitimate users" clause.
- Security group deny rules for hundreds of IPs are not a thing (security groups are allow-only) and would not scale anyway.

## Layer 7 DDoS specifics

- "Layer 7 DDoS, CloudFront, automated, no manual effort" → Shield Advanced with **automatic application layer DDoS mitigation enabled**, paired with a rate-based rule. Plain Shield Advanced adds visibility, cost protection, and SRT access, none of which are automated mitigation.
- CloudWatch alarms and DRT proactive engagement are both human-in-the-loop and lose to any "no manual effort" requirement.
- Network Firewall is L3/L4 in the VPC and cannot inspect HTTP arriving through CloudFront.

## Question triggers

- "Both accounts, application writes to a bucket in the other account, no public internet" → S3 gateway endpoint in the caller's VPC plus a route table update. Ownership of the bucket is irrelevant to the network path.
- "Instances churn, only some need database access across a peering connection" → security group referencing.
- "Certificate-based authentication from on-premises, remove the bastion" → IAM Roles Anywhere.
- "Stop exfiltration to an outside bucket, keep the job running, no internet egress" → endpoint policy with `aws:PrincipalOrgID` and `aws:ResourceOrgID`.