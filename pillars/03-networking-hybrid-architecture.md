# 🌐 Pillar 03 — Networking & Hybrid Architecture

> **Networking is the connective tissue of the cloud. Master VPC design, hybrid connectivity, and global routing.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Design multi-AZ VPCs with public/private subnets and NAT Gateways
- [ ] Implement VPC peering, Transit Gateway, and Cloud WAN for multi-VPC connectivity
- [ ] Configure Direct Connect for on-premises hybrid connectivity
- [ ] Set up AWS Site-to-Site VPN as backup connectivity
- [ ] Design Route 53 DNS strategies for failover and latency-based routing
- [ ] Implement VPC Flow Logs and VPC Endpoints for private access
- [ ] Understand AWS Global Accelerator vs CloudFront vs API Gateway
- [ ] Plan for network segmentation using security groups and NACLs
- [ ] Design cross-region and multi-region networking architectures

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **VPC** | Virtual Private Cloud — network isolation | Foundation of all AWS networking |
| **Transit Gateway** | Centralized hub for VPC and on-prem connectivity | Simplifies multi-VPC mesh |
| **Direct Connect** | Dedicated network connection from on-premises | Low-latency, high-throughput hybrid |
| **Site-to-Site VPN** | Encrypted tunnel over public internet | Backup/fallback connectivity |
| **Route 53** | DNS service with routing policies | Global traffic management |
| **VPC Endpoints** | Private connectivity to AWS services | Prevents data leaving VPC |
| **VPC Flow Logs** | Network traffic capture | Security monitoring, forensics |
| **AWS Cloud WAN** | Global network fabric | Multi-region and multi-account networking |
| **AWS PrivateLink** | Private connectivity to services | Secure access to shared services |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Enterprise VPC Design

```
┌──────────────────────────────────────────────────────────────────┐
│  VPC Architecture Template                                      │
│                                                                  │
│  ┌───────────────────────────────────────────────────────┐      │
│  │  VPC: 10.0.0.0/16                                    │      │
│  │                                                      │      │
│  │  AZ-1 (10.0.1.0/24)  │  AZ-2 (10.0.2.0/24)  │  AZ-3  │      │
│  │                      │                          │      │      │
│  │  Public Subnet      │  Public Subnet         │      │      │
│  │  (ALBs, Bastions)   │  (ALBs, Bastions)      │      │      │
│  │  NAT GW             │  NAT GW                │      │      │
│  │                      │                          │      │      │
│  │  Private Subnet     │  Private Subnet        │      │      │
│  │  (App servers)      │  (App servers)         │      │      │
│  │  RDS Subnet         │  RDS Subnet            │      │      │
│  │  (Read replicas)    │  (Read replicas)       │      │      │
│  │  Elasticache        │  Elasticache           │      │      │
│  │                      │                          │      │      │
│  │  Isolated Subnet    │  Isolated Subnet       │      │      │
│  │  (DynamoDB/local)   │  (No public access)    │      │      │
│  └───────────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────────────┘
```

### Workflow: Transit Gateway Multi-VPC Mesh

```hcl
# Terraform: Transit Gateway Central Hub

resource "aws_ec2_transit_gateway" "main" {
  description         = "Central transit gateway"
  auto_accept_shared_attachments = "enable"
  dns_support         = "enable"
  vpn_ecmp_support    = "enable"

  tags = {
    Name = "transit-gateway-main"
    Environment = "production"
  }
}

# Attach production VPC
resource "aws_ec2_transit_gateway_vpc_attachment" "prod" {
  subnet_ids         = ["subnet-prod-1", "subnet-prod-2"]
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = var.prod_vpc_id

  tags = {
    Name = "tgw-prod-attachment"
  }
}

# Attach development VPC
resource "aws_ec2_transit_gateway_vpc_attachment" "dev" {
  subnet_ids         = ["subnet-dev-1", "subnet-dev-2"]
  transit_gateway_id = aws.ec2_transit_gateway.main.id
  vpc_id             = var.dev_vpc_id

  tags = {
    Name = "tgw-dev-attachment"
  }
}
```

### Workflow: Route 53 Failover Configuration

