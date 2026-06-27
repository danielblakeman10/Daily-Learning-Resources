# 🚀 Pillar 12 — Migration & Cost Optimization

> **Modernize workloads, minimize costs — strategic migration planning and FinOps practices for cloud success.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Design migration strategies using the 6 Rs (Rehost, Replatform, Refactor, Repurchase, Retire, Retain)
- [ ] Use AWS DMS for database migration with zero downtime
- [ ] Implement AWS Migration Hub for tracking migration progress
- [ ] Use AWS Application Migration Service (MGN) for lift-and-shift
- [ ] Set up AWS Cost Explorer for spend analysis
- [ ] Implement FinOps practices with Cost Allocation Tags, Budgets, and Savings Plans
- [ ] Use AWS Trusted Advisor for cost optimization recommendations
- [ ] Design Reserved Instances and Savings Plans strategies
- [ ] Implement Spot Instance strategies for cost optimization
- [ ] Create cost allocation reports and chargeback models

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **AWS Migration Hub** | Centralized migration tracking | Single pane for all migrations |
| **AWS Application Migration Service** | Lift-and-shift migration | Minimal downtime server migration |
| **AWS DMS** | Database migration | Zero-downtime DB migration |
| **AWS Schema Conversion Tool** | Schema migration | PostgreSQL → Aurora, SQL Server → Aurora |
| **AWS Cost Explorer** | Spend analysis | Trend analysis, forecasting |
| **AWS Budgets** | Cost and usage budgets | Alerts, auto-scaling, resource control |
| **Savings Plans** | Committed use discounts | Up to 72% savings vs On-Demand |
| **Trusted Advisor** | Best practices checks | Cost, security, performance checks |
| **AWS RAIs (Resource Access Manager)** | Shared resources | Resource sharing across accounts |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Migration Strategy Decision Tree

```
┌─────────────────────────────────────────────────────────────┐
│  The 6 Rs of Migration                                     │
│                                                             │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Rehost (Lift & Shift)                           │      │
│  │  Move as-is to EC2 or MGN                        │      │
│  │  Speed: FAST | Risk: LOW | Savings: MODERATE     │      │
│  │  Use when: Time-to-market is critical            │      │
│  └──────────────────────────────────────────────────┘      │
│                       ↓                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Replatform (Lift, Tweak, & Shift)               │      │
│  │  Small changes (RDS instead of self-hosted DB)   │      │
│  │  Speed: MEDIUM | Risk: MEDIUM | Savings: HIGH     │      │
│  │  Use when: Quick win with some cloud-native gain  │      │
│  └──────────────────────────────────────────────────┘      │
│                       ↓                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Refactor (Re-architect)                          │      │
│  │  Rewrite for cloud-native (Lambda, ECS, EKS)     │      │
│  │  Speed: SLOW | Risk: HIGH | Savings: MAXIMUM     │      │
│  │  Use when: Long-term value, cloud-native benefits │      │
│  └──────────────────────────────────────────────────┘      │
│                       ↓                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Repurchase (Move to SaaS)                        │      │
│  │  CRM → Salesforce, ERP → SAP on AWS               │      │
│  │  Speed: FAST | Risk: LOW | Savings: VARIABLE     │      │
│  │  Use when: SaaS is clearly better than custom     │      │
│  └──────────────────────────────────────────────────┘      │
│                       ↓                                    │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Retain or Retire                                  │      │
│  │  Keep on-prem or decommission                      │      │
│  │  Speed: N/A | Risk: LOW | Savings: NONE          │      │
│  │  Use when: Regulatory, legacy, or no business case│      │
│  └──────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: FinOps Cost Optimization

```
┌─────────────────────────────────────────────────────────────┐
│  FinOps Implementation                                     │
│                                                             │
│  Phase 1: Visibility                                       │
│  ├── Tag all resources (CostCenter, Owner, Environment)     │
│  ├── Enable Cost Explorer with granularity                  │
│  ├── Set up Cost Allocation Tags                            │
│  └── Create Budgets per team/project                        │
│                                                             │
│  Phase 2: Optimization                                     │
│  ├── Right-size underutilized EC2/RDS                       │
│  ├── Buy Savings Plans (compute, 1yr/3yr)                  │
│  ├── Use Spot Instances where possible                      │
│  ├── Implement S3 lifecycle policies                        │
│  ├── Delete unused EBS volumes and snapshots               │
│  └── Clean up unattached EBS volumes                        │
│                                                             │
│  Phase 3: Forecasting                                      │
│  ├── Set up Cost Anomaly Detection                          │
│  ├── Predictive alerts for spend thresholds                 │
│  ├── Monthly cost review meetings                           │
│  └── Projected spend vs budget                              │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: AWS Application Migration Service (MGN)

