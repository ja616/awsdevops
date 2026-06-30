# Challenge 1 — Findings

## What I asked the agent
1. What resources do I have in this account?
2. Is anything unhealthy right now?
3. Give me a health summary of my environment.
4. Can you show me the topology of my AWS resources and explain how they are connected?

## What the agent told me
The agent discovered 95 AWS resources across my account, with the primary application being a serverless application called `finchat-lite`.

The application architecture consists of:
- API Gateway exposing five endpoints
- Five Lambda functions handling chatbot, authentication, transfers, and beneficiary management
- Three DynamoDB tables storing customers, beneficiaries, and transactions
- Additional infrastructure including SageMaker, VPC resources, IAM roles, and CloudWatch log groups.

The agent reported that the overall environment health was **Degraded** because an RDS Proxy existed without any database backend attached to it. It also identified several recommendations, including:
- API endpoints do not have authentication configured.
- No CloudWatch alarms are configured.
- API Gateway is using TLS 1.0 instead of TLS 1.2.

The agent generated a topology view and explained how requests flow through the system:

Client → API Gateway → Lambda Functions → DynamoDB Tables.

## Key Insights
- The `finchat-lite` application is a fully serverless architecture.
- The orphaned RDS Proxy is currently not part of the application's request path.
- The agent was able to reason about relationships between resources instead of only listing them.

## Evidence
- [x] Screenshot: Agent response listing account resources.
- [x] Screenshot: Environment health summary.
- [x] Screenshot: Infrastructure topology and request flow.