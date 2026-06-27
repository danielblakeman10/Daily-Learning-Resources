# 📊 Pillar 10 — Observability & Monitoring

> **CloudWatch, X-Ray, OpenSearch — full-stack observability with metrics, logs, and traces.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Design centralized logging with CloudWatch Logs and Log Groups
- [ ] Create CloudWatch dashboards for application and infrastructure monitoring
- [ ] Implement CloudWatch Alarms with SNS notifications and auto-remediation
- [ ] Use X-Ray for distributed tracing across microservices
- [ ] Set up CloudTrail for API audit trails
- [ ] Deploy OpenSearch Service for log aggregation and search
- [ ] Implement EventBridge rules for event-driven alerting
- [ ] Create DevOps Guru for ML-based anomaly detection
- [ ] Design runbooks and playbooks for common incidents

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **CloudWatch Logs** | Centralized log storage | Application logs, audit trails |
| **CloudWatch Metrics** | Time-series monitoring | CPU, memory, custom metrics |
| **CloudWatch Dashboards** | Visual monitoring | Single pane of glass |
| **CloudWatch Alarms** | Threshold-based alerting | Auto-scaling, notifications |
| **AWS X-Ray** | Distributed tracing | Microservice debugging, latency analysis |
| **CloudTrail** | API activity logging | Security audit, compliance |
| **Amazon OpenSearch** | Log analytics and search | Full-text log search, visualization |
| **DevOps Guru** | ML-based anomaly detection | Proactive issue detection |
| **EventBridge** | Event-driven alerting | Route alerts to appropriate handlers |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Centralized Logging Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│  Observability Architecture                                 │
│                                                             │
│  Sources:  CloudWatch Logs ← EC2 / ECS / EKS               │
│            CloudTrail  ← API calls across accounts           │
│            X-Ray     ← Microservice traces                  │
│            Metrics   ← Custom application metrics            │
│                                                             │
│  Processing:                                                 │
│  ├── CloudWatch Logs → OpenSearch (searchable)              │
│  ├── CloudWatch Metrics → Dashboards                        │
│  ├── CloudWatch Alarms → EventBridge → SNS → PagerDuty     │
│  └── X-Ray → Service Map → Latency Analysis                │
│                                                             │
│  Response:                                                   │
│  ├── CloudWatch Alarms → Auto-scaling actions               │
│  ├── EventBridge → SSM Automation (auto-remediation)       │
│  └── SNS → Slack / Email / PagerDuty                       │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: CloudWatch Dashboard + Alarms

```yaml
# CloudFormation: CloudWatch Dashboard

Resources:
  ApplicationDashboard:
    Type: AWS::CloudWatch::Dashboard
    Properties:
      DashboardName: !Sub "${Environment}-application-dashboard"
      DashboardBody: !Sub |
        {
          "widgets": [
            {
              "type": "metric",
              "properties": {
                "title": "Request Count",
                "metrics": [
                  ["AWS/ApplicationELB", "RequestCount", "LoadBalancer", "${ALBFullName}"]
                ],
                "period": 300,
                "stat": "Sum"
              }
            },
            {
              "type": "metric",
              "properties": {
                "title": "5XX Errors",
                "metrics": [
                  ["AWS/ApplicationELB", "HTTPCode_ELB_5XX_Count", "LoadBalancer", "${ALBFullName}"]
                ],
                "period": 300,
                "stat": "Sum"
              }
            },
            {
              "type": "metric",
              "properties": {
                "title": "Target Response Time",
                "metrics": [
                  ["AWS/ApplicationELB", "TargetResponseTime", "LoadBalancer", "${ALBFullName}"]
                ],
                "period": 300,
                "stat": "p95"
              }
            },
            {
              "type": "metric",
              "properties": {
                "title": "Active Connections",
                "metrics": [
                  ["AWS/ApplicationELB", "ActiveConnectionCount", "LoadBalancer", "${ALBFullName}"]
                ],
                "period": 300,
                "stat": "Average"
              }
            }
          ]
        }

  HighErrorRateAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Sub "${Environment}-high-error-rate"
      AlarmDescription: "5XX errors exceeded threshold"
      MetricName: HTTPCode_ELB_5XX_Count
      Namespace: AWS/ApplicationELB
      Statistic: Sum
      Period: 300
      EvaluationPeriods: 2
      Threshold: 10
      ComparisonOperator: GreaterThanThreshold
      AlarmActions:
        - !Ref ErrorNotificationTopic
      Dimensions:
        - Name: LoadBalancer
          Value: !Ref ALBFullName
```

