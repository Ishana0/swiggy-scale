# ARCHITECTURE REDESIGN

---

# Section 1 — Current Architecture Diagram

## Existing Monolith Architecture

                           +-------------------+
                           |   Mobile Clients  |
                           +-------------------+
                                      |
                                      |
                                      v
                    +--------------------------------+
                    |   Single Node.js Express App   |
                    |   1 CPU / 4GB RAM              |
                    +--------------------------------+
                       |        |            |
                       |        |            |
                       v        v            v

          ⚠️ Failure 5          ⚠️ Failure 2
          Static assets         Event loop saturation
          served directly       at 12k–15k RPS

                                      |
                                      |
                                      v

                    +--------------------------------+
                    |       PostgreSQL DB            |
                    |   max_connections = 100        |
                    +--------------------------------+

                     ⚠️ Failure 1
                     Pool exhaustion
                     at ~400 RPS

                                      |
                                      |
                                      v

                    +--------------------------------+
                    |     Razorpay Payment API       |
                    +--------------------------------+

                     ⚠️ Failure 3
                     Synchronous payment calls
                     hold DB connections 200–2000ms


                     ⚠️ Failure 4
                     Promo code race condition
                     caused by concurrent requests

---

# Section 2 — New Architecture Diagram

## Redesigned Scalable Architecture

                         +----------------------+
                         |    Mobile Clients    |
                         +----------------------+
                                      |
                                      |
                                      v

                    +-----------------------------------+
                    |        CloudFront CDN             |
                    |  Caches images/static files       |
                    |  TTL = 5 minutes                  |
                    +-----------------------------------+

                                      |
                                      |
                                      v

                    +-----------------------------------+
                    |   AWS Application Load Balancer   |
                    | - SSL termination                 |
                    | - Health checks                   |
                    | - Rate limiting                   |
                    +-----------------------------------+

                                      |
                     -----------------------------------
                     |                |                |
                     v                v                v

           +----------------+ +----------------+ +----------------+
           | Node.js App 1  | | Node.js App 2  | | Node.js App N  |
           +----------------+ +----------------+ +----------------+

                Auto-scale trigger:
                CPU > 70%
                OR
                RPS spike detected

                     |                |                |
                     -----------------------------------
                                      |
                                      v

                    +-----------------------------------+
                    |         Redis Cache               |
                    | - Product cache                   |
                    | - Restaurant cache                |
                    | - Session cache                   |
                    | - Promo code locking              |
                    | TTL = 1–5 minutes                 |
                    +-----------------------------------+

                                      |
                                      |
                                      v

                    +-----------------------------------+
                    |           PgBouncer               |
                    | Connection pooling layer          |
                    | Reuses DB connections             |
                    +-----------------------------------+

                                      |
                     -----------------------------------
                     |                                 |
                     v                                 v

         +------------------------+      +------------------------+
         | PostgreSQL Primary DB  |      | PostgreSQL Read Replica|
         +------------------------+      +------------------------+

          Writes:
          - orders
          - payments
          - coupons

                                          Reads:
                                          - menus
                                          - restaurants
                                          - order history

                                      |
                                      |
                                      v

                    +-----------------------------------+
                    |          Amazon SQS               |
                    |     Payment processing queue      |
                    +-----------------------------------+

                                      |
                                      |
                                      v

                    +-----------------------------------+
                    |      Payment Worker Service       |
                    | - Reads messages from SQS         |
                    | - Calls Razorpay asynchronously   |
                    | - Updates payment status          |
                    +-----------------------------------+

---

# Section 3 — Component Justification Table

| Component | Failure It Prevents | How It Prevents It |
|-----------|---------------------|--------------------|
| CloudFront CDN | Failure 5: NIC saturation | Serves static assets from edge servers so Node.js never handles image traffic |
| Application Load Balancer | Failure 2: Event loop saturation | Distributes traffic across multiple Node.js instances |
| Multiple Node.js Instances | Failure 2: Event loop saturation | Horizontal scaling increases request handling capacity |
| Auto Scaling Group | Failure 2: Event loop saturation | Automatically launches more servers during traffic spikes |
| Redis Cache | Failure 1: DB pool exhaustion | Reduces repeated database reads by caching hot data |
| Redis Promo Locking | Failure 4: Promo code race condition | Prevents multiple users from redeeming same coupon simultaneously |
| PgBouncer | Failure 1: DB connection exhaustion | Reuses and multiplexes PostgreSQL connections efficiently |
| PostgreSQL Read Replicas | Failure 1: DB overload | Offloads read traffic from primary database |
| Amazon SQS | Failure 3: Payment amplification | Removes synchronous payment dependency from request lifecycle |
| Payment Worker Service | Failure 3: Payment amplification | Processes payments asynchronously outside user request flow |
| Health Checks | Failure 2: Application crashes | Removes unhealthy Node.js instances automatically |
| Rate Limiting | Failure 2: Traffic spikes | Prevents overload from sudden burst traffic |
| SSL Termination at ALB | Failure 2: CPU overhead | Offloads TLS encryption from application servers |

---

# Final Architecture Summary

The redesigned architecture converts the original monolith into a horizontally scalable distributed system capable of handling millions of concurrent users.

The key architectural improvements are:

- CDN-based static asset delivery
- Horizontal Node.js scaling
- Redis caching
- Database connection pooling
- Read replica separation
- Asynchronous payment processing
- Auto scaling and load balancing

These changes eliminate the single points of failure identified in the failure cascade analysis and significantly improve resiliency during massive traffic spikes such as the India vs Pakistan World Cup Final.