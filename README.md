# Instagram-Style Photo Sharing System – System Design

System design project for an Instagram-like photo sharing platform completed for  
**CISC 3140 – Design & Implementation of Large-Scale Applications, Brooklyn College**.

> This repository contains the architecture, data model, and design report for the system
> (not a full production implementation).

---

## 1. Overview

The goal of this project is to design a large-scale, Instagram-style application that allows users to:

- Create accounts and manage profiles  
- Upload and view photos (and image captions)  
- Follow other users and see a personalized news feed  
- Like and comment on posts  
- Receive notifications about relevant activity  

The design focuses on **high availability**, **low latency**, and **horizontal scalability** for millions of users.

---

## 2. Problem Statement

Design the backend architecture and data model for a global photo sharing platform that:

- Handles high read/write throughput (news feed, uploads, likes, comments)  
- Minimizes feed latency for active users  
- Supports large media storage and efficient content delivery  
- Remains resilient to server failures and regional outages  

The scope of this project is **system design**, not a full deployment.

---

## 3. Core Features

**User-facing**

- User registration, login, and authentication  
- User profiles (bio, avatar, follower / following counts)  
- Photo upload with captions and timestamps  
- Home feed (posts from followed users)  
- Post interactions: likes, comments  
- Notifications for follows, likes, and comments  
- Basic search / discovery (user search, simple explore)

**Admin / internal**

- Basic moderation hooks for users and posts  
- Metrics and logging for observability

---

## 4. Non-Functional Requirements

- **Availability:** Service remains up despite individual server failures.  
- **Scalability:** Horizontal scaling to millions of daily active users.  
- **Latency:** P95 feed load under X ms (course-level assumption).  
- **Consistency:** Eventual consistency for feeds and counts is acceptable; strong consistency for auth and critical writes.  
- **Security & Privacy:** Authentication, authorization, and encryption in transit.

---

## 5. High-Level Architecture

The design follows a **microservices** approach:

- **API Gateway** – Single entry point for client requests, routing to internal services.  
- **User Service** – Manages user accounts, profiles, and authentication.  
- **Media Service** – Handles photo uploads, metadata, and integration with object storage/CDN.  
- **Social Graph Service** – Stores and queries follower/following relationships.  
- **Feed Service** – Generates and serves personalized news feeds (fan-out write / fan-out read hybrid).  
- **Engagement Service** – Manages likes and comments.  
- **Notification Service** – Sends in-app notifications for follows, likes, and comments.  
- **Background Workers & Message Queue** – Asynchronous processing (feed fan-out, notification delivery, counter updates).  
- **Cache Layer** – Redis or similar cache for hot feeds, user profiles, and counts.  
- **Databases** – Combination of relational DB for strong-consistency data and NoSQL / key-value stores for high-volume items.  
- **CDN & Object Storage** – Stores and delivers photos efficiently across regions.  

Architecture and sequence diagrams are included in the `/diagrams` and `/docs` directories.

---

## 6. Data Model Overview

Key entities (simplified):

- **User** – `user_id`, username, email, hashed_password, profile fields  
- **Post** – `post_id`, `user_id`, caption, media_url, created_at  
- **Follow** – follower_id, followee_id, created_at  
- **Like** – user_id, post_id, created_at  
- **Comment** – comment_id, post_id, user_id, text, created_at  
- **Notification** – notification_id, user_id, type, actor_id, entity_id, created_at, read_flag  

ER and schema diagrams are provided in the report.

---

## 7. Key Design Decisions

- **Microservices vs. Monolith:** Chose microservices to scale read-heavy components independently (feed, media) and enable clear service boundaries.  
- **Feed Generation:** Hybrid fan-out model – precompute feeds for active users while using on-demand queries for cold users.  
- **Storage:** Use relational DB for user/auth data; NoSQL / wide-column or key-value store for feeds, likes, and timelines.  
- **Caching:** Cache user profiles, timelines, and counters to reduce DB load and speed up feed retrieval.  
- **Messaging:** Introduced a message queue to decouple write paths (post creation) from expensive operations (fan-out, notifications).  

Trade-offs and alternatives (e.g., SQL vs NoSQL, consistency models) are discussed in detail in the report.

---

## 8. Scalability, Reliability & Security

- **Scalability:** Horizontal scaling of stateless services behind load balancers; partitioned databases and sharded social graph.  
- **Reliability:** Replication, health checks, and automatic failover for critical services; regular backups for persistent data.  
- **Observability:** Centralized logging, metrics, and alerting for latency and error monitoring.  
- **Security:**  
  - Authentication and authorization for all user actions  
  - HTTPS for all traffic  
  - Role-based access for admin endpoints  