### Workflow: Distributed Tracing with X-Ray

```python
# Instrumented Lambda with X-Ray
import boto3
import requests
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patchall

# Patch all supported libraries
patchall()

@xray_recorder.capture('process_order')
def lambda_handler(event, context):
    xray_recorder.put_metadata('request_id', event.get('requestId'))

    # Subsegment for S3 operation
    with xray_recorder.in_subsegment('query_inventory'):
        s3 = boto3.resource('s3')
        bucket = s3.Bucket('inventory-bucket')
        inventory = bucket.Object('current-stock.json').get()['Body'].read()

    # Subsegment for DynamoDB
    with xray_recorder.in_subsegment('update_order'):
        dynamodb = boto3.resource('dynamodb')
        table = dynamodb.Table('orders')
        table.put_item(Item={
            'orderId': event['orderId'],
            'status': 'processing',
            'timestamp': event['timestamp']
        })

    # Subsegment for HTTP call
    with xray_recorder.in_subsegment('notify_partner'):
        response = requests.post(
            'https://partner-api.com/orders',
            json={'orderId': event['orderId']},
            timeout=5
        )

    return {'statusCode': 200, 'body': 'Order processed'}
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_FreeTier.html)
- [CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
- [AWS X-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/xray-service-map.html)
- [CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [Amazon OpenSearch Service Documentation](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/what-is-amazon-opensearch.html)

### 🏗️ Well-Architected Framework

- [Operations Pillar — Monitoring](https://docs.aws.amazon.com/wellarchitected/latest/operations-pillar/monitoring.html)
- [Operations Pillar — Alerting](https://docs.aws.amazon.com/wellarchitected/latest/operations-pillar/alerting.html)

### 🛠️ Hands-On Tutorials

- [CloudWatch Logs Insights Tutorial](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [X-Ray Getting Started](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python.html)
- [CloudTrail Trail Setup](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-a-trail.html)
- [OpenSearch Service Quick Start](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/qs-getting-started.html)

### 📄 Whitepapers

- [Implementing CloudWatch Monitoring](https://docs.aws.amazon.com/whitepapers/latest/cloudwatch-monitoring-guide/welcome.html)

---

## 🏋️ Practical Exercises

### Exercise 1: CloudWatch Dashboard + Alarms
1. Create a CloudWatch dashboard with 5+ widgets (metrics, text, logs query)
2. Set up alarms for: CPU > 80%, Error Rate > 5%, Latency > 2s
3. Create an SNS topic for alarm notifications
4. Configure auto-remediation with SSM Automation
5. Test by generating load and verifying alarms fire

### Exercise 2: Centralized Logging with OpenSearch
1. Deploy an OpenSearch domain
2. Configure CloudWatch Logs subscription filter
3. Create an OpenSearch index pattern and visualizations
4. Build a log search query for ERROR and WARN events
5. Set up a dashboard with top error patterns

### Exercise 3: Distributed Tracing
1. Create 3 Lambda functions that call each other
2. Instrument all functions with X-Ray SDK
3. Deploy an ALB → API Gateway → Lambda → DynamoDB stack
4. Generate sample traffic and view the service map in X-Ray
5. Identify the slowest service and optimize it

### Exercise 4: EventBridge Alerting Pipeline
1. Create EventBridge rules for: CloudWatch Alarms, CodePipeline failures, Cost anomalies
2. Route critical alerts to SNS → Slack
3. Route informational alerts to Email
4. Create an auto-remediation SSM document (e.g., restart unhealthy EC2)
5. Simulate failures and verify the end-to-end pipeline

---

## 💡 Pro Tips

- **Structured logging > plain text** — JSON logs are easier to query in CloudWatch
- **Set log retention policies** — 30 days for most, 90 days for compliance
- **X-Ray sampling is automatic** — Don't sample everything; let X-Ray handle it
- **CloudWatch Metrics + Alarms → EventBridge → SNS** — This is your alerting backbone
- **Dashboards are as code** — CloudFormation/CDK for dashboards, not console UI
- **OpenSearch for deep log analysis, CloudWatch for quick metrics** — Use the right tool

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Built CloudWatch dashboard
- [ ] Configured CloudWatch Alarms with SNS
- [ ] Set up X-Ray distributed tracing
- [ ] Deployed OpenSearch for log analytics
- [ ] Configured CloudTrail with log validation
- [ ] Implemented EventBridge alerting pipeline
- [ ] Created DevOps Guru anomaly detection
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
