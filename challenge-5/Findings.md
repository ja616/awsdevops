# Challenge 5 — Findings

# Phantom Retraining Storm
An Agentic SRE Investigation into a Hidden MLOps Cost Incident

---

## What I built and how I broke it

I built a simulated machine learning retraining platform representing a fraud-detection model pipeline.

The environment consisted of:

- Lambda: `challenge5-training-trigger`
- EventBridge Schedule: `challenge5-training-schedule`
- CloudWatch custom metric: `RetrainingJobsStarted`
- CloudWatch alarm: `challenge5-retraining-storm`
- A SageMaker-themed retraining workflow that simulated model retraining activity.

The Lambda represented a production retraining orchestrator that would normally trigger SageMaker training jobs whenever new training data arrived.

I intentionally introduced a bad deployment by configuring the EventBridge scheduler to run continuously:

```text
rate(1 minute)
```

instead of a reasonable retraining cadence.

As a result, the retraining pipeline executed every minute and continuously generated retraining activity, simulating an unexpected increase in machine learning operational costs.

---

## Initial Symptoms

There were:

- No application failures
- No customer-facing outages
- No infrastructure errors
- No unhealthy services

The only observable symptom was unexpected retraining activity and increasing operational cost.

This mirrors real-world MLOps incidents where infrastructure appears healthy while hidden automation continuously consumes resources.

---

## What the agent found

I asked AWS DevOps Agent:

> A recent deployment caused AWS costs to increase unexpectedly. Investigate the environment and determine what changed.

The agent investigated the environment and determined that:

1. `challenge5-training-trigger` was executing continuously.
2. Retraining metrics were increasing over time.
3. The Lambda was being invoked by EventBridge.
4. The schedule `challenge5-training-schedule` had been configured to execute every minute.
5. The recent deployment introduced this aggressive schedule.

The agent correctly eliminated other possibilities such as:

- SageMaker service failures
- Application bugs
- Infrastructure outages
- Resource exhaustion

and identified the EventBridge schedule configuration as the root cause.

---

## Root Cause

A deployment introduced an incorrect scheduler configuration:

```text
rate(1 minute)
```

which continuously triggered the fraud-model retraining pipeline.

This created a **Phantom Retraining Storm**—a situation where machine learning infrastructure performs unnecessary work despite there being no customer impact or system failures.

The incident was caused entirely by infrastructure configuration and not by application code.

---

## Impact Analysis

### Operational Impact
- Continuous Lambda executions
- Continuous retraining activity
- Increased CloudWatch metric generation
- Increased operational noise

### Business Impact
- Unexpected AWS costs
- Risk of unnecessary model retraining
- Potential future SageMaker cost explosion if real training jobs were enabled

### Customer Impact
None.

This was purely an operational efficiency and FinOps incident.

---

## Fix applied

I disabled the EventBridge schedule:

```text
challenge5-training-schedule
```

which immediately stopped the retraining storm.

An alternative fix would be changing the schedule to a realistic cadence such as:

```text
rate(1 day)
```

or triggering retraining only when new data becomes available.

---

## Recovery Verification

After remediation:

✅ EventBridge schedule disabled.

✅ Lambda invocation count stabilized.

✅ `RetrainingJobsStarted` metric stopped increasing.

✅ AWS DevOps Agent confirmed the environment was healthy.

---

## Lessons Learned

1. Healthy applications can still generate incidents.

2. Cost anomalies are often caused by hidden automation and configuration drift.

3. Scheduled machine learning workflows require safeguards such as:
   - idempotency checks,
   - deployment reviews,
   - approval workflows,
   - schedule validation.

4. Agentic investigations are extremely valuable for identifying hidden operational problems that do not surface as traditional outages.

---

# Evidence

## Screenshot 1 – Agent Root Cause Analysis
![Agent Investigation](Screenshots/Screenshot%202026-07-01%20162334.png)

AWS DevOps Agent identifying the runaway retraining workflow and tracing the cost increase to `challenge5-training-schedule`.

![EventBridge Schedule](Screenshots/Screenshot%202026-07-01%20162347.png)



![CloudWatch Metrics](Screenshots/Screenshot%202026-07-01%20162909.png)



![Recovery](Screenshots/Screenshot%202026-07-01%20163208.png)



---

## Screenshot 5 – Agent Confirmation
![Healthy Environment](Screenshots/Screenshot%202026-07-01%20164631.png)

AWS DevOps Agent confirming that the environment is healthy after remediation.