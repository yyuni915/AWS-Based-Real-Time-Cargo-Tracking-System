# Real-Time Cargo Tracking System with Mobile Push Alerts

> Architected and implemented a real-time logistics tracking and mobile notification system for **Sendy**, a leading freight platform in Korea.

## Project Overview

This project enables customers to track their cargo delivery in real-time after matching with a driver. It supports WebSocket-based live updates and push notifications for delivery status changes.

Key goals included:
- Real-time delivery location updates via WebSocket.
- Scalable stream processing architecture using AWS Kinesis.
- Efficient push alert delivery using Lambda and SNS.
- Fully serverless design with automated CI/CD.

## 📷 System Architecture

![System Diagram](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a4cf07a-8316-47cb-ad62-93c5b2e3c38a/Untitled.png)

### Core Flow (Live Tracking):
1. Driver sends location via API Gateway.
2. Data is streamed into **Kinesis Data Stream**.
3. Stream is forwarded to:
   - **Kinesis Firehose A** → OpenSearch + S3 backup.
   - **Kinesis Firehose B** → Lambda.
4. Lambda checks active WebSocket connections via **DynamoDB**.
5. Sends real-time data to the user using **WebSocket API**.

### Push Notification Flow (Status Change):
1. Driver triggers status update → API Gateway → Lambda.
2. Lambda publishes message to **SNS**.
3. SNS triggers:
   - **SQS A** → Lambda → Mobile/email push.
   - **SQS B** → Lambda → OpenSearch update for status log.

## ⚙️ Technologies Used

- **Backend/Architecture**: AWS Lambda, API Gateway, Kinesis Stream/Firehose, DynamoDB, OpenSearch
- **Messaging**: SNS, SQS
- **Frontend Interface**: WebSocket API
- **Infrastructure as Code**: Terraform, Serverless Framework
- **CI/CD**: GitHub Actions, Shell Script (for sequencing)
- **Monitoring/Logging**: CloudWatch, OpenSearch
- **Languages**: Python, Shell

## 🧠 Architectural Decisions

After evaluating multiple options, the final design chose to directly connect **Kinesis Firehose → Lambda → WebSocket API** instead of using OpenSearch as a buffer.

### Why?

- Lambda-based routing offers lower latency and simpler logic than querying OpenSearch.
- Serverless structure fits well with Kinesis Firehose → Lambda pipeline.
- Querying OpenSearch with polling or webhooks introduced unnecessary delay and cost.

We used **DynamoDB** to manage open WebSocket connections and track which clients are listening to which truck IDs in real time.

## 🔄 CI/CD Strategy

- Used **GitHub Actions** for deployment automation.
- **Lambda functions** managed with Serverless Framework.
- **Infrastructure** like Kinesis/SQS/SNS managed via Terraform.
- To handle deployment order (e.g., Lambda URL needed before Firehose config), we:
  - Extracted Lambda URL using AWS CLI.
  - Wrote a shell script to deploy Serverless first, then apply Terraform.

## 🧩 Key Challenges & Solutions

| Challenge | Solution |
|----------|----------|
| Deployment order between Serverless and Terraform | Used shell script to orchestrate CLI-based staged deployment |
| Mapping Lambda URL into Firehose HTTP destination | Extracted URL via CLI and injected it using `terraform apply -var` |
| Real-time connection filtering | Used DynamoDB lookups for WebSocket connections and active truck mapping |

## 📈 Lessons Learned

As a DevOps-focused engineer, I realized that **architecture decisions drive success**. Theoretical designs that look great on paper may introduce latency, cost, or management complexity. The best architecture balances simplicity, scalability, and reliability.

**Key Takeaways**:
- Serverless + Kinesis is powerful for real-time stream processing.
- Direct WebSocket push beats polling for instant feedback.
- CI/CD across multiple IaC tools (Terraform + Serverless) must consider dependency order.

## 🛠️ Deployment (Coming Soon)

Terraform/Serverless code will be added with examples for:

- `driver-location-lambda`
- `firehose-to-websocket-pipeline`
- `sns-sqs-status-push`
- `DynamoDB connection tracking`

## 🔗 Related Tech/Concepts

- Kafka (evaluated for streaming)
- Kubernetes (explored for future scalability)
- CI/CD automation (GitHub Actions + AWS CLI)
- Monitoring: OpenSearch, CloudWatch
- Real-time systems: WebSocket, Pub/Sub, Serverless Patterns

---


