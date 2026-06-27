# 🔧 Pillar 08 — DevOps & Automation

> **CI/CD pipelines, GitOps, and release strategies — automating the software delivery lifecycle end-to-end.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Design end-to-end CI/CD pipelines with AWS CodePipeline + CodeBuild
- [ ] Implement GitOps workflows with AWS CodeCommit, GitHub, and CodePipeline
- [ ] Use AWS CodeDeploy for blue/green and in-place deployments
- [ ] Configure automated testing stages in CI/CD pipelines
- [ ] Implement infrastructure changes with automated CI/CD
- [ ] Use AWS Systems Manager for configuration management and patching
- [ ] Implement change management with AWS CloudCenter/Change Manager
- [ ] Use AWS Artifact for compliance documentation and reporting
- [ ] Design rollback strategies for failed deployments
- [ ] Implement canary deployments with CodeDeploy

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **AWS CodePipeline** | CI/CD orchestration | End-to-end pipeline orchestration |
| **AWS CodeBuild** | Build service | Compile, test, package code |
| **AWS CodeDeploy** | Deployment automation | EC2, Lambda, ECS deployments |
| **AWS CodeCommit** | Source control | Git-based code repository |
| **AWS CodeArtifact** | Package management | Store and share build artifacts |
| **AWS CodeGuru** | Code quality & reviews | AI-powered code suggestions |
| **AWS Systems Manager** | Configuration & patching | EC2 management without SSH |
| **AWS DevOps Guru** | Anomaly detection | ML-based ops insights |
| **AWS CloudCenter** | Application lifecycle management | Multi-cloud application deployment |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: End-to-End CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│  CI/CD Pipeline Architecture                                        │
│                                                                     │
│  Source: CodeCommit / GitHub ──→ Build: CodeBuild                  │
│       │                               │                             │
│       │                    ┌──────────┴──────────┐                  │
│       │                    │  Unit Tests          │                  │
│       │                    │  Integration Tests   │                  │
│       │                    │  Security Scan       │                  │
│       │                    │  CodeGuru Analysis   │                  │
│       │                    └──────────┬──────────┘                  │
│       │                               │                             │
│       └───────────────────────────────┼─────────────────────────────┤
│                   Deploy              │                             │
│                   ┌───────────────────┼───────────────────┐         │
│                   │                   │                   │         │
│                   ▼                   ▼                   ▼         │
│             CodeDeploy          ECS/Fargate         Lambda       │
│             (EC2 Blue/Green)    (Rolling Update)    (Canary)     │
│                   │                   │                   │         │
│                   └───────────────────┼───────────────────┘         │
│                                     │                              │
│                              Validation:                          │
│                              • Health checks                      │
│                              • Smoke tests                        │
│                              • Synthetic monitoring               │
│                              • Automated rollback on failure      │
└─────────────────────────────────────────────────────────────────────┘
```

### Workflow: GitOps with CodePipeline

```yaml
# AWS CodePipeline — CodeCommit → CodeBuild → CodeDeploy

# stages:
#   1. Source (CodeCommit)
#   2. Build (CodeBuild — test + package)
#   3. Deploy (CodeDeploy — blue/green)

# buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm ci

  pre_build:
    commands:
      - echo "Running security scan..."
      - npm audit
      - echo "Building Docker image..."

  build:
    commands:
      - npm test
      - npm run build
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $REPOSITORY_URI
      - docker build -t $IMAGE_NAME:$CODEBUILD_RESOLVED_SOURCE_VERSION .
      - docker tag $IMAGE_NAME:$CODEBUILD_RESOLVED_SOURCE_VERSION $REPOSITORY_URI:$CODEBUILD_RESOLVED_SOURCE_VERSION
      - docker push $REPOSITORY_URI:$CODEBUILD_RESOLVED_SOURCE_VERSION

  post_build:
    commands:
      - echo "Build completed on $(date)"
      - printf '[{"name":"app","imageUri":"%s"}]' $REPOSITORY_URI:$CODEBUILD_RESOLVED_SOURCE_VERSION > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
    - appspec.yml
```

### Workflow: Blue/Green Deployment with CodeDeploy

```yaml
# appspec.yml for EC2 Blue/Green

version: 0.0
os: linux
files:
  - source: /
    destination: /opt/app
