# Backend System Design — A Structured Course

**A 12-week self-study syllabus · 5 parts · 9 modules · 23 core concepts · 35 production case studies**

Source material: a curated list of 10 learning libraries, 23 canonical system-design concepts, and 35 production architecture write-ups from major engineering teams. Every concept below is paired with a hands-on lab; every case study is mapped back to the concepts it reinforces.

---

## 1. How this course works

Each week follows the same four-step rhythm (~9 hours):

| Step | Time | What you do |
|---|---|---|
| **Read** | 3 h | Study the module's concepts from the core reading libraries |
| **Diagram** | 2 h | Redraw the architecture from memory — no peeking |
| **Build** | 3 h | Implement the module's lab (small, working, real) |
| **Design doc** | 1 h | Write a one-page answer to "what breaks at 100× scale?" |

Rules of engagement:

1. **Diagram before you build.** If you can't draw it, you can't build it.
2. **Every lab must run.** A queue that "mostly works" teaches you exactly-once lies.
3. **End every module with a design doc.** One page: requirements, estimates, architecture, top 2 bottlenecks.
4. **Case studies come after concepts, never before.** They are the exam, not the lecture.

---

## 2. Course map

| Part | Module | Concepts covered | Weeks |
|---|---|---|---|
| I — Foundations | M01 Scaling Fundamentals | Horizontal vs Vertical Scaling; Back-of-the-Envelope Estimation | 1 |
| I — Foundations | M02 The Request Path | Load Balancing; CDN; API Gateway; Rate Limiting | 2 |
| II — The Data Layer | M03 Storage at Scale | Database Scaling; Replication; Sharding; Partitioning | 3–4 |
| II — The Data Layer | M04 Consistency & Transactions | CAP Theorem; Consistency Models; Eventual Consistency; Distributed Transactions | 5 |
| III — Performance & Resilience | M05 Caching | Caching; Cache Invalidation | 6 |
| III — Performance & Resilience | M06 Resilience | Fault Tolerance; Idempotency & Data Latency | 7 |
| IV — Architecture Patterns | M07 Async Systems & Coordination | Queues; Microservices; Microservices vs Monoliths; Service Discovery; Leader Election | 8–9 |
| V — Practice | M08 Case Study Lab | 35 production architectures, in 8 themes | 10–11 |
| V — Practice | M09 Design Workshop & Capstone | Interview framework; capstone builds | 12 |

Progression logic: **cost math → traffic path → state → failure → interaction patterns → proof in production.** Each part assumes the previous one; don't skip ahead.

---

## 3. Module syllabus

### Part I — Foundations

#### M01 · Scaling Fundamentals (Week 1)

**Goal:** reason about load mathematically before touching any architecture diagram.

- **Concepts:** Horizontal vs Vertical Scaling · Back-of-the-Envelope Estimation
- **Core questions:** When does buying a bigger box stop working? What does replication cost in latency and complexity? What is "a lot" of QPS, honestly?
- **Drill:** memorize the latency numbers every engineer should know (L1/L2 cache, RAM, SSD seek, network round trip within/between datacenters) and practice powers-of-2 storage math.
- **Lab:** estimate QPS, storage growth/day, bandwidth, and cache footprint for five products: URL shortener, news feed, group chat, video streaming, ride hailing.
- **Design doc prompt:** "Estimate the infrastructure for a WhatsApp-class messenger at 50M DAU."

#### M02 · The Request Path (Week 2)

**Goal:** master everything between a user's tap and your server.

- **Concepts:** Load Balancing (L4 vs L7, algorithms, health checks) · CDN (edge caching, pull vs push, cache headers) · API Gateway (routing, auth, aggregation) · Rate Limiting (token bucket, sliding window, distributed limiters)
- **Core questions:** Where does TLS terminate? What belongs in the gateway vs the service? Where do you rate-limit: edge, gateway, or service?
- **Lab:** implement a token-bucket rate limiter; then draw the complete request path for a read-heavy product, edge to database.
- **Design doc prompt:** "Your launch-day traffic is 40× forecast. Where does the path break first, and in what order?"

