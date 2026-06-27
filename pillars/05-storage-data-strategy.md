# 💾 Pillar 05 — Storage & Data Strategy

> **From S3 to EBS to FSx — right storage class for every workload and data lifecycle management.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Design S3 bucket policies with lifecycle rules and versioning
- [ ] Implement S3 Intelligent-Tiering and storage class transitions
- [ ] Configure S3 Cross-Region Replication (CRR) and MFA Delete
- [ ] Use EBS for block storage with snapshots, snapshots encryption, and performance tuning
- [ ] Deploy EFS for shared file systems across multiple EC2/EKS instances
- [ ] Set up FSx for Windows/Linux/NetApp ONTAP
- [ ] Implement AWS Backup for centralized backup management
- [ ] Design data lifecycle strategies using S3 Lifecycle and Glacier Deep Archive

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **S3** | Object storage at any scale | Data lake, static hosting, artifact storage |
| **EBS** | Block storage for EC2 | Persistent volumes, snapshots, encryption |
| **EFS** | Managed NFS file system | Shared storage for EC2, ECS, EKS |
| **FSx** | Managed Windows/NetApp/Lustre | Domain-joined, high-performance file storage |
| **S3 Glacier** | Archive storage (minutes-to-hours retrieval) | Compliance, long-term retention |
| **AWS Backup** | Centralized backup service | Automated backups across AWS services |
| **Storage Gateway** | Hybrid storage connecting on-prem to cloud | On-premises to cloud data replication |
| **Data Lifecycle Manager** | Automated EBS snapshot management | Compliance-driven snapshot schedules |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: S3 Storage Strategy

```
┌─────────────────────────────────────────────────────────────┐
│  S3 Storage Class Decision Tree                             │
│                                                             │
│  Access Frequency → Storage Class                          │
│  ────────────────── → ─────────────────                    │
│  Frequently accessed  → S3 Standard                        │
│  Infrequent access    → S3 Standard-IA                     │
│  Archive (hourly)     → S3 Glacier Instant                 │
│  Archive (minutes)    → S3 Glacier Flexible                │
│  Archive (12 hours+)  → S3 Glacier Deep Archive            │
│  Auto-optimize        → S3 Intelligent-Tiering             │
│                                                             │
│  Lifecycle Rule Example:                                    │
│  Day 0:    S3 Standard                                     │
│  Day 30:    → S3 Standard-IA                               │
│  Day 90:    → S3 Glacier                                   │
│  Day 365:   → S3 Glacier Deep Archive                      │
│  Day 2555:  → Delete                                     │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: EBS Snapshot Automation with Data Lifecycle Manager

```hcl
# Terraform: EBS Volume with Automated Snapshots

resource "aws_ebs_volume" "app_data" {
  availability_zone = "us-east-1a"
  size              = 100
  type              = "gp3"
  encrypted         = true
  kms_key_id        = aws_kms_key.ebs.arn

  tags = {
    Name        = "app-data-volume"
    Environment = "production"
    BackupPolicy = "daily"
  }
}

# AWS Data Lifecycle Manager policy
resource "aws_dlm_lifecycle_policy" "ebs_snapshots" {
  description        = "Daily EBS snapshots for production"
  execution_role_arn = aws_iam_role.dlm.arn
  state              = "ENABLED"

  policy_details {
    resource_types = ["VOLUME"]
    target_tags = {
      BackupPolicy = "daily"
    }
    schedule {
      name = "daily-snapshots"
      create_rule {
        interval      = 24
        interval_unit = "HOURS"
        times         = ["02:00"]
      }
      retain_rule {
        count = 30  # Keep 30 days of snapshots
      }
      copy_tags = true
      fast_backup_enabled = true
    }
  }
}
```

### Workflow: S3 Cross-Region Replication

```yaml
# CloudFormation: S3 CRR Configuration

