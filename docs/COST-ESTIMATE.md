# AWS Cost Estimate

---

# Assumptions

- Region: AWS ap-south-1 (Mumbai)
- Runtime: 24/7 production deployment
- Monthly hours = 720
- Peak traffic event duration = 4 hours
- User scale target = 10 million users
- Architecture includes:
  - CloudFront CDN
  - ALB
  - Auto-scaled Node.js EC2 cluster
  - Redis cache
  - PostgreSQL primary + replicas
  - SQS payment queue
  - Payment workers

---

# Baseline Monthly Cost (Normal Traffic)

## 1. EC2 — Node.js Application Servers

### Instance Type
t3.large

### Pricing
$0.0832/hour

### Quantity
6 instances

### Calculation

0.0832 × 720 × 6

= $359.42/month

---

## 2. RDS PostgreSQL Primary Database

### Instance Type
db.r6g.large

### Pricing
$0.252/hour

### Quantity
1 primary instance

### Calculation

0.252 × 720 × 1

= $181.44/month

---

## 3. RDS PostgreSQL Read Replicas

### Instance Type
db.r6g.large

### Pricing
$0.252/hour

### Quantity
2 replicas

### Calculation

0.252 × 720 × 2

= $362.88/month

---

## 4. ElastiCache Redis Cluster

### Instance Type
cache.r6g.large

### Pricing
$0.138/hour

### Quantity
2 nodes

### Calculation

0.138 × 720 × 2

= $198.72/month

---

## 5. Application Load Balancer (ALB)

### Base Pricing
$0.0225/hour

### LCU Estimate
$0.008 per LCU-hour

### Estimated LCUs
5 LCUs average

### Calculation

Base:
0.0225 × 720
= $16.20

LCU:
0.008 × 5 × 720
= $28.80

Total ALB Cost:
= $45.00/month

---

## 6. CloudFront CDN

### Estimated Data Transfer
25 TB/month

### Transfer Cost
Approx $0.085/GB

### Request Estimate
500 million requests/month

### Request Cost
$0.0075 per 10,000 requests

---

### Transfer Calculation

25,000 GB × 0.085

= $2,125/month

---

### Request Calculation

(500,000,000 / 10,000) × 0.0075

= $375/month

---

### Total CloudFront Cost

= $2,500/month

---

## 7. Amazon SQS

### Estimated Messages
150 million/month

### Pricing
$0.40 per million requests

### Calculation

150 × 0.40

= $60/month

---

## 8. Payment Worker EC2 Instances

### Instance Type
t3.medium

### Pricing
$0.0416/hour

### Quantity
2 instances

### Calculation

0.0416 × 720 × 2

= $59.90/month

---

# Baseline Monthly Total

| Service | Monthly Cost |
|----------|--------------|
| EC2 Node.js Cluster | $359.42 |
| RDS Primary | $181.44 |
| RDS Replicas | $362.88 |
| Redis Cluster | $198.72 |
| ALB | $45.00 |
| CloudFront | $2,500.00 |
| SQS | $60.00 |
| Payment Workers | $59.90 |

---

# Total Baseline Cost

# = $3,767.36/month

---

# Peak Event Cost (World Cup Final Night)

## Peak Assumptions

- Traffic spike duration = 4 hours
- Auto scaling activates
- CDN traffic increases massively
- Extra payment workers enabled

---

## 1. Additional EC2 Node.js Instances

### Extra Instances
20 additional t3.large instances

### Calculation

0.0832 × 4 × 20

= $6.66

---

## 2. Additional Redis Scaling

### Extra Redis Node
1 additional cache.r6g.large

### Calculation

0.138 × 4 × 1

= $0.55

---

## 3. Additional Payment Workers

### Extra Workers
4 additional t3.medium instances

### Calculation

0.0416 × 4 × 4

= $0.67

---

## 4. CloudFront Surge Traffic

### Additional Transfer
10 TB during event

### Calculation

10,000 GB × 0.085

= $850

---

## 5. Additional ALB LCU Usage

### Increased LCUs
25 LCUs for 4 hours

### Calculation

0.008 × 25 × 4

= $0.80

---

# Total Peak Event Cost

| Component | Additional Cost |
|------------|----------------|
| Extra EC2 Instances | $6.66 |
| Extra Redis Scaling | $0.55 |
| Extra Payment Workers | $0.67 |
| CloudFront Surge Traffic | $850.00 |
| Extra ALB LCUs | $0.80 |

---

# Total Peak Event Cost

# = $858.68 for 4 hours

---

# Business Justification

A 45-minute outage during a major event such as the India vs Pakistan World Cup Final is estimated to cause revenue losses of:

₹4.2 crore/minute × 45 minutes

= ₹189 crore revenue loss

Compared to this, the monthly AWS infrastructure cost of approximately:

$3,767/month

(roughly ₹3.1 lakh/month)

is extremely small.

Even the peak event scaling cost of:

$858.68

is negligible compared to the potential business loss from downtime.

The redesigned architecture therefore provides extremely high ROI because it:

- prevents catastrophic outages
- maintains customer trust
- supports 10 million concurrent users
- enables safe horizontal scaling
- avoids payment system collapse during traffic spikes

The infrastructure investment is financially justified because preventing even a few minutes of outage saves significantly more revenue than the monthly cloud operating cost.