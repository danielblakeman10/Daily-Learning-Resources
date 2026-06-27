# 🏗️ Pillar 01 — Foundation & Governance

> **Build your cloud on a solid foundation. IaC, organizational structure, and governance are the bedrock of enterprise AWS.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Explain the cloud adoption framework and organizational readiness for AWS
- [ ] Set up multi-account AWS Organizations with OU structure
- [ ] Implement SCPs (Service Control Policies) to enforce guardrails
- [ ] Deploy infrastructure as code (IaC) using CloudFormation and Terraform
- [ ] Use AWS Control Tower to establish a secure multi-account landing zone
- [ ] Manage shared services with AWS Service Catalog
- [ ] Monitor compliance with AWS Config rules
- [ ] Implement tagging strategies for cost allocation and resource management

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **AWS Organizations** | Multi-account management at scale | Centralized governance across environments |
| **Service Control Policies (SCPs)** | Service-level permission boundaries | Prevent misuse, enforce compliance |
| **AWS Control Tower** | Automated landing zone setup | Standardized, secure multi-account setup |
| **AWS CloudFormation** | Infrastructure as Code (native) | Repeatable, version-controlled deployments |
| **Terraform (HashiCorp)** | Infrastructure as Code (cross-cloud) | Industry-standard IaC, multi-provider |
| **AWS Config** | Resource inventory & compliance | Continuous monitoring, drift detection |
| **AWS Service Catalog** | Approved product catalog | Self-service with governance |
| **AWS Tag Editor** | Bulk tagging operations | Cost allocation, resource discovery |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Multi-Account Landing Zone

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: Account Structure                                │
│  ├── Create Management Account (AWS Organizations)          │
│  ├── Create OU: Sandbox, Development, Production, Audit     │
│  └── Add Accounts to OUs                                    │
├─────────────────────────────────────────────────────────────┤
│  Phase 2: Guardrails & Policies                            │
│  ├── Apply SCPs per OU (restrict regions, services)         │
│  ├── Enable CloudTrail across all accounts                  │
│  └── Enable GuardDuty in each account                       │
├─────────────────────────────────────────────────────────────┤
│  Phase 3: Landing Zone Automation                          │
│  ├── Deploy Control Tower                                  │
│  ├── Enable Guardrails (mandatory & recommended)            │
│  ├── Set up Custom Landing Zone (if needed)                │
│  └── Configure Default Logs & Metrics                       │
├─────────────────────────────────────────────────────────────┤
│  Phase 4: IaC Foundation                                   │
│  ├── Create shared resources stack (CloudFormation)         │
│  ├── Write Terraform modules for account bootstrap          │
│  └── Version control all IaC in git                         │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: Terraform Organization Bootstrap

```hcl
# providers.tf
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket         = "terraform-state-danielblakeman"
    key            = "foundation/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

# main.tf — AWS Organizations
resource "aws_organizations_organization" "this" {
  feature_set = "ALL"
  aws_service_access_principals = [
    "securityhub.amazonaws.com",
    "config.amazonaws.com",
    "control-tower.amazonaws.com"
  ]
}

resource "aws_organizations_organizational_unit" "prod" {
  name      = "Production"
  parent_id = aws_organizations_organization.this.roots[0].id
}

resource "aws_organizations_organizational_unit" "dev" {
  name      = "Development"
  parent_id = aws_organizations_organization.this.roots[0].id
}
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [AWS Organizations User Guide](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
- [AWS Control Tower Getting Started](https://docs.aws.amazon.com/controltower/latest/userguide/getting-started.html)
- [AWS Config Rules Reference](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-services.html)
- [AWS Service Catalog User Guide](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/servicecatalog_introduction.html)

### 🏗️ Well-Architected Framework

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Operations Pillar](https://docs.aws.amazon.com/wellarchitected/latest/framework/operations-pillar.html)
- [Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/framework/security-pillar.html)

### 🛠️ Hands-On Tutorials

- [AWS Control Tower Hands-On Guide](https://docs.aws.amazon.com/controltower/latest/userguide/getting-started.html)
- [AWS Config Rules — Walkthrough](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config_getstarted-tutorial.html)
- [Terraform AWS Quickstart](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)
- [CloudFormation Getting Started](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-getting-started.html)

### 📄 Whitepapers

- [AWS Multi-Account Strategy](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)
- [AWS Organizations Best Practices](https://aws.amazon.com/whitepapers/organizing-your-aws-environment/)

---

## 🏋️ Practical Exercises

### Exercise 1: Build Your Landing Zone
1. Create an AWS Organizations hierarchy: Sandbox → Dev → Staging → Prod
2. Apply SCPs that prevent resource deletion in Prod
3. Enable CloudTrail in every account
4. Deploy a "hello world" CloudFormation stack in each environment

### Exercise 2: Terraform Bootstrap
1. Write Terraform code to create 2 OUs and 4 accounts via Organizations
2. Apply SCPs per environment tier (restrict regions in Sandbox, restrict everything in Prod)
3. Store Terraform state in a remote S3 bucket with DynamoDB locking
4. Add a Git branch strategy: `main` = production, `dev` = development

### Exercise 3: Compliance-as-Code
1. Deploy 10 AWS Config managed rules across an account
2. Create custom Lambda-backed Config rules (e.g., "no unencrypted EBS volumes")
3. Set up CloudWatch Alarms for non-compliant resources
4. Export compliance data to S3 for auditing

### Exercise 4: Service Catalog Portfolio
1. Create a Service Catalog portfolio with 3 approved products
2. Add CloudFormation templates as products (VPC, S3 bucket, EC2 instance)
3. Configure launch constraints and IAM roles
4. Have a team member request and deploy a product

---

## 💡 Pro Tips

- **Tag early, tag everywhere** — Define a tag strategy before deploying anything
- **Use `terraform plan` in CI** — Never apply without reviewing the plan output
- **SCPs are a safety net, not permissions** — They deny at the org level, IAM handles per-account permissions
- **Control Tower is opinionated by design** — Override guardrails only when you have a specific business case
- **Version everything** — Infrastructure, policies, and configurations should all be in git

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Built multi-account Organization
- [ ] Applied SCPs across OUs
- [ ] Deployed landing zone with Control Tower
- [ ] Wrote CloudFormation templates
- [ ] Wrote Terraform modules
- [ ] Set up AWS Config with rules
- [ ] Created Service Catalog portfolio
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
