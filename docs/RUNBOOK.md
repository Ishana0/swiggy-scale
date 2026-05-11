# INCIDENT RESPONSE RUNBOOK

---

# Purpose

This runbook is designed for on-call engineers responding to production incidents during high traffic events such as the India vs Pakistan World Cup Final.

The goal is to:
- detect failures quickly
- identify root cause within 30 seconds
- restore platform availability safely
- minimize revenue impact

This runbook assumes the engineer may have no prior familiarity with the system.

---

# STEP 1 — DETECT

## Required CloudWatch Alarms

| Alarm Name | Metric | Threshold | Severity |
|-------------|---------|------------|------------|
| High CPU Usage | EC2 CPUUtilization | > 75% for 5 minutes | Warning |
| Critical CPU Saturation | EC2 CPUUtilization | > 90% for 2 minutes | Critical |
| PostgreSQL Connection Exhaustion | DatabaseConnections | > 85 active connections | Critical |
| ALB 5XX Error Spike | HTTPCode_ELB_5XX_Count | > 100/minute | Critical |
| Redis Cache Miss Spike | Redis CacheMisses | > 40% miss rate | Warning |
| SQS Payment Queue Backlog | ApproximateNumberOfMessagesVisible | > 50,000 messages | Critical |
| Node.js Memory Spike | EC2 MemoryUtilization | > 85% | Warning |
| High Request Latency | ALB TargetResponseTime | > 2 seconds | Warning |
| Payment Failure Spike | PaymentFailureRate | > 5% failures | Critical |

---

# STEP 2 — TRIAGE

## Incident Triage Checklist

Follow these checks in order.

---

## 1. Check ALB 5XX Errors

Metric:
HTTPCode_ELB_5XX_Count

If:
- RED
- increasing rapidly

Then:
Go to Step 3A — Compute Saturation Response

If NOT:
Continue to next check.

---

## 2. Check PostgreSQL Connections

Metric:
DatabaseConnections

If:
> 85 active connections

Then:
Go to Step 3B — Database Pool Exhaustion Response

If NOT:
Continue to next check.

---

## 3. Check SQS Payment Queue

Metric:
ApproximateNumberOfMessagesVisible

If:
Queue depth growing continuously

Then:
Go to Step 3C — Payment Queue Backup Response

If NOT:
Continue to next check.

---

## 4. Check Redis Cache Miss Rate

Metric:
CacheMisses

If:
Miss rate exceeds 40%

Then:
Go to Step 3D — Redis Cache Failure Response

If NOT:
Continue to next check.

---

## 5. Check EC2 CPU Utilization

Metric:
CPUUtilization

If:
CPU > 90%

Then:
Go to Step 3A — Compute Saturation Response

If NOT:
Escalate to Senior SRE Team.

---

# STEP 3 — RESPOND

---

# Step 3A — Compute Saturation Response

## Symptoms

- High ALB 5XX errors
- CPU > 90%
- API latency spikes

---

## Immediate Action

### Increase Auto Scaling Group Capacity

Run:

```bash
aws autoscaling set-desired-capacity \
--auto-scaling-group-name swifteats-app-asg \
--desired-capacity 30
```

---

## Verify Recovery

Success indicators:

| Metric | Target |
|---------|---------|
| CPUUtilization | < 70% |
| ALB 5XX | decreasing |
| Response time | < 1 second |

Expected recovery:
5–10 minutes

---

## Responsible Team

Team:
Platform Engineering

Slack:
#oncall-platform

---

# Step 3B — Database Pool Exhaustion Response

## Symptoms

- DatabaseConnections > 85
- Request timeout spike
- Checkout failures

---

## Immediate Action

### Enable Read Replica Routing

Verify read traffic routing in application config.

Then restart application deployment:

```bash
kubectl rollout restart deployment/swifteats-api
```

---

### Increase PgBouncer Pool

Run:

