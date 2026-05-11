# FAILURE CASCADE ANALYSIS

---

# Section 1 — Traffic Simulation Math

## Notification Reach

- Total users targeted = 180 million
- Expected click-through rate = 8%

Active users opening app:

14,400,000 users

Calculation:

14,400,000 = 180,000,000 × 0.08

---

## API Calls Per User

Average user actions in first 60 seconds:

- Open homepage
- Load restaurant list
- Search food
- Open restaurant
- Load menu

Approximate API calls per user:

5 calls

---

## Peak RPS Calculation

Formula:

Peak RPS = (active users × calls per user) / 60

Calculation:

= (14,400,000 × 5) / 60

= 1,200,000 RPS

Peak traffic:

1.2 million requests per second

---

# Section 2 — Component Capacity Numbers

## PostgreSQL Capacity

Configured max connections:

100

Average query time:

50ms

Average payment hold time:

200ms–2000ms

Formula:

Connections held =
(% non-payment RPS × query_time)
+
(% payment RPS × payment_hold_time)

Assume:

- 80% normal requests
- 20% payment requests
- Payment hold time = 1 second average

At only 500 RPS:

Connections held:

(400 × 0.05)
+
(100 × 1)

=
20 + 100

=
120 connections

Pool exhausted at approximately:

394–500 RPS

---

## Node.js Capacity

Single Node.js process:

- 1 CPU
- 4GB RAM

Estimated saturation:

12,000–15,000 RPS

Beyond this:
- callback queue backs up
- event loop latency spikes
- memory usage increases
- requests timeout

---

## Static Asset Saturation

No CDN exists.

All:
- images
- CSS
- JavaScript

served directly from Node.js.

NIC saturation occurs around:

2–3 Gbps traffic

Result:
- image load failures
- slow API responses
- TCP retries

---

# Section 3 — Failure Cascade

---

## Failure 1 — PostgreSQL Connection Pool Exhaustion

Severity: CRITICAL

Trigger:
~400 RPS

What users see:
- API requests timeout
- checkout hangs
- restaurants fail to load

What breaks next:
Node.js request queue grows uncontrollably

---

## Failure 2 — Node.js Event Loop Saturation

Severity: CRITICAL

Trigger:
12,000–15,000 RPS

What users see:
- 502 errors
- infinite loading
- app freeze

What breaks next:
OOM crash and process restart

---

## Failure 3 — Synchronous Payment Amplification

Severity: HIGH

Trigger:
Heavy checkout traffic

What users see:
- payment stuck
- duplicate payments
- order confirmation delays

What breaks next:
DB connections remain occupied longer

---

## Failure 4 — Promo Code Race Condition

Severity: HIGH

Trigger:
Concurrent coupon redemption

What users see:
- unlimited promo reuse
- incorrect discounts

What breaks next:
financial loss and DB contention

---

## Failure 5 — Static Asset NIC Saturation

Severity: MEDIUM

Trigger:
Traffic spike on homepage images

What users see:
- broken images
- extremely slow app loads

What breaks next:
API latency increases further

---

# Section 4 — Timeline

## T+0s

Push notification sent to 180 million users.

---

## T+10s

Traffic spike begins.
RPS exceeds safe PostgreSQL capacity.

---

## T+20s

Database connection pool exhausted.

New requests begin timing out.

---

## T+30s

Node.js event loop backlog increases rapidly.

Memory usage spikes.

---

## T+45s

Static assets slow dramatically.

Homepage partially unusable.

---

## T+60s

Checkout latency exceeds 15 seconds.

Payments begin failing.

---

## T+2m

Node.js process crashes due to memory exhaustion.

Entire platform unavailable.

---

## T+5m

Users retry aggressively.
Traffic doubles.

---

## T+15m

Social media complaints trend publicly.

Revenue impact severe.

---

## T+30m

Emergency scaling attempts begin.

No auto-scaling available.

---

## T+1h

Partial recovery after manual intervention.

---

## T+2h

System stabilizes.
Backlog drains slowly.