### Part II — The Data Layer

#### M03 · Storage at Scale (Weeks 3–4)

**Goal:** make databases scale on purpose, not by accident.

- **Concepts:** Database Scaling (indexes, denormalization, read replicas, SQL vs NoSQL selection) · Replication (leader–follower, multi-leader, leaderless; sync vs async; failover) · Sharding (shard-key choice, hotspots, rebalancing, consistent hashing) · Partitioning (range vs hash)
- **Core questions:** Does your workload scale reads or writes? What makes a good shard key, and what makes a famous outage? What does resharding cost at 2 a.m.?
- **Lab:** build a key-value store with consistent-hashing sharding (route via *Build Your Own X*).
- **Design doc prompt:** "Shard a 10 TB user table with near-zero downtime. Walk through key choice and rebalancing."

#### M04 · Consistency & Transactions (Week 5)

**Goal:** stop using "eventually consistent" as an excuse; start using it as a decision.

- **Concepts:** CAP Theorem (plus the PACELC intuition) · Consistency Models (strong, sequential, causal, read-your-writes, eventual) · Eventual Consistency (anti-entropy, read repair, versioning/vector clocks) · Distributed Transactions (2PC and why people fear it, Sagas, the outbox pattern)
- **Core questions:** Which consistency model does each user-facing feature actually need? What compensates a failed saga step? Why do dual writes corrupt data?
- **Lab:** implement a saga with compensating actions for an order + payment two-service flow; then break it with a dual write and fix it with an outbox.
- **Design doc prompt:** "Pick the consistency model for: a shopping cart, a bank ledger, a social feed. Defend each choice."

### Part III — Performance & Resilience

#### M05 · Caching (Week 6)

**Goal:** make reads fast without making correctness subtle in a bad way.

