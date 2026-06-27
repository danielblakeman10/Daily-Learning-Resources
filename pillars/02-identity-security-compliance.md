# 🔐 Pillar 02 — Identity, Security & Compliance

> **Zero trust, least privilege, and comprehensive audit trails. Identity is the new perimeter in cloud computing.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Design IAM policies following least-privilege principles
- [ ] Implement IAM Identity Center (SSO) with SCIM provisioning
- [ ] Set up MFA enforcement and IAM Access Analyzer
- [ ] Configure AWS KMS for encryption key management
- [ ] Deploy AWS GuardDuty for threat detection
- [ ] Set up AWS Security Hub for centralized security findings
- [ ] Implement AWS Audit Manager for compliance frameworks
- [ ] Create IAM policies as code with policy analysis tools
- [ ] Design SAML/OIDC federation for enterprise identity

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **AWS IAM** | Identity and Access Management | Least-privilege access for humans & workloads |
| **IAM Identity Center (SSO)** | Centralized SSO & SCIM | Enterprise SSO, JIT provisioning, access reviews |
| **AWS KMS** | Key Management Service | Encryption keys, key policies, rotation |
| **AWS GuardDuty** | Intelligent threat detection | ML-based anomaly detection |
| **AWS Security Hub** | Security findings aggregation | Multi-account security posture |
| **AWS Audit Manager** | Compliance automation | SOC 2, HIPAA, ISO 27001, PCI DSS |
| **IAM Access Analyzer** | Find unintended sharing | External resource exposure detection |
| **AWS CloudTrail** | API activity logging | Audit trail, forensics, compliance |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Least-Privilege IAM Policy Design

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: Identity Foundation                               │
│  ├── Enable IAM Identity Center (SSO)                       │
│  ├── Connect to enterprise IdP (Azure AD / Okta / SAML)     │
│  ├── Assign permission sets per role                         │
│  └── Enable SCIM for JIT provisioning                       │
├─────────────────────────────────────────────────────────────┤
│  Phase 2: Guardrails & Enforcement                          │
│  ├── Enforce MFA for all IAM users                          │
│  ├── Set password complexity policy                            │
│  ├── Enable IAM Access Analyzer                              │
│  └── Rotate credentials on schedule                          │
├─────────────────────────────────────────────────────────────┤
│  Phase 3: Workload Identity                                  │
│  ├── Use IAM Roles for EC2 / ECS / EKS (not access keys)     │
│  ├── Implement OIDC federation for GitHub Actions           │
│  └── Use IRSA for Kubernetes service accounts               │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: OIDC Federation for CI/CD

```hcl
# Terraform: OIDC Federated Identity for GitHub Actions

resource "aws_iam_openid_connect_provider" "github" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["a031c46782e6e6c662c2c87c76da9aa62ccabd8e"] # GitHub's certificate
}

resource "aws_iam_role" "github_actions" {
  name = "github-actions-deploy"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Federated = aws_iam_openid_connect_provider.github.arn
        }
        Action = "sts:AssumeRoleWithWebIdentity"
        Condition = {
          StringEquals = {
            "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
          }
          StringLike = {
            "token.actions.githubusercontent.com:sub" = "repo:danielblakeman10/aws-platform-projects:*"
          }
        }
      }
    ]
  })
}
```

### Workflow: Security Hub & GuardDuty Integration

```yaml
# Example: CloudFormation — GuardDuty + Security Hub

Resources:
  GuardDutyDetector:
    Type: AWS::GuardDuty::Detector
    Properties:
      Enable: true

  GuardDutyFinding:
    Type: AWS::GuardDuty::IPSet
    DependsOn: GuardDutyDetector
    Properties:
      DetectorId: !GetAtt GuardDutyDetector.Id
      Format: TEXT
      SourceUrl: https://rules.emergingthreats.net/open/suricata/rules/SCAN-INFO-IP list.txt

  SecurityHubHub:
    Type: AWS::SecurityHub::Hub
    # Security Hub is enabled via AWS Control Tower guardrail
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Identity Center (SSO) Admin Guide](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
- [GuardDuty Documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_intro.html)
- [Security Hub User Guide](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-intro.html)

### 🏗️ Well-Architected Framework

- [Security Pillar FAQ](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [IAM Best Practices in Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/iam-best-practices.html)
- [Encryption Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/encryption-at-rest.html)

### 🛠️ Hands-On Tutorials

- [IAM Access Analyzer Walkthrough](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [GuardDuty Getting Started](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_welcome.html)
- [AWS KMS Crypto Utilities](https://docs.aws.amazon.com/kms/latest/developerguide/crypto-utils.html)
- [IAM Role for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)

### 📄 Whitepapers

- [AWS Identity and Access Management Best Practices](https://d1.awsstatic.com/whitepapers/aws-iam-best-practices.pdf)
- [AWS Identity Center Implementation Guide](https://docs.aws.amazon.com/singlesignon/latest/userguide/sso_idp_saml.html)

---

## 🏋️ Practical Exercises

### Exercise 1: Least-Privilege Policy Challenge
1. Write an IAM policy that allows a developer to:
   - View all S3 buckets in their account
   - Read from a specific S3 bucket (`dev-data-bucket`)
   - Launch EC2 instances in a specific subnet
   - **Not** delete anything, **not** access secrets, **not** use root
2. Test with IAM Access Analyzer
3. Use IAM Policy Simulator to validate

### Exercise 2: OIDC Federation for GitHub Actions
1. Set up AWS OIDC provider for GitHub
2. Create an IAM role with minimal permissions
3. Write a GitHub Actions workflow that assumes the role to deploy to S3
4. Add conditions: only run on `main` branch, only specific repos

### Exercise 3: Security Hub Baseline
1. Enable Security Hub in a new account
2. Enable all foundational standards (CIS AWS Foundations)
3. Review the initial security findings
4. Create a CloudWatch Events rule to alert on critical findings
5. Auto-remediate with SSM Automation for a low-severity finding

### Exercise 4: KMS Encryption Strategy
1. Create KMS CMKs for different environments (dev, prod)
2. Write key policies with separate admin and encryption-only principals
3. Enable automatic key rotation
4. Encrypt an S3 bucket using a customer-managed key
5. Test decryption with and without proper IAM permissions

---

## 💡 Pro Tips

- **Never use long-term access keys for CI/CD** — Use OIDC federation or IRSA
- **Permission boundaries + SCPs = triple protection** — Apply at multiple layers
- **Audit, don't just configure** — Use Access Analyzer and IAM Access Advisor regularly
- **SCIM > manual provisioning** — Automate user lifecycle with your IdP
- **KMS key policies are not IAM policies** — They define who can use the key, not who can manage it
- **CloudTrail logs must be encrypted and protected** — Use SCPs to prevent log deletion

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Designed least-privilege IAM policies
- [ ] Set up IAM Identity Center (SSO)
- [ ] Configured MFA enforcement
- [ ] Deployed GuardDuty in accounts
- [ ] Enabled Security Hub
- [ ] Set up Audit Manager for a compliance framework
- [ ] Implemented OIDC federation for CI/CD
- [ ] Created KMS key policies
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