Resources:
  SourceBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-source-bucket
      VersioningConfiguration:
        Status: Enabled
      LifecycleConfiguration:
        Rules:
          - Id: TransitionToGlacier
            Status: Enabled
            Transitions:
              - StorageClass: GLACIER
                TransitionInDays: 90
          - Id: ExpireCurrentVersions
            Status: Enabled
            NoncurrentVersionExpirationInDays: 365

  DestinationBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-replica-bucket
      VersioningConfiguration:
        Status: Enabled

  ReplicationRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: s3.amazonaws.com
            Action: sts:AssumeRole

  BucketReplication:
    Type: AWS::S3::Bucket
    DependsOn: ReplicationRole
    Properties:
      BucketName: my-source-bucket
      ReplicationConfiguration:
        Role: !GetAtt ReplicationRole.Arn
        Rules:
          - Id: ReplicateToWest2
            Status: Enabled
            Destination:
              Bucket: !GetAtt DestinationBucket.Arn
              StorageClass: STANDARD
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [S3 Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [EBS User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html)
- [EFS User Guide](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
- [AWS Backup User Guide](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)

### 🏗️ Well-Architected Framework

- [Storage Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/storage-pillar/welcome.html)
- [Cost Optimization — Right-size Storage](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/optimize-cost-storage.html)

### 🛠️ Hands-On Tutorials

- [S3 Workshop](https://s3workshop.awscloud.com/)
- [EFS Getting Started](https://docs.aws.amazon.com/efs/latest/ug/creating-using-create-efs.html)
- [AWS Backup Setup](https://docs.aws.amazon.com/aws-backup/latest/devguide/creating-a-backup-plan.html)
- [S3 Replication Walkthrough](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

### 📄 Whitepapers

- [Data Storage Options on AWS](https://docs.aws.amazon.com/whitepapers/latest/choosing-a-storage-option/storage-options.html)
- [Cost Optimization on AWS](https://d1.awsstatic.com/whitepapers/aws-cost-optimization.pdf)

---

## 🏋️ Practical Exercises

### Exercise 1: S3 Lifecycle & Replication
1. Create two S3 buckets in different regions
2. Enable versioning and MFA Delete on the source bucket
3. Configure Cross-Region Replication
4. Upload objects and verify replication
5. Set up lifecycle rules: Standard → IA → Glacier → Delete
6. Test the transition by waiting (or using S3 Select to verify storage class)

### Exercise 2: EBS Snapshot Strategy
1. Create a gp3 EBS volume and attach it to an EC2 instance
2. Write test data to the volume
3. Set up a DLM lifecycle policy for daily snapshots
4. Create a manual snapshot and verify retention
5. Restore from snapshot to a new volume
6. Test cross-region snapshot copy

### Exercise 3: EFS for Shared Storage
1. Deploy an EFS file system in multi-AZ mode
2. Mount it on 2 EC2 instances using ECS (EFS CSI Driver)
3. Write test files from one instance, read from another
4. Set up backup with AWS Backup
5. Test elasticity by adding a 3rd instance in another AZ

### Exercise 4: Complete Storage Architecture
1. Design an architecture with: S3 (primary), EBS (boot + data), EFS (shared configs), FSx (Windows file shares)
2. Implement lifecycle policies for S3
3. Set up encrypted EBS volumes with KMS
4. Configure EFS backups
5. Write a backup runbook for each storage type

---

## 💡 Pro Tips

- **S3 is infinitely scalable** — Start with S3 for everything, move to EBS/EFS when you need file semantics
- **Encrypt everything at rest** — Use SSE-S3 for default, SSE-KMS for fine-grained key control
- **S3 Intelligent-Tiering pays for itself** — If you can't predict access patterns, let it optimize
- **EFS is NFS, not block storage** — Use for shared state, configs, home directories — not databases
- **Snapshots are incremental** — First full, subsequent only changes. Fast and cost-effective
- **Use Storage Gateway for legacy apps** — Migrate on-prem SMB/NFS to cloud without rewriting

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Configured S3 lifecycle rules and transitions
- [ ] Set up S3 Cross-Region Replication
- [ ] Created EBS volumes with encryption
- [ ] Deployed EFS for shared storage
- [ ] Configured FSx for Windows/Linux
- [ ] Set up AWS Backup for storage services
- [ ] Implemented data lifecycle strategies
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
