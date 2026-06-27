# ⚡ Pillar 07 — Serverless & Event-Driven

> **Lambda, EventBridge, SQS — building scalable, decoupled systems with zero server management.**

---

## 🎯 Learning Objectives

By the end of this pillar, you will be able to:

- [ ] Deploy Lambda functions with proper IAM roles, layers, and configuration
- [ ] Design event-driven architectures using EventBridge, SQS, and SNS
- [ ] Implement step functions for complex workflow orchestration
- [ ] Build serverless APIs with API Gateway + Lambda
- [ ] Use Kinesis for real-time data streaming
- [ ] Configure Lambda concurrency limits, provisioned concurrency, and error handling
- [ ] Design fan-out patterns with EventBridge and SNS/SQS
- [ ] Implement CI/CD for serverless applications (SAM/CDK)

---

## 🔑 Key Services

| Service | Purpose | DevOps Relevance |
|---------|---------|------------------|
| **AWS Lambda** | Serverless compute | Event-driven processing, microservices |
| **Amazon API Gateway** | API management | REST/HTTP/WebSocket APIs with Lambda |
| **Amazon EventBridge** | Serverless event bus | Decoupled, event-driven architectures |
| **Amazon SQS** | Message queuing | Decoupling, fan-out, reliable processing |
| **Amazon SNS** | Pub/Sub messaging | Notification, fan-out, topic distribution |
| **AWS Step Functions** | Workflow orchestration | Multi-step, stateful workflows |
| **Amazon Kinesis** | Real-time streaming | Real-time data processing at scale |
| **AWS SAM** | Serverless Application Model | Build, test, deploy serverless apps |
| **AWS CDK** | Cloud Development Kit | Define serverless in code |

---

## 🔄 Real DevOps Workflow Mapping

### Workflow: Event-Driven Order Processing

```
┌─────────────────────────────────────────────────────────────┐
│  Event-Driven Architecture                                  │
│                                                             │
│  API Gateway → Lambda (Create Order)                       │
│       │                                                    │
│       ├──→ EventBridge: "OrderCreated" event               │
│       │    ├──→ SQS: Inventory Queue (async processing)    │
│       │    ├──→ SNS: Notification Topic (email/SMS)        │
│       │    └──→ Kinesis: Real-time analytics stream         │
│       │                                                    │
│       └──→ Step Functions: Order Fulfillment Workflow       │
│            ├── Reserve Inventory                             │
│            ├── Process Payment                               │
│            ├── Ship Order                                    │
│            └── Send Confirmation                             │
└─────────────────────────────────────────────────────────────┘
```

### Workflow: Serverless API with SAM

```yaml
# template.yaml — AWS SAM Serverless API

AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Serverless API for order processing

Globals:
  Function:
    Timeout: 30
    MemorySize: 256
    Tracing: Active
    Environment:
      Variables:
        ORDER_TABLE: !Ref OrdersTable
        NOTIFY_TOPIC: !Ref NotifyTopic

Resources:
  # API Gateway
  OrdersApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
      Cors:
        AllowMethods: "'POST,GET,OPTIONS'"
        AllowHeaders: "'Content-Type,X-Amz-Date,Authorization,X-Api-Key'"

  # Lambda function
  CreateOrderFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/create-order/
      Handler: app.handler
      Runtime: python3.11
      Architecture: x86_64
      Events:
        CreateOrder:
          Type: Api
          Properties:
            Path: /orders
            Method: POST
            RestApiId: !Ref OrdersApi
      Policies:
        - DynamoDBWritePolicy:
            TableName: !Ref OrdersTable
        - SNSPublishMessagePolicy:
            TopicName: !Ref NotifyTopic
      EventsTable:
        Type: AWS::Serverless::FunctionUrl
        Properties:
          AuthType: NONE

  # DynamoDB table
  OrdersTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: orderId
        Type: String

  # SNS topic
  NotifyTopic:
    Type: AWS::SNS::Topic
    Properties:
      DisplayName: Order Notifications
```

