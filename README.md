# Swiggy Scale — World Cup Final Traffic Resilience Analysis

A large-scale backend architecture redesign and incident response analysis for handling 10 million concurrent users during the India vs Pakistan World Cup Final traffic spike.

---

# Project Scenario

This project analyzes how a monolithic Swiggy-like food delivery platform would behave during one of the highest traffic events possible:

- India vs Pakistan World Cup Final
- 180 million push notifications sent
- 10+ million active concurrent users
- Massive checkout and payment surge

The original system architecture contains:
- a single Node.js server
- a single PostgreSQL database
- no Redis cache
- no CDN
- synchronous payment processing
- no auto scaling

The goal of this project is to:
- identify the exact failure cascade
- redesign the architecture for resiliency
- estimate real AWS infrastructure costs
- create a production-grade incident runbook

---

# Project Documents

| Document | Description |
|----------|-------------|
| FAILURE-CASCADE.md | Complete traffic simulation, bottleneck analysis, and outage timeline |
| ARCHITECTURE.md | Redesigned distributed system architecture for 10 million users |
| COST-ESTIMATE.md | Real AWS pricing calculations for baseline and peak traffic |
| RUNBOOK.md | 2 AM production incident response guide and recovery procedures |

---

# Key Findings

- The original PostgreSQL database exhausts its connection pool at approximately 394–500 RPS.
- Peak traffic during the World Cup Final is estimated at 1.2 million requests per second.
- A single Node.js process saturates at roughly 12k–15k RPS.
- A 45-minute outage could result in approximately ₹189 crore revenue loss.
- Synchronous payment processing dramatically amplifies database connection exhaustion during checkout spikes.

---

# Architecture Overview

The redesigned architecture converts the original monolith into a horizontally scalable distributed system using AWS cloud infrastructure.

The new system includes:
- CloudFront CDN for static asset delivery
- Application Load Balancer with health checks
- Auto-scaled Node.js application servers
- Redis caching and promo locking
- PgBouncer connection pooling
- PostgreSQL primary with read replicas
- Amazon SQS payment queues
- Dedicated payment worker services

This redesign removes single points of failure and allows the platform to safely handle massive traffic spikes.

---

# Tech Stack Context

This analysis and redesign focuses on technologies commonly used in modern scalable backend systems:

- Node.js
- PostgreSQL
- Redis
- AWS EC2
- AWS RDS
- AWS CloudFront
- AWS SQS
- AWS Auto Scaling
- PgBouncer
- Application Load Balancer

---

# Repository Goal

The purpose of this repository is to demonstrate:
- backend scalability analysis
- distributed systems thinking
- cloud infrastructure planning
- incident response design
- production resiliency engineering

under realistic high-scale traffic conditions.