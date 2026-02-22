# 🏗️ High-Level Design (HLD) Mastery

Welcome to the **System Design Lecture Series**. This repository is a curated guide to mastering practical, large-scale system design. We focus on reasoning through trade-offs, operational excellence, and the "why" behind architectural choices.

---

## 📖 Overview
This curriculum is designed to take you from basic client-server interactions to complex distributed systems. Whether you are preparing for a "Big Tech" interview or designing a production-grade service, these notes provide a repeatable framework for success.

### 🛠️ What’s Inside?
- **17 Detailed Lectures:** From fundamentals to deep-dive case studies.
- **Architectural Blueprints:** Diagrams illustrating patterns like Consistent Hashing, Pub/Sub, and Feed Generation.
- **Estimation Frameworks:** How to calculate QPS, storage, and bandwidth like a pro.

---

## 📅 Curriculum Roadmap

| Lecture | Topic | Highlights |
| :---: | :--- | :--- |
| 01 | [**Introduction to HLD**](./Lecture-1/README.md) | Networking basics, scaling 101, and estimation. |
| 02 | [**HTTP & Load Balancing**](./Lecture-2/README.md) | Tier separation, CDNs, and stateless architecture. |
| 03 | [**CAP & Microservices**](./Lecture-3/README.md) | Partition tolerance, monolith vs. microservices. |
| 04 | [**Sharding & Hashing**](./Lecture-4/README.md) | DB partitioning and Virtual Nodes. |
| 05 | [**Rate Limiting**](./Lecture-5/README.md) | Token/Leaky bucket and Sliding Window algorithms. |
| 06 | [**Case Study: URL Shortener**](./Lecture-6/README.md) | Base62 encoding, hashing, and redirect logic. |
| 07 | [**Notification Systems**](./Lecture-7/README.md) | Async delivery, retry strategies, and fan-out. |
| 08 | [**Messaging Basics**](./Lecture-8/README.md) | Producer-Consumer patterns and queueing. |
| 09 | [**Kafka & RabbitMQ**](./Lecture-9/README.md) | Topics, partitions, offsets, and DLQs. |
| 10 | [**Authentication & AuthZ**](./Lecture-10/README.md) | JWT vs. Sessions, Salt/Pepper, and RBAC. |
| 11 | [**OAuth & Autocomplete**](./Lecture-11/README.md) | OAuth2 flows and Trie-based search. |
| 12 | [**Case Study: News Feed**](./Lecture-12/README.md) | Instagram-style feed, Push vs. Pull models. |
| 13 | [**Case Study: YouTube**](./Lecture-13/README.md) | Video codecs, chunking, and manifests. |
| 14 | [**NoSQL & Replication**](./Lecture-14/README.md) | DynamoDB/Cassandra, Quorum, and replication. |
| 15 | [**RDBMS Internals**](./Lecture-15/README.md) | Indexing, B+ Trees, and page/block storage. |
| 16 | [**Collaborative Editing**](./Lecture-16/README.md) | WebSockets, OT/CRDT concepts, and real-time. |
| 17 | [**Chat & API Gateways**](./Lecture-17/README.md) | WhatsApp design and Service Discovery. |

---

## 🧠 Core Concepts & Skills

### 📂 Architectures & Scaling
* **Scaling:** Vertical vs. Horizontal, Auto-scaling, Stateless vs. Stateful.
* **Infrastructure:** Load Balancers , DNS/Geo-DNS, CDNs.
* **Services:** API Gateways, Service Discovery, Idempotency.

### 💾 Data Storage & Partitioning
* **Databases:** Relational (B+ Trees) vs. NoSQL (LSM Trees), Sharding, Consistent Hashing.
* **Consistency:** CAP Theorem, Strong vs. Eventual Consistency, Quorums (N, R, W).
* **Caching:** Cache-aside, Write-through, Write-back, LRU/LFU policies.

### ⚡ Messaging & Performance
* **Async Patterns:** Message Queues (Kafka, RabbitMQ), Pub/Sub, DLQs.
* **Ops:** Rate limiting (Token Bucket, Leaky Bucket).

---

## 💡 Study Tips for Success

> [!TIP]
> **Quantify Everything:** Never just say "we need a cache." Say "With 100M DAU and a 10% read-heavy load, we need a cache to handle 100k QPS."

- **The Rule of Three:** Every design choice has a downside. If you can't explain the "cons" of your choice, you don't understand the trade-off yet.
- **Start Broad:** Draw high-level boxes (Client → LB → Web → DB) before diving into specific database engines or algorithms.

---

## 🚀 Getting Started

Ready to dive in? Start with the fundamentals:

👉 **[Go to Lecture 1: Introduction to HLD →](./Lecture-1/README.md)**

---
*Happy learning! If you find this helpful, feel free to ⭐ the repo.*