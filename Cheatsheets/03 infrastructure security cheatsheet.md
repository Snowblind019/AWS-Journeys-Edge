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