```bash
kubectl edit configmap pgbouncer-config
```

Increase:

```text
default_pool_size = 500
```

Restart PgBouncer:

```bash
kubectl rollout restart deployment/pgbouncer
```

---

## Verify Recovery

| Metric | Target |
|---------|---------|
| DB connections | < 70 |
| Query latency | < 100ms |
| API errors | decreasing |

Expected recovery:
5 minutes

---

## Responsible Team

Team:
Database Reliability Team

Slack:
#oncall-database

---

# Step 3C — Payment Queue Backup Response

## Symptoms

- SQS queue depth increasing
- Delayed payment confirmations

---

## Immediate Action

### Scale Payment Workers

Run:

```bash
kubectl scale deployment payment-worker --replicas=20
```

---

## Verify Recovery

| Metric | Target |
|---------|---------|
| Queue depth | decreasing |
| Payment latency | < 5 seconds |

Expected recovery:
10–15 minutes

---

## Responsible Team

Team:
Payments Team

Slack:
#oncall-payments

---

# Step 3D — Redis Cache Failure Response

## Symptoms

- Cache miss spike
- Increased DB load
- Product API slowdown

---

## Immediate Action

### Flush Invalid Cache Entries

Run:

```bash
redis-cli FLUSHALL
```

---

### Restart Redis Cluster

Run:

```bash
kubectl rollout restart deployment/redis-cluster
```

---

## Verify Recovery

| Metric | Target |
|---------|---------|
| Cache hit rate | > 80% |
| DB CPU | decreasing |
| API latency | < 500ms |

Expected recovery:
5 minutes

---

## Responsible Team

Team:
Caching Infrastructure Team

Slack:
#oncall-cache

---

# STEP 4 — ROLLBACK

## When to Roll Back

Rollback ONLY if:

- error rate continues increasing after scaling
- deployment introduced new failures
- rollback risk is lower than outage impact

Do NOT rollback if:
- infrastructure saturation is the primary issue
- traffic spike exceeds current capacity

---

## Rollback Command

Rollback application deployment:

```bash
kubectl rollout undo deployment/swifteats-api
```

---

## Important Warning

NEVER rollback database schema changes during active incidents.

Only rollback:
- application code
- API deployments
- frontend deployments

Database rollback can cause:
- irreversible data corruption
- payment inconsistency
- order loss

---

## Verify Rollback

| Metric | Target |
|---------|---------|
| 5XX errors | decreasing |
| CPU usage | stabilizing |
| Latency | improving |

Expected stabilization:
5–15 minutes

---

# STEP 5 — POSTMORTEM TEMPLATE

# Incident Postmortem

---

## 1. Incident Summary

Describe:
- what failed
- customer impact
- duration
- business impact

---

## 2. Timeline

Format:

| Time | Event |
|------|-------|
| T+0m | Notification sent |
| T+2m | DB pool exhausted |
| T+5m | API failures spike |

Document exact timeline of events.

---

## 3. Root Cause

Describe:
- primary technical root cause
- triggering event
- why safeguards failed

---

## 4. Customer Impact

Include:
- number of users affected
- failed orders
- payment failures
- outage duration

---

## 5. What Worked Well

List:
- successful mitigations
- alerts that worked
- systems that remained healthy

---

## 6. What Failed

List:
- monitoring gaps
- delayed escalations
- architectural weaknesses

---

## 7. Action Items

| Action Item | Owner | Due Date |
|--------------|--------|-----------|
| Add more Redis nodes | Platform Team | 2026-06-01 |
| Improve payment retries | Payments Team | 2026-06-05 |

Every action item must include:
- owner
- deadline
- measurable outcome

---

# Final Notes

This runbook is intended to:
- reduce incident response time
- standardize recovery procedures
- prevent confusion during high-pressure outages

During incidents:
- prioritize restoring availability first
- communicate clearly in Slack war rooms
- document every major action taken
- avoid risky manual database modifications