```yaml
# CloudFormation: Route 53 Failover Records

Resources:
  PrimaryFailoverRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneId: Z1234567890
      Name: app.example.com
      Type: A
      SetIdentifier: primary-region
      Failover: PRIMARY
      AliasTarget:
        DNSName: primary-alb.us-east-1.elb.amazonaws.com
        HostedZoneId: Z2...
        EvaluateTargetHealth: true

  SecondaryFailoverRecord:
    Type: AWS::Route53::RecordSet
    Properties:
      HostedZoneId: Z1234567890
      Name: app.example.com
      Type: A
      SetIdentifier: secondary-region
      Failover: SECONDARY
      AliasTarget:
        DNSName: secondary-alb.us-west-2.elb.amazonaws.com
        HostedZoneId: Z3...
        EvaluateTargetHealth: true
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-overview.html)
- [Transit Gateway User Guide](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-aws-transit-gateway.html)
- [Direct Connect User Guide](https://docs.aws.amazon.com/directconnect/latest/UserGuide/dx-getstarted.html)
- [Route 53 Developer Guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/route-53-concepts.html)
- [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)

### 🏗️ Well-Architected Framework

- [Networking Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/networking-pillar/welcome.html)
- [Security Pillar — Network Isolation](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/network-segmentation.html)

### 🛠️ Hands-On Tutorials

- [VPC Workshop](https://vpcworkshop.awscloud.com/) — Interactive VPC design
- [Transit Gateway Deployment Guide](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-tutorial.html)
- [Route 53 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [VPC Flow Logs Walkthrough](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)

### 📄 Whitepapers

- [Network Architecture for Highly Sensitive Workloads on AWS](https://d1.awsstatic.com/whitepapers/aws-networking-architecture-for-highly-sensitive-workloads.pdf)
- [Disaster Recovery on AWS](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/direct-connect-for-disaster-recovery.html)

---

## 🏋️ Practical Exercises

### Exercise 1: Build a Multi-AZ VPC from Scratch
1. Create a VPC with CIDR 10.0.0.0/16
2. Create 3 public subnets (one per AZ) with Route Tables pointing to IGW
3. Create 3 private subnets with Route Tables pointing to NAT GWs
4. Deploy 2 NAT Gateways (one per AZ for redundancy)
5. Place an ALB in the public subnets, an EC2 in private, and test connectivity

### Exercise 2: Transit Gateway Mesh
1. Deploy 3 VPCs (Web, App, Data) in different accounts
2. Create a Transit Gateway in a shared services account
3. Attach all VPCs and configure routing
4. Add a Direct Connect or VPN attachment
5. Verify cross-VPC connectivity using traceroute/ping

### Exercise 3: Route 53 Global Failover
1. Deploy identical stacks in us-east-1 and us-west-2
2. Create a Route 53 Hosted Zone with failover routing
3. Set up health checks for each ALB
4. Simulate a region outage — verify DNS fails over to secondary
5. Practice latency-based routing for a global application

### Exercise 4: PrivateLink Shared Service
1. Create a "shared services" VPC with a private ALB exposing internal APIs
2. Create 2 consumer VPCs
3. Set up VPC Endpoints (Gateway + Interface) for S3 and the shared API
4. Verify consumer VPCs can reach the shared service without internet
5. Write Terraform modules for the endpoint setup

---

## 💡 Pro Tips

- **Always use multiple NAT Gateways** — Single AZ = single point of failure
- **Prefer Transit Gateway over VPC peering at scale** — Peering doesn't scale beyond 125 VPCs
- **Use VPC Endpoints instead of NAT for AWS services** — Saves money, keeps traffic private, faster
- **Security Groups are stateful, NACLs are stateless** — SGs for instance-level, NACLs for subnet-level
- **VPC Flow Logs → S3 → OpenSearch → CloudWatch Alarms** — Classic monitoring pipeline
- **Use separate CIDRs for each VPC** — Even between peered VPCs, avoid overlap

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Designed and deployed multi-AZ VPC
- [ ] Configured NAT Gateways and route tables
- [ ] Set up Transit Gateway attachments
- [ ] Configured Direct Connect / VPN
- [ ] Implemented Route 53 routing policies
- [ ] Deployed VPC Endpoints
- [ ] Set up VPC Flow Logs
- [ ] Configured VPC peering between accounts
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
