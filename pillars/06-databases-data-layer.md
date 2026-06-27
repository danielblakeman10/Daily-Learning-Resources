# 🗄️ Pillar 06 — Databases & Data Layer

> **From relational to NoSQL, caching to data lakes — architecting the right data layer for every workload.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Choose between RDS, Aurora, DynamoDB, ElastiCache, and other data services
- [ ] Deploy multi-AZ RDS with automated backups and read replicas
- [ ] Configure Aurora with multi-master clusters and global databases
- [ ] Design DynamoDB tables with partition keys, sort keys, and GSI/LSI indexes
- [ ] Set up ElastiCache (Redis/Memcached) for caching strategies
- [ ] Use AWS DMS for database migration between engines
- [ ] Implement database automation with Lambda and event-driven patterns
- [ ] Plan data lake architectures with S3 + Athena + Glue

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **Amazon RDS** | Managed relational databases | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server |
| **Amazon Aurora** | High-performance relational | MySQL/PostgreSQL-compatible, up to 5x throughput |
| **Amazon DynamoDB** | NoSQL key-value/document | Serverless, single-digit millisecond latency |
| **Amazon ElastiCache** | In-memory caching | Redis, Memcached — reduce database load |
| **Amazon DocumentDB** | NoSQL document database | MongoDB-compatible |
| **Amazon Neptune** | Graph database | Social networks, recommendation engines |
| **AWS Database Migration Service** | Migrate between engines | Zero-downtime migration to AWS |
| **Amazon Athena** | Query S3 with SQL | Serverless data lake analytics |
| **AWS Glue** | ETL and data catalog | Data preparation, schema discovery |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Database Migration with DMS

```
┌─────────────────────────────────────────────────────────────┐
│  DMS Migration Architecture                                 │
│                                                             │
│  Source Database (On-prem RDS PostgreSQL)                  │
│    │                                                        │
│    ├── Replication Instance (DMS)                          │
│    │   ├── Full Load: Migrate existing data                │
│    │   ├── CDC (Change Data Capture): Stream ongoing changes│
│    │   └── Validate: Verify data integrity                 │
│    │                                                        │
│    └── Target: Aurora PostgreSQL (Multi-AZ)                 │
│                                                             │
│  Phase 1: Schema Migration (Schema Conversion Tool)        │
│  Phase 2: Full Load Migration                              │
│  Phase 3: CDC (Online migration, zero downtime)             │
│  Phase 4: Cutover (Update application connection strings)  │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: DynamoDB Design for Scale

```hcl
# Terraform: DynamoDB Table with GSI

resource "aws_dynamodb_table" "orders" {
  name         = "orders-table"
  billing_mode = "PAY_PER_REQUEST" # Serverless mode

  hash_key   = "user_id"
  range_key  = "order_date"

  attribute {
    name = "user_id"
    type = "S"
  }
  attribute {
    name = "order_date"
    type = "S"
  }
  attribute {
    name = "status"
    type = "S"
  }
  attribute {
    name = "created_at"
    type = "N"
  }

  # Global Secondary Index: query by status + created_at
  global_secondary_index {
    name            = "status_created_at_index"
    hash_key        = "status"
    range_key       = "created_at"
    projection_type = "ALL"
    read_capacity   = 5
    write_capacity  = 5
  }

  point_in_time_recovery {
    enabled = true
  }

  server_side_encryption {
    enabled     = true
    kms_key_arn = aws_kms_key.dynamodb.arn
  }

  deletion_protection_enabled = true

  tags = {
    Name        = "orders-table"
    Environment = "production"
  }
}
```

### Workflow: Aurora Serverless v2 Deployment

```yaml
# CloudFormation: Aurora PostgreSQL Serverless