### Workflow: Lambda Error Handling & Dead Letter Queue

```python
# Lambda function with SQS DLQ
import json
import boto3
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)
sqs = boto3.resource('sqs')

def lambda_handler(event, context):
    try:
        # Process the message
        message = json.loads(event['Records'][0]['body'])
        logger.info(f"Processing order: {message['order_id']}")

        # Business logic
        process_order(message)

        return {
            'statusCode': 200,
            'body': json.dumps({'status': 'success'})
        }

    except Exception as e:
        logger.error(f"Error processing order: {str(e)}")
        # SQS DLQ handles retries automatically
        raise e
```

---

## 📚 Recommended Resources

### 📖 AWS Documentation

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Amazon API Gateway Documentation](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Amazon EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [AWS Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)

### 🏗️ Well-Architected Framework

- [Serverless Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-pillar/welcome.html)
- [Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/using-lambda-with-dlq.html)

### 🛠️ Hands-On Tutorials

- [Serverless Web Application Tutorial](https://docs.aws.amazon.com/serverless-ref/architectures/web-architecture.html)
- [EventBridge Event Bus Workshop](https://catalog.workshops.aws/eventbridge)
- [Step Functions Visual Workflow](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)
- [SAM CLI Getting Started](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)

### 📄 Whitepapers

- [Building Serverless Architectures on AWS](https://aws.amazon.com/whitepapers/building-serverless-architectures-on-aws/)
- [Serverless on AWS](https://d1.awsstatic.com/whitepapers/serverless-compute-on-aws.pdf)

---

## 🏋️ Practical Exercises

### Exercise 1: Serverless API
1. Deploy a CRUD API for a todo app using SAM
2. Connect Lambda to DynamoDB for data persistence
3. Add API Gateway throttling and caching
4. Write integration tests with SAM local
5. Deploy to production with `sam deploy`

### Exercise 2: Event-Driven Fan-Out
1. Create an EventBridge custom event bus
2. Publish "OrderCreated" events
3. Create 3 rules targeting: SQS (inventory), SNS (email), Kinesis (analytics)
4. Deploy Lambda consumers for each target
5. Simulate 1000 events and verify all are processed

### Exercise 3: Step Functions Workflow
1. Design a 5-step order fulfillment workflow
2. Implement human approval step (wait state)
3. Add error handling with retries and catch states
4. Implement parallel processing for shipping and billing
5. View execution history in the Step Functions console

### Exercise 4: Lambda Best Practices
1. Create 3 Lambda functions: process, transform, notify
2. Configure provisioned concurrency for the critical path
3. Set up dead letter queues (SQS) for error handling
4. Implement Lambda layers for shared dependencies
5. Set up CloudWatch alarms for errors, duration, and throttles

---

## 💡 Pro Tips

- **Keep functions small and focused** — Single responsibility, under 10MB deployment package
- **Use provisioned concurrency for cold-start-sensitive apps** — Pre-warm for consistent performance
- **EventBridge over custom HTTP endpoints** — EventBridge is the event backbone; use API Gateway only for external APIs
- **DLQ is non-negotiable** — Every Lambda that processes messages needs a dead letter queue
- **SAM for local testing, CDK for complex stacks** — SAM for serverless-first, CDK for infrastructure-heavy
- **Set appropriate timeouts** — Default is 3s; most functions need 15-30s

---

## ✅ Progress Checklist

- [ ] Completed learning objectives
- [ ] Deployed Lambda functions with proper IAM
- [ ] Built serverless API with API Gateway + Lambda
- [ ] Designed event-driven architecture with EventBridge
- [ ] Implemented SQS fan-out pattern
- [ ] Created Step Functions workflow
- [ ] Deployed with SAM/CDK
- [ ] Configured error handling and DLQ
- [ ] Completed all practical exercises
- [ ] Documented learnings in personal notes
