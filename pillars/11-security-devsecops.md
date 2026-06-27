# 🛡️ Pillar 11 — Security & DevSecOps

> **Shift-left security, compliance-as-code, and secrets management — security baked into every stage of the pipeline.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Implement secrets management with AWS Secrets Manager and Systems Manager Parameter Store
- [ ] Build shift-left security pipelines with CodeGuru, CodeBuild, and CodeCommit
- [ ] Configure AWS Inspector for vulnerability scanning
- [ ] Use AWS Macie for sensitive data discovery in S3
- [ ] Implement AWS Artifact for compliance reporting
- [ ] Design security scanning in CI/CD (SAST, DAST, container scanning)
- [ ] Configure AWS WAF for web application firewall
- [ ] Implement infrastructure security scanning (Checkov, tfsec, cfn_nag)
- [ ] Design incident response runbooks for security events

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **AWS Secrets Manager** | Secrets rotation and management | Database credentials, API keys |
| **Systems Manager Parameter Store** | Secure parameters | Environment config, secrets |
| **AWS Inspector** | Vulnerability assessment | EC2 and container image scanning |
| **AWS Macie** | Sensitive data discovery | Automated PII detection in S3 |
| **AWS Artifact** | Compliance documentation | Audit reports, SOC 2, ISO 27001 |
| **AWS WAF** | Web Application Firewall | SQL injection, XSS protection |
| **AWS CodeGuru Reviewer** | AI-powered code review | Security issues, performance |
| **Amazon Cognito** | User authentication | Sign-up, sign-in, federation |
| **KMS (Customer Master Keys)** | Encryption key management | Key policies, rotation, access control |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Shift-Left Security Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│  DevSecOps Pipeline                                         │
│                                                             │
│  Code Commit → Pre-Commit Hooks                             │
│       │                                                    │
│       ├── Trivy (Container Scanning)                       │
│       ├── Checkov (IaC Scanning)                           │
│       ├── Semgrep (SAST - Static Analysis)                 │
│       └── GitLeaks (Secret Detection)                     │
│       │                                                    │
│  Build → Post-Build Security Checks                         │
│       │                                                    │
│       ├── CodeGuru Reviewer (Code Analysis)                │
│       ├── Inspect Container Images (ECR Scanning)          │
│       ├── OWASP ZAP (DAST)                                 │
│       └── Dependency Scanning (Snyk/Dependabot)           │
│       │                                                    │
│  Deploy → Runtime Security                                 │
│       ├── Inspector (EC2 + Container Scanning)             │
│       ├── GuardDuty (Threat Detection)                    │
│       ├── Macie (Data Protection)                         │
│       └── Security Hub (Security Posture)                 │
│       │                                                    │
│  Monitor → Continuous Compliance                           │
│       ├── Config Rules (Compliance Checks)                │
│       ├── Audit Manager (Compliance Frameworks)           │
│       └── Security Hub (Security Scoreboard)             │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: Secrets Management

```hcl
# Terraform: Secrets Manager with Rotation

resource "aws_secretsmanager_secret" "database" {
  name                    = "prod/database-credentials"
  description             = "Production database credentials"
  kms_key_id              = aws_kms_key.secrets.arn
  restore_secret          = false

  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

resource "aws_secretsmanager_secret_version" "database" {
  secret_id     = aws_secretsmanager_secret.database.id
  secret_string = jsonencode({
    username = var.db_username
    password = var.db_password
    host     = var.db_host
    port     = "5432"
    dbname   = "appdb"
  })
}

# Auto-rotate every 30 days
resource "aws_secretsmanager_secret_rotation" "database" {
  secret_id           = aws_secretsmanager_secret.database.id
  rotation_lambda_arn = aws_lambda_function.rotation.arn

  rotation_rules {
    automatically_after_days = 30
  }
}

# Application access policy
resource "aws_iam_policy" "secret_access" {
  name        = "app-database-secret-access"
  description = "Allow app to read database credentials"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = "secretsmanager:GetSecretValue"
        Resource = aws_secretsmanager_secret.database.arn
      }
    ]
  })
}
```

