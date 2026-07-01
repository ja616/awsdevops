# Challenge 4 — Findings

## Root cause

The Lambda function `challenge4-app-fn` failed on every invocation immediately after deployment, even though the application code itself was correct.

After investigating with AWS DevOps Agent and reviewing the CloudWatch logs, the root cause was identified as an **IAM permissions issue**. The Lambda execution role (`challenge-4-AppRole-*`) only had the managed policy `AWSLambdaBasicExecutionRole`, which provides permissions for writing logs to CloudWatch but does not allow access to DynamoDB.

The function attempts to retrieve an item from the DynamoDB table `challenge4-data` using the `dynamodb:GetItem` API. Since the execution role lacked this permission, every invocation failed with the following error:

```text
AccessDeniedException:
User: arn:aws:sts::069089526426:assumed-role/challenge-4-AppRole-.../challenge4-app-fn
is not authorized to perform:
dynamodb:GetItem
on resource:
arn:aws:dynamodb:us-east-1:069089526426:table/challenge4-data
```

This issue was particularly difficult to diagnose because:

- The Lambda function code contained no bugs.
- The DynamoDB table existed and was healthy.
- The failure was caused by infrastructure configuration introduced during deployment.
- The issue only became visible when the function attempted to access its dependency.

This demonstrates an important lesson in serverless applications:

> Correct application code can still fail if the execution environment and permissions are incorrectly configured.

---

## Investigation Process

### Step 1: Reproduced the issue
After deploying the `challenge-4` CloudFormation stack, I invoked the Lambda function multiple times using the **Test** tab in the Lambda console.

Each invocation failed and triggered the CloudWatch alarm:

```text
challenge4-app-fn-errors
```

---

### Step 2: Verified that the code looked correct
I reviewed the Lambda function source code and found that:

- The DynamoDB client was initialized correctly.
- The table name was correct.
- The API call syntax was valid.
- There were no syntax or logic errors.

Since the code appeared healthy, I suspected the issue was related to configuration or permissions.

---

### Step 3: Investigated with AWS DevOps Agent
I asked AWS DevOps Agent:

> My app `challenge4-app-fn` started failing on every request after a deploy, but the code looks correct. Investigate the root cause and tell me how to fix it.

The agent inspected:

- Lambda configuration
- CloudWatch logs
- IAM execution role
- DynamoDB dependencies
- Recent deployment changes

The agent identified the exact root cause:

- Missing `dynamodb:GetItem` permissions on the Lambda execution role.

---

### Step 4: Validated the finding
I checked the CloudWatch logs and confirmed the same `AccessDeniedException` reported by the agent.

This validated that the issue was not with the application code but with the IAM configuration.

---

## Fix applied

I updated the Lambda execution role by creating an inline IAM policy granting permission to read from the DynamoDB table.

### Policy added

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:069089526426:table/challenge4-data"
    }
  ]
}
```

The policy was attached to:

```text
challenge-4-AppRole-*
```

No application code changes were required.

---

## Verification

After updating the IAM permissions:

1. I invoked the Lambda function again.
2. The function executed successfully.
3. The function returned the seeded product from DynamoDB:

```json
{
  "statusCode": 200,
  "body": "{\"item\":{\"id\":{\"S\":\"1\"},\"product\":{\"S\":\"Builders Hoodie\"},\"price\":{\"N\":\"49\"}}}"
}
```

4. The CloudWatch alarm `challenge4-app-fn-errors` returned to the **OK** state.

---

## Lessons Learned

This challenge demonstrated an important DevOps principle:

### Code correctness does not guarantee application health.

Applications also depend on:

- IAM permissions
- Environment configuration
- Resource policies
- Service dependencies
- Infrastructure changes introduced during deployment

Even a small IAM misconfiguration can completely break an otherwise correct application.

This incident highlighted the value of:

- Infrastructure observability
- CloudWatch log analysis
- AWS DevOps Agent's dependency investigation capabilities
- Treating IAM configuration as part of the application itself

---

## Evidence

### Screenshot 1 – Root Cause Analysis
![Root Cause](Screenshots/root-image copy.png)

AWS DevOps Agent identifying the missing `dynamodb:GetItem` permission and explaining why the function was failing.

---

### Screenshot 2 – Recovery Verification
![Recovery](Screenshots/image.png)

Successful Lambda execution returning the product from DynamoDB and the `challenge4-app-fn-errors` alarm returning to the **OK** state.

Bonus
### Screenshot 3 – Recovery Verification
![Recovery](Screenshots/image copy 2.png)