Resources:
  AuroraCluster:
    Type: AWS::RDS::DBCluster
    Properties:
      Engine: aurora-postgresql
      EngineVersion: "15.4"
      DBClusterIdentifier: my-aurora-cluster
      StorageEncrypted: true
      KmsKeyId: !Ref AuroraKMSKey
      BackupRetentionPeriod: 35
      DeletionProtection: true

      ServerlessV2ScalingConfiguration:
        MinCapacity: 0.5
        MaxCapacity: 10.0

      VpcSecurityGroupIds:
        - !Ref AuroraSecurityGroup
      DBSubnetGroupName: !Ref DBSubnetGroup

      # Enable enhanced monitoring
      MonitoringRoleArn: !GetAtt RDSMonitoringRole.Arn
      MonitoringInterval: 60

      # Performance insights
      EnablePerformanceInsights: true
      PerformanceInsightsKMSKeyId: !Ref PIKMSKey

  AuroraReader:
    Type: AWS::RDS::DBInstance
    DependsOn: AuroraCluster
    Properties:
      DBClusterIdentifier: !Ref AuroraCluster
      DBInstanceClass: db.serverless
      Engine: aurora-postgresql
      DBInstanceIdentifier: my-aurora-read-replica
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Amazon Aurora User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)
- [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [ElastiCache User Guide](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)
- [AWS DMS User Guide](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.html)

### 🏗️ Well-Architected Framework

- [Database Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/data-pillar/welcome.html)
- [NoSQL Design Patterns](https://docs.aws.amazon.com/wellarchitected/latest/data-pillar/choose-the-right-database.html)

### 🛠️ Hands-On Tutorials

- [Aurora Getting Started](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Tutorials.html)
- [DynamoDB Local & CLI Workshop](https://catalog.workshops.aws/dynamodb)
- [AWS DMS Migration Walkthrough](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tutorials.html)
- [Athena Query S3 Data](https://docs.aws.amazon.com/athena/latest/ug/querying.html)

### 📄 Whitepapers

- [Building Serverless Architectures on AWS](https://aws.amazon.com/whitepapers/building-serverless-architectures-on-aws/)
- [Database Migration Strategies on AWS](https://d1.awsstatic.com/whitepapers/aws-database-migration-whitepaper.pdf)

---

## 🏋️ Practical Exercises

### Exercise 1: RDS Multi-AZ Deployment
1. Deploy an RDS PostgreSQL instance in Multi-AZ
2. Enable automated backups with a 7-day retention
3. Create a read replica in another AZ
4. Test failover by simulating a primary failure
5. Configure CloudWatch alarms for CPU, free storage, connections

### Exercise 2: Aurora Global Database
1. Deploy an Aurora PostgreSQL cluster in us-east-1
2. Add a secondary cluster in us-west-2 (Global Database)
3. Write test data in the primary — verify replication latency
4. Simulate a primary region failure
5. Promote the secondary cluster

### Exercise 3: DynamoDB Table Design
1. Design a DynamoDB table for an e-commerce orders system
2. Choose partition keys (user_id) and sort keys (order_date)
3. Create GSIs for: "get orders by status", "get orders by date range"
4. Use DynamoDB Local for local development
5. Implement a DynamoDB Stream for order processing

### Exercise 4: Database Migration with DMS
1. Set up an on-premises PostgreSQL (or EC2 PostgreSQL)
2. Create a DMS replication instance
3. Configure source and target endpoints
4. Run a full load migration to Aurora
5. Enable CDC, verify ongoing replication
6. Perform the cutover with minimal downtime

---

## 💡 Pro Tips

- **Aurora is worth the premium** — 5x RDS throughput, multi-master, instant failover
- **Design DynamoDB around access patterns** — Not schema, not joins. Access patterns dictate key design
- **Use read replicas for reads, not DR** — Replica lag can cause stale reads
- **Cache, don't query** — ElastiCache for frequently-read, rarely-changed data
- **DMS CDC is your friend** — Online migration with zero downtime for production databases
- **Athena + Parquet = cheap analytics** — Transform S3 CSV to Parquet, query with SQL

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Deployed RDS Multi-AZ
- [ ] Configured Aurora with read replicas
- [ ] Designed and deployed DynamoDB tables
- [ ] Set up ElastiCache (Redis)
- [ ] Performed database migration with DMS
- [ ] Created Athena data lake queries
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
