# Challenge 2 — Findings

## Root cause
The AWS DevOps Agent investigated the failing Lambda function `challenge2-broken-fn` and identified that every invocation was failing due to a Python `NameError`.

The function code attempted to access:

```python
config["value"]
```

However, the variable `config` was never defined anywhere in the function code, not passed in the event payload, and not configured as an environment variable. As a result, each invocation terminated with the following error:

```text
NameError: name 'config' is not defined
```

## Fix applied
I modified the Lambda function code by defining the missing `config` variable before it was referenced.

```python
config = {"value": "fixed"}

def handler(event, context):
    return {"result": config["value"]}
```

After deploying the updated code, I invoked the function multiple times and verified that the executions completed successfully without errors. The CloudWatch alarm `challenge2-broken-fn-errors` subsequently returned to the **OK** state.

## Investigation Process
1. Deployed the `challenge-2` CloudFormation stack.
2. Triggered the Lambda function several times to reproduce the failure.
3. Asked AWS DevOps Agent to investigate the failing function.
4. Reviewed the agent's root-cause analysis and error details.
5. Updated the Lambda code to define the missing variable.
6. Redeployed and retested the function.
7. Verified recovery through successful invocations and CloudWatch alarms.

## Evidence

### Screenshot 1 – Root Cause Analysis
![Root Cause](screenshots/root-cause-analysis.png)

The AWS DevOps Agent identified the `NameError` and explained why the function was failing.

### Screenshot 2 – Recovery Verification
![Recovery](screenshots/recovery-verification.png)

Successful Lambda invocation and the `challenge2-broken-fn-errors` alarm returning to the **OK** state.