```yaml
# CloudFormation: Migration Hub

Resources:
  MigrationHubTrack:
    Type: AWS::MigrationHub::Project
    Properties:
      Name: "production-migration"
      Description: "Production workloads migration to AWS"

  # Migration progress tracking
  MigrationProgress:
    Type: AWS::MigrationHub::MigrationTask
    DependsOn: MigrationHubTrack
    Properties:
      ApplicationId: !Ref ApplicationId
      MigrationTaskName: "web-server-migration"
      ProgressPercent: 50
      UpdateToken: !Ref UpdateToken
```

### Workflow: Savings Plans Implementation

```hcl
# Terraform: Cost Analysis & Savings Plans Strategy

# Step 1: Cost Explorer API for spend analysis
data "aws_cost_by_service" "monthly" {
  time_period {
    start = "2024-01-01"
    end   = "2024-12-31"
  }
  group_by {
    type  = "DIMENSION"
    key   = "SERVICE"
  }
}

# Step 2: Budgets with alerts
resource "aws_budgets_budget" "production" {
  name              = "production-budget"
  budget_type       = "COST"
  limit_amount      = "5000"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"

  notifications {
    comparison_operator       = "GREATER_THAN"
    threshold                 = 80
    threshold_type            = "ABSOLUTE"
    notification_type         = "ACTUAL"
    subscribers {
      subscription_type = "EMAIL"
      address           = "finance@company.com"
    }
  }

  notifications {
    comparison_operator       = "GREATER_THAN"
    threshold                 = 100
    threshold_type            = "ABSOLUTE"
    notification_type         = "ACTUAL"
    subscribers {
      subscription_type = "EMAIL"
      address           = "finance@company.com"
    }
  }
}
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [AWS Migration EBook](https://d1.awsstatic.com/whitepapers/aws-migration-ebook.pdf)
- [AWS Migration Hub User Guide](https://docs.aws.amazon.com/migrationhub/latest/ug/migrationhub.html)
- [AWS DMS User Guide](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tutorials.html)
- [AWS Cost Explorer User Guide](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/cost-explorer-creating.html)
- [AWS Budgets User Guide](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)

### 🏗️ Well-Architected Framework

- [Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
- [Operational Excellence — Cost Management](https://docs.aws.amazon.com/wellarchitected/latest/operations-pillar/cost-management.html)

### 🛠️ Hands-On Tutorials

- [Migration Hub Getting Started](https://docs.aws.amazon.com/migrationhub/latest/ug/migrationhub-getting-started.html)
- [AWS Cost Optimization Workshop](https://catalog.workshops.aws/cost-optimization)
- [Savings Plans Calculator](https://aws.amazon.com/savingsplans/pricing/)
- [Trusted Advisor Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-cost-optimization/best-practices.html)

### 📄 Whitepapers

- [AWS Cost Optimization Best Practices](https://d1.awsstatic.com/whitepapers/aws-cost-optimization.pdf)
- [FinOps Foundation Framework](https://www.finops.org/framework/)

---

## 🏋️ Practical Exercises

### Exercise 1: Migration Assessment
1. Use Migration Evaluator (CloudHealth by VMware) to assess on-prem workload
2. Map each workload to a migration strategy (6 Rs)
3. Create a migration plan with timelines and risk assessment
4. Use Migration Hub to track progress
5. Document the business case for each migration

### Exercise 2: DMS Zero-Downtime Migration
1. Set up a source PostgreSQL on EC2
2. Create a target Aurora PostgreSQL cluster
3. Configure DMS replication instance and endpoints
4. Run full load + CDC migration
5. Perform cutover with minimal downtime

### Exercise 3: FinOps Implementation
1. Tag 20+ resources with CostCenter, Owner, Environment
2. Set up AWS Budgets with 80%/100% alerts
3. Implement Cost Anomaly Detection
4. Analyze Cost Explorer for 3 months of data
5. Calculate potential savings from Savings Plans
6. Create a cost allocation report

### Exercise 4: Cost Optimization Audit
1. Run Trusted Advisor checks (Cost Optimization)
2. Identify underutilized EC2 instances (<20% CPU)
3. Find unattached EBS volumes and snapshots
4. Analyze S3 storage classes for optimization
5. Calculate savings from right-sizing and Savings Plans
6. Create an optimization action plan

---

## 💡 Pro Tips

- **Tag everything from day one** — No tags = no visibility = no control
- **Savings Plans > Reserved Instances** — More flexible, up to 72% savings
- **Trusted Advisor is free** — Use the 7 cost checks every month
- **S3 lifecycle policies pay for themselves** — Auto-transition to cheaper classes
- **FinOps is a culture, not a tool** — Teams need visibility into their own costs
- **Migrate, then optimize** — Don't refactor everything before migrating. Move first, optimize later
- **Budget alerts at 80% and 100%** — Catch surprises before they become incidents

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Designed migration strategy (6 Rs)
- [ ] Set up AWS Migration Hub
- [ ] Performed database migration with DMS
- [ ] Set up Cost Explorer with tagging
- [ ] Created AWS Budgets with alerts
- [ ] Implemented Savings Plans strategy
- [ ] Ran Trusted Advisor cost checks
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