hooks:
  BeforeInstall:
    - Location: scripts/before_install.sh
      Timeout: 300
      RunAs: root
  ApplicationStop:
    - Location: scripts/application_stop.sh
      Timeout: 300
      RunAs: root
  AfterInstall:
    - Location: scripts/after_install.sh
      Timeout: 300
      RunAs: root
  ApplicationStart:
    - Location: scripts/application_start.sh
      Timeout: 300
      RunAs: root
  ValidateService:
    - Location: scripts/validate_service.sh
      Timeout: 300
```

### Workflow: AWS Systems Manager Patching

```hcl
# Terraform: SSM Automation — Automated EC2 Patching

resource "aws_ssm_document" "ec2_patch" {
  name          = "EC2-Patch-Day"
  document_type = "Automation"
  document_format = "YAML"

  content = yamlencode({
    schemaVersion = "0.3"
    description = "Automated EC2 patching with SSM"
    mainSteps = [
      {
        name = "FindUnpatchedInstances"
        action = "aws:runInstances"
        inputs = {
          InstanceType = "t3.micro"
          ImageId      = "ami-latest-amazon-linux-2"
        }
      }
      {
        name = "PatchInstances"
        action = "aws:executeAutomation"
        inputs = {
          DocumentName = "AWS-RunPatchBaseline"
          Targets = [{
            Key    = "InstanceIds"
            Values = ["{{ FindUnpatchedInstances.Output.InstanceIds }}"]
          }]
        }
      }
      {
        name = "RebootIfRequired"
        action = "aws:changeInstanceState"
        inputs = {
          InstanceIds    = ["{{ PatchInstances.InstanceIds }}"]
          DesiredState   = "reboot"
        }
      }
    ]
  })
}
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [AWS CodeBuild User Guide](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [AWS CodeDeploy User Guide](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
- [AWS Systems Manager User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)

### 🏗️ Well-Architected Framework

- [Operations Pillar — Automation](https://docs.aws.amazon.com/wellarchitected/latest/operations-pillar/automate-change-management.html)
- [Release Management Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/operations-pillar/release-management.html)

### 🛠️ Hands-On Tutorials

- [AWS DevOps Tutorial](https://docs.aws.amazon.com/dtcentral/latest/userguide/dtcentral-tutorial-1-get-started.html)
- [CodeDeploy Blue/Green Tutorial](https://docs.aws.amazon.com/codedeploy/latest/userguide/BLUEGREEN_DEPLOYMENTS.html)
- [CodeBuild with buildspec](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)
- [SSM Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html)

### 📄 Whitepapers

- [Continuous Integration and Delivery on AWS](https://docs.aws.amazon.com/whitepapers/latest/continuous-integration-delivery-aws-devops/ci-cd.html)

---

## 🏋️ Practical Exercises

### Exercise 1: Build a CI/CD Pipeline
1. Set up a CodeCommit repository
2. Create a CodePipeline that triggers on git push
3. Configure CodeBuild to run tests and build
4. Deploy to an EC2 instance using CodeDeploy
5. Implement automatic rollback on failure

### Exercise 2: Blue/Green Deployment
1. Set up a CodeDeploy application with 2 Auto Scaling groups (blue/green)
2. Create an appspec.yml for EC2 deployment
3. Deploy an application with blue/green strategy
4. Simulate a failure — verify automatic rollback
5. Implement traffic shifting with a 10/90 → 50/50 → 100/0 gradual shift

### Exercise 3: SSM Patch Management
1. Install SSM Agent on EC2 instances
2. Configure patch baselines (approved OS updates)
2. Run a maintenance window to patch instances
3. Verify patch compliance via AWS Systems Manager
4. Set up CloudWatch Alarms for failed patches

### Exercise 4: GitHub Actions → AWS CI/CD
1. Create a GitHub Actions workflow
2. Configure OIDC authentication to AWS
3. Run tests and build in GitHub Actions
4. Deploy to AWS Lambda via AWS CLI
5. Add Slack notifications for deploy status

---

## 💡 Pro Tips

- **Test everything in CI** — Unit, integration, security, and linting stages
- **Blue/Green > Rolling for zero-downtime** — Full swap, instant rollback
- **CodeDeploy supports Lambda, EC2, and ECS** — Use the right deployment type
- **SSM replaces SSH entirely** — Session Manager, Patch Manager, Run Command
- **Immutable deployments > in-place** — New server, new AMI, swap
- **Pipeline is your source of truth** — Everything goes through the pipeline, no manual deploys

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Built end-to-end CI/CD pipeline
- [ ] Implemented blue/green deployment
- [ ] Configured CodeDeploy with appspec
- [ ] Set up automated testing in CI
- [ ] Configured SSM for patch management
- [ ] Implemented rollback strategies
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