- **Concepts:** Caching (layers: browser → CDN → app → DB; patterns: cache-aside, write-through, write-behind; eviction: LRU/LFU/TTL; thundering herd) · Cache Invalidation (TTL, explicit purge, versioned keys — and why it's called one of the two hard problems)
- **Core questions:** Which layer gives the biggest win per dollar? What happens when a hot key expires at peak? What is your invalidation story when data changes out-of-band?
- **Lab:** add cache-aside to a real API, measure hit/miss ratios, then trigger a stampede and fix it (request coalescing or locks).
- **Design doc prompt:** "Design the full cache stack for a product-detail page with 10M SKUs."

#### M06 · Resilience (Week 7)

**Goal:** design for the dependency that will fail — it will.

- **Concepts:** Fault Tolerance (retries, timeouts, circuit breakers, bulkheads, graceful degradation, chaos testing) · Idempotency & Data Latency (at-least-once vs exactly-once, idempotency keys, deduplication, why exactly-once is a contract not a network property)
- **Core questions:** What does your system do when the payment provider is down but reachable? Which retries make things worse (retry storms)? How do duplicate submits stay safe?
- **Lab:** wrap a deliberately flaky dependency with a circuit breaker + retry with exponential backoff and jitter; verify state stays consistent under duplicates.
- **Design doc prompt:** "Make a payment API safe under duplicate submits and a 30 s provider outage."

### Part IV — Architecture Patterns

#### M07 · Async Systems & Coordination (Weeks 8–9)

**Goal:** move work off the request path, and get services to agree on who's in charge.

- **Concepts:** Queues (Kafka vs RabbitMQ mental models, delivery semantics, retries, dead-letter queues) · Microservices (service boundaries, data ownership) · Microservices vs Monoliths (splitting costs, the modular monolith option) · Service Discovery (registries, health checks, client-side vs server-side) · Leader Election (leases, consensus, Raft intuition)
- **Core questions:** Which operations are sync by contract and which are only sync by habit? What breaks when two services both "own" the user record? Who leads when the leader is half-dead?
- **Lab:** build a mini message queue with at-least-once delivery, consumer retries, and an idempotent consumer.
- **Design doc prompt:** "Monolith or microservices for a 12-engineer startup shipping a marketplace? Write the decision memo both ways."

### Part V — Practice

#### M08 · Case Study Lab (Weeks 10–11)

**Goal:** see every concept from Parts I–IV surviving contact with production.

Work the 35 case studies in 8 themed sprints (full catalog in §4). Per case: 20-minute skim → redraw the architecture from memory → write two lines: *one decision you'd copy, one trade-off you'd question*.

#### M09 · Design Workshop & Capstone (Week 12)

**Goal:** perform under pressure, and ship one thing you built end-to-end.

- **The design-interview framework** (also your real-world design-doc skeleton):
  1. Requirements: functional + non-functional (scale, latency, consistency, availability)
  2. Estimation: QPS, storage, bandwidth (M01 pays off here)
  3. API surface: core endpoints
  4. Data model: entities, indexes, partitioning
  5. High-level design: request path, async boundaries
  6. Deep dives: bottlenecks, failure modes, consistency choices
- **Capstone options** (all via *Build Your Own X*): a Raft-based KV store with leader election · a message queue with delivery guarantees · a load balancer · a rate limiter · a URL shortener with a full design doc and cache/cdn layering.
- **Interview prep layer:** work through *Tech Interview Handbook* and *Coding Interview University* in parallel from Week 8, not Week 12.

---

## 4. Case study catalog — 35 architectures in 8 themes

| # | Theme | Cases | Reinforces |
|---|---|---|---|
| 1 | **Real-Time & Messaging** | Discord (Trillion Message Indexing) · Twilio (Exactly-Once Delivery) · Slack (Cellular Architecture Migration) · Netflix (Distributed Tracing Infrastructure) | M04 consistency, M06 idempotency, M07 queues, observability |
| 2 | **Storage & Data Infrastructure** | Dropbox (Magic Pocket) · GitHub (Distributed Storage System) · Airbnb (Key-Value Architecture) · Datadog (Husky Event Store) | M03 storage/sharding, M04 consistency, M07 queues |
| 3 | **Edge & Global Scale** | Cloudflare (Global Edge Architecture) · Pinterest (Cache Infrastructure Scaling) · eBay (Distributed Listing) · Walmart (Autocomplete Backend Rebuild) | M02 CDN/load balancing, M05 caching, M03 partitioning |
| 4 | **Payments & Financial Reliability** | Stripe (Database Migration Platform) · PayPal (Kafka Scaling) · Razorpay (Reliable Dual Writes) · Coinbase (Solana Processing) · PhonePe (Distributed Job Scheduler) · Capital One (Resilient Systems) | M04 transactions/outbox, M06 idempotency & resilience, M07 queues |
| 5 | **Migration Journeys** | Shopify (Sharded Monolith) · DoorDash (Microservices Migration) · Zomato (Billing Platform Scaling) | M03 sharding, M07 monolith↔microservices, M06 fault tolerance |
| 6 | **Event-Driven Architectures** | AWS (Event-Driven Architecture) · Meta (Distributed Priority Queue) · Salesforce (Guaranteed Data Delivery) · Canva (Analytics Event Pipeline) · Etsy (Kafka Zonal Resiliency) | M07 queues/event-driven, M06 fault tolerance |
| 7 | **Platform Engineering & Internal Infrastructure** | Google (System Design Principles) · Microsoft (Platform Engineering Paths) · Atlassian (Cloud Engineering) · Expedia (Configuration Management Platform) · Adobe (Unified Search Architecture) | M02 API gateway, M07 service discovery, organizational design |
| 8 | **Consumer Apps at Scale** | Uber (Rider App Architecture) · Spotify (Backend Infrastructure) · Figma (Multi-Database Scaling) · Instacart (Multi-Database Scaling) | M03 database scaling, M05 caching, M02 request path |

Reading order note: Theme 1–4 map directly onto Parts II–III; read those first, then 5–8.

---

## 5. The 12-week schedule

| Week | Focus | Deliverable |
|---|---|---|
| 1 | M01 Scaling fundamentals | Estimation drill sheet + 50M-DAU messenger estimate |
| 2 | M02 Request path | Token-bucket rate limiter + full request-path diagram |
| 3 | M03 Database scaling & replication | Read replica setup notes + failover walkthrough |
| 4 | M03 Sharding & partitioning | Consistent-hashing KV store (lab) |
| 5 | M04 Consistency & transactions | Saga lab + consistency-model cheat sheet |
| 6 | M05 Caching | Cache-aside lab with hit-rate measurements + stampede fix |
| 7 | M06 Resilience | Circuit-breaker lab + payment-API resilience doc |
| 8 | M07 Queues & event-driven | Mini message queue with delivery guarantees |
| 9 | M07 Microservices & coordination | Monolith-vs-microservices decision memo + Raft reading |
| 10 | M08 Case sprint A | Teardowns: Themes 1–4 (18 cases) |
| 11 | M08 Case sprint B | Teardowns: Themes 5–8 (17 cases) |
| 12 | M09 Capstone + mock interviews | Capstone repo + two timed mock designs |

---

## 6. Core reading libraries (the 10 source repos)

| Library | Role in this course |
|---|---|
| System Design Academy | Core curriculum text for Parts I–IV |
| Developer Roadmaps | Track progress; place yourself on the backend map |
| Tech Interview Handbook | Interview layer for M09 |
| Coding Interview University | CS-fundamentals refresher behind every module |
| Build Your Own X | Source of every lab and the capstone |
| Engineering Leadership | Context for Part IV org-level trade-offs |
| Path to Senior Engineer Handbook | Leveling context: why senior engineers think in trade-offs |
| freeCodeCamp | Prerequisite refresh (networking, databases, APIs) |
| Public APIs | Real data sources for capstone projects |
| Free Programming Books | Deep reference reading (DDIA and friends) |

The original curated list circulates with shortened `lnkd.in` links; the full list is preserved verbatim in the appendix below.

---

## Appendix · Source links, verbatim from the material

**Learning libraries (10):**

1. System design academy — https://lnkd.in/eKATU6QV
2. Public APIs — https://lnkd.in/epWSyzqs
3. Tech interview handbook — https://lnkd.in/e7EjsJNF
4. Coding interview university — https://lnkd.in/evJSNCPE
5. Engineering leadership — https://lnkd.in/ePCzV3zF
6. Freecodecamp — https://lnkd.in/e_4pA8xV
7. Developer roadmaps — https://lnkd.in/e9MuB_Yg
8. Path to senior engineer handbook — https://lnkd.in/exkJCxVi
9. Free programming books — https://lnkd.in/eXAzAJ3M
10. Build your own x — https://lnkd.in/ekZQbTPz

**System design concepts (23):**

1. Load Balancing — https://lnkd.in/gH9rdjCx
2. CDN — https://lnkd.in/g83A7-rM
3. Caching — https://lnkd.in/gTjxhv2V
4. Cache Invalidation — https://lnkd.in/geC955AY
5. Rate Limiting — https://lnkd.in/gWqJzCNJ
6. API Gateway — https://lnkd.in/gBNKpecH
7. CAP Theorem — https://lnkd.in/g4yFYkEi
8. Sharding — https://lnkd.in/gFi23iNV
9. Replication — https://lnkd.in/gikkrmNp
10. Partitioning — https://lnkd.in/gQhJS8ii
11. Queues — https://lnkd.in/gPGiuxtu
12. Microservices — https://lnkd.in/gZfYV2Qu
13. Microservices Vs Monoliths — https://lnkd.in/gM-dKE3D
14. Fault Tolerance — https://lnkd.in/gdamMmtc
15. Database Scaling — https://lnkd.in/ghq4v_gQ
16. Service Discovery — https://lnkd.in/gjfbNVBe
17. Consistency models — https://lnkd.in/gGkMENA3
18. Eventual Consistency — https://lnkd.in/gdSn54SK
19. Distributed Transactions — https://lnkd.in/gTc8pSbH
20. Leader Election — https://lnkd.in/g-kwhzSb
21. Horizontal vs Vertical Scaling — https://lnkd.in/gW-Vi9Qt
22. Back of the Envelope Estimation — https://lnkd.in/gQ6vtM3U
23. Idempotency, Data Latency & Finale — https://lnkd.in/gapgNSgh

**Company architecture case studies (35):**

1. Google System Design Principles — https://lnkd.in/gPKFwSYD
2. Meta Distributed Priority Queue — https://lnkd.in/gBBr4Vjq
3. Microsoft Platform Engineering Paths — https://lnkd.in/gymvyfbd
4. Adobe Unified Search Architecture — https://lnkd.in/g_SNVuzE
5. Salesforce Guaranteed Data Delivery — https://lnkd.in/gxCVtXZu
6. AWS Event Driven Architecture — https://lnkd.in/gT463eWY
7. Netflix Distributed Tracing Infrastructure — https://lnkd.in/gESgRhWU
8. Uber Rider App Architecture — https://lnkd.in/gGbxqbpU
9. Airbnb Key Value Architecture — https://lnkd.in/gqTSagR4
10. Dropbox Magic Pocket Architecture — https://lnkd.in/gfdfk7FV
11. Pinterest Cache Infrastructure Scaling — https://lnkd.in/g3NZnxAm
12. Slack Cellular Architecture Migration — https://lnkd.in/gtnkNEzF
13. Spotify Backend Infrastructure Architecture — https://lnkd.in/gCebV4sR
14. Cloudflare Global Edge Architecture — https://lnkd.in/gfAq7JTH
15. Stripe Database Migration Platform — https://lnkd.in/gVvP7VWQ
16. Shopify Sharded Monolith Changes — https://lnkd.in/g_KZ-BA4
17. DoorDash Microservices Migration Journey — https://lnkd.in/gBC5E3g2
18. Discord Trillion Message Indexing — https://lnkd.in/gRkk_d9G
19. Twilio Exactly Once Delivery — https://lnkd.in/ga7Zfank
20. Datadog Husky Event Store — https://lnkd.in/gWcZgw3S
21. Atlassian Cloud Engineering Architecture — https://lnkd.in/gyaGJxHY
22. PayPal Kafka Scaling Architecture — https://lnkd.in/gBRT8R-P
23. eBay Distributed Listing Architecture — https://lnkd.in/gqUVQRiW
24. Walmart Autocomplete Backend Rebuild — https://lnkd.in/gbVKZT2p
25. Capital One Resilient Systems — https://lnkd.in/gSv385XX
26. Canva Analytics Event Pipeline — https://lnkd.in/gReCsAkW
27. Figma Multi Database Scaling — https://lnkd.in/gQqayzyk
28. Razorpay Reliable Dual Writes — https://lnkd.in/gRiV9ypn
29. PhonePe Distributed Job Scheduler — https://lnkd.in/gZ4ZVDkN
30. Zomato Billing Platform Scaling — https://lnkd.in/g9kcikQy
31. Coinbase Solana Processing Architecture — https://lnkd.in/gyYwXm8g
32. Etsy Kafka Zonal Resiliency — https://lnkd.in/gJcnfTer
33. Expedia Configuration Management Platform — https://lnkd.in/gkj5GerG
34. Instacart Multi Database Scaling — https://lnkd.in/gUfawb2B
35. GitHub Distributed Storage System — https://lnkd.in/gDAAq6RP