### Workflow: IaC Security Scanning with Checkov

```yaml
# GitHub Actions — IaC Security Scanning

name: Security Scan
on:
  pull_request:
    paths:
      - 'infra/**'
      - 'terraform/**'
      - 'cloudformation/**'

jobs:
  checkov:
    name: Checkov IaC Security Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Checkov
        id: checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: ./infra
          framework: cloudformation,terraform
          output_format: sarif
          output_file_path: results.sarif

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: results.sarif

  trivy:
    name: Container Image Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t app:${{ github.sha }} .

      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: app:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: trivy-results.sarif
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [AWS Inspector User Guide](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_dg.html)
- [AWS WAF Developer Guide](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)
- [AWS Macie User Guide](https://docs.aws.amazon.com/macie/latest/userguide/what-is-macie.html)

### 🏗️ Well-Architected Framework

- [Security Pillar — Shift-Left Security](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shift-left-security.html)
- [Security Pillar — Data Protection](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/data-protection.html)

### 🛠️ Hands-On Tutorials

- [Secrets Manager Rotation Tutorial](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotate-secrets.html)
- [Inspector Getting Started](https://docs.aws.amazon.com/inspector/latest/userguide/inspector-getting-started.html)
- [WAF Rule Groups](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html)
- [Macie Sensitive Data Discovery](https://docs.aws.amazon.com/macie/latest/userguide/find-sensitive-data.html)

### 📄 Whitepapers

- [AWS IAM Best Practices](https://d1.awsstatic.com/whitepapers/aws-iam-best-practices.pdf)
- [AWS Security Best Practices](https://d1.awsstatic.com/whitepapers/aws-security-best-practices.pdf)

---

## 🏋️ Practical Exercises

### Exercise 1: Secrets Management
1. Create a secret in Secrets Manager for a database connection
2. Set up automatic rotation (every 30 days)
3. Deploy an EC2 instance that reads the secret
4. Test that the application can connect with the rotated secret
5. Verify the secret version history in the console

### Exercise 2: Shift-Left Pipeline
1. Add pre-commit hooks (gitleaks, checkov) to a repo
2. Create a CI pipeline that runs CodeGuru and Trivy
3. Block deployments if critical vulnerabilities are found
4. Configure SARIF results to show in GitHub code scanning
5. Add a "security gate" stage in your CodePipeline

### Exercise 3: AWS WAF + CloudFront
1. Deploy a static site with CloudFront
2. Create a WAF web ACL with rate-based rules
3. Add AWS Managed Rules (SQL injection, XSS)
4. Deploy the WAF to CloudFront
5. Test with simulated attack traffic (sqlmap or custom scripts)

### Exercise 4: Infrastructure Security Scanning
1. Write Terraform with intentional misconfigurations (public S3, no encryption)
2. Run Checkov — verify it catches all issues
3. Fix the misconfigurations
4. Run Checkov again — verify clean
5. Integrate Checkov into CI/CD as a blocking step

---

## 💡 Pro Tips

- **Secrets Manager > environment variables** — Rotation, audit trail, fine-grained access
- **Scan at every stage** — Pre-commit, CI, build, deploy, runtime
- **Checkov for IaC, Trivy for containers, Semgrep for code** — Use the right tool
- **WAF is cheap protection** — Always put WAF in front of public-facing applications
- **Inspector + GuardDuty = comprehensive coverage** — Host-level + network-level security
- **Compliance as code, not as afterthought** — Config rules, security hub, audit manager

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Configured Secrets Manager with rotation
- [ ] Built shift-left security pipeline
- [ ] Set up Inspector for vulnerability scanning
- [ ] Configured WAF for web applications
- [ ] Implemented IaC security scanning (Checkov)
- [ ] Set up Macie for S3 data protection
- [ ] Created incident response runbooks
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
