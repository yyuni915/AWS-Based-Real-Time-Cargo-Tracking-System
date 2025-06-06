# Real-Time Freight Tracking System + Mobile Push Notification Architecture

> **Sendy** is Korea's No.1 freight booking service. After a successful reservation, customers should be able to track the driver’s real-time location throughout the delivery process.

## Project Overview

This project enables customers to track their cargo delivery in real-time after matching with a driver. It supports WebSocket-based live updates and push notifications for delivery status changes.

Key goals included:
- Customers must receive real-time location data for the matched driver after booking.
- A separate driver-only app should transmit location data in JSON format as a stream.
- Consider using Kinesis Data Stream and Kinesis Data Firehose for handling the stream.
- Log location information via Elasticsearch (OpenSearch).

## AWS Architecture

![System Diagram](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a4cf07a-8316-47cb-ad62-93c5b2e3c38a/Untitled.png)

### Core Flow (Live Tracking):
1. Driver clicks "Start Location Sharing" → sends current location to API Gateway.
2. API Gateway forwards location data to Kinesis Data Stream.
3. Kinesis Data Stream routes the data to:
   - Firehose A → OpenSearch for log indexing.
   - Firehose A → S3 for backup storage.
4. Firehose B sends data to a Lambda function for WebSocket delivery.
5. Lambda checks if the WebSocket connection for the corresponding truck ID exists using DynamoDB.
6. If it exists, Lambda sends the data via WebSocket to the user.
7. Customer receives location updates in real time through WebSocket API.
8. When the WebSocket opens/closes, the connection ID and truck ID are recorded/deleted in DynamoDB.

### Push Notification Flow (Status Change):
9. Driver updates delivery status → API Gateway forwards the event.
10. Lambda processes the status change and publishes to SNS.
11. SNS pushes the message to:
   - SQS A → triggers Lambda to send mobile/email notifications.
   - SQS B → triggers Lambda to log the status in OpenSearch.

## Technologies Used

- **Backend/Architecture**: AWS Lambda, API Gateway, Kinesis Stream/Firehose, DynamoDB, OpenSearch
- **Messaging**: SNS, SQS
- **Frontend Interface**: WebSocket API
- **Infrastructure as Code**: Terraform, Serverless Framework
- **CI/CD**: GitHub Actions, Shell Script (for sequencing)
- **Monitoring/Logging**: CloudWatch, OpenSearch
- **Languages**: Python, Shell

## Architectural Decisions

After evaluating multiple options, the final design chose to directly connect **Kinesis Firehose → Lambda → WebSocket API** instead of using OpenSearch as a buffer.

### Why?

- Lambda-based routing offers lower latency and simpler logic than querying OpenSearch.
- Serverless structure fits well with Kinesis Firehose → Lambda pipeline.
- Querying OpenSearch with polling or webhooks introduced unnecessary delay and cost.

We used **DynamoDB** to manage open WebSocket connections and track which clients are listening to which truck IDs in real time.

## CI/CD Strategy

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

## Related Technologies & Concepts
- Kafka – Evaluated for large-scale stream processing
- Kubernetes – Explored as a future option for container orchestration and scaling
- Jenkins – Considered for legacy CI workflows
---


