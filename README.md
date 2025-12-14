# Designing Instagram — System Design (CISC 3140)

This repository contains a **system design report** for an Instagram-like social media platform focused on **massive scale**, **low-latency user experience**, and **high availability + durability** for media content. :contentReference[oaicite:0]{index=0

## What this design covers

Instagram-style capabilities including:
- Photo/video upload with transcoding, metadata, and efficient direct-to-object-storage uploads (pre-signed URLs). :contentReference[oaicite:1]{index=1}
- Follow/unfollow, likes, comments, shares, and notifications. :contentReference[oaicite:2]{index=2}
- Personalized infinite-scrolling feed with ranking + cursor pagination. :contentReference[oaicite:3]{index=3}
- 24-hour Stories with edge caching (optional highlights). :contentReference[oaicite:4]{index=4}
- Search & discovery using an index (e.g., Elasticsearch) + ML signals. :contentReference[oaicite:5]{index=5}

## Non-functional goals (targets)

- **Latency:** p95 ~200ms for feed & primary UI interactions; fast media start via CDN + adaptive streaming. :contentReference[oaicite:6]{index=6}  
- **Availability:** target 99.99% uptime with multi-AZ/region failover and clear RTO/backup policies. :contentReference[oaicite:7]{index=7}  
- **Durability:** highly durable media storage with cross-region replication (object store). :contentReference[oaicite:8]{index=8}  
- **Consistency model:** eventual consistency is acceptable for feeds/recommendations; strong consistency is required for auth (and other critical data). :contentReference[oaicite:9]{index=9}  
- **Security & privacy:** TLS in transit, encryption at rest, least-privilege IAM, secret rotation, DSAR support. :contentReference[oaicite:10]{index=10}  

## Architecture (high-level)

The design selects a **microservices + event-driven** architecture (Kafka) to scale services independently and improve fault isolation. :contentReference[oaicite:11]{index=11}

Core components:
- **API Gateway:** token validation, rate limiting, routing, request shaping, versioning. :contentReference[oaicite:12]{index=12}  
- **Auth Service:** login, token issuance/refresh, MFA, session management. :contentReference[oaicite:13]{index=13}  
- **Read Services:** low-latency feed/profile/search using cache + timeline DB. :contentReference[oaicite:14]{index=14}  
- **Write Services:** posts/likes/comments/follows; publish events to message bus. :contentReference[oaicite:15]{index=15}  
- **Kafka Message Bus:** durable event stream for feed generation, notifications, analytics. :contentReference[oaicite:16]{index=16}  
- **Timeline DB (Cassandra):** per-user feed entries optimized for appends/range scans. :contentReference[oaicite:17]{index=17}  
- **Cache (Redis):** hot cache for feed fragments/sessions/top posts. :contentReference[oaicite:18]{index=18}  
- **Media Processor:** transcoding, thumbnails, metadata extraction. :contentReference[oaicite:19]{index=19}  
- **Blob Storage + CDN:** durable media storage + global edge delivery. :contentReference[oaicite:20]{index=20}  
- **Monitoring/Observability:** metrics/tracing/logging + SLO dashboards. :contentReference[oaicite:21]{index=21}  

## Data model (core entities)

Primary entities: **User, Post, Comment, Media, Story, Notification** and follow relationships. :contentReference[oaicite:22]{index=22}  
Storage strategy uses a **hybrid model (SQL + NoSQL)**: SQL for structured interactions (User/Post/Comment), and NoSQL for high-volume evolving data (Stories/feeds). :contentReference[oaicite:23]{index=23}

## Communication & security notes

- **REST APIs** for core operations; JSON for requests/responses. :contentReference[oaicite:24]{index=24}  
- **Push notifications** via APNs/FCM triggered from Kafka consumers. :contentReference[oaicite:25]{index=25}  
- **Security:** HTTPS/TLS, JWT-based stateless sessions, Argon2/bcrypt-style hashing + salting, plus rate limiting/behavior checks.
