00 — Backend Foundations and Networking

Understand how a backend works before building one.

1. Backend Architecture and Request Lifecycle
2. First-Principles Thinking and Technical Trade-offs
3. Networking Fundamentals — DNS, TCP vs. UDP, connection establishment, and client-server communication
4. HTTP and HTTPS — request/response lifecycle, methods, headers, and status codes
5. TLS and Connection Management — TLS handshakes, connection reuse, and persistent connections
6. HTTP/2 and HTTP/3 — multiplexing, QUIC, and head-of-line blocking
7. Concurrency and Parallelism
8. I/O-Bound vs. CPU-Bound Work

Priority: Learn the fundamentals of HTTP and the request lifecycle first. Study the lower-level networking details as needed.

01 — API Engineering and Application Architecture

Learn how to structure your backend and expose functionality to your mobile and web clients.

1. API Architecture and Request Handling
2. Separation of Concerns — controllers, services, repositories, and middleware
3. REST API Design — resources, HTTP methods, idempotency, and status codes
4. Data Serialization, Validation, and Transformation
5. OpenAPI: The Universal Contract Between Clients and Servers
6. API Versioning and Compatibility
7. Error Handling and Consistent API Responses
8. Authentication and Authorization Fundamentals
9. Session-Based and Token-Based Authentication
10. OAuth 2.0 and OpenID Connect
11. Input Validation and Trust Boundaries

Priority: High. You need these concepts to build a maintainable backend that your mobile application can communicate with reliably.

02 — Database Engineering and Data Integrity

Learn how to design, query, and maintain your application’s data.

1. Database Fundamentals and Relational Data Modeling
2. Relationships, Constraints, and Normalization
3. Business Logic and Data Integrity — business invariants, atomic operations, and race conditions
4. Transactions and ACID Properties
5. Transaction Isolation and Concurrency Control
6. Database Internals and Query Execution
7. Indexing Strategies and Trade-offs
8. Database Performance and Diagnosis — query plans, inefficient queries, and N+1 problems
9. ORMs and Database Abstraction — benefits, limitations, and generated queries
10. Database Schema Evolution and Migrations
11. Connection Pooling and Connection Management
12. Database Triggers — automatic database actions, use cases, and trade-offs
13. Pagination and Efficient Data Retrieval — offset vs. cursor pagination and stable ordering

Priority: Very high. Your database design affects listings, messaging, reviews, favorites, and almost every other feature in 3sha.

03 — Authentication, Security, and Access Control

Learn how to protect users and their data.

1. Authentication and Authorization — roles, permissions, and resource-level access control
2. OAuth 2.0 and OpenID Connect
3. Session-Based and Token-Based Authentication
4. Cryptography and Sensitive Data — password hashing, encryption, and secret protection
5. Common Web/API Vulnerabilities — XSS, CSRF, SQL injection, broken access control, and IDOR
6. API Security and Abuse Prevention — rate limiting, brute-force protection, and request-size limits
7. Network Security and Firewalls
8. Security Fundamentals and Threat Modeling

Priority: High, starting with authentication, authorization, and access control.

You don’t need to master every security topic before launching. However, you must correctly implement authentication, authorization, input validation, and protection against unauthorized access before handling real users.

04 — Production Readiness and Reliability

Learn how to make your backend dependable outside your development environment.

1. Production Configuration and Secrets Management
2. Environment Variables and Configuration Validation
3. Logging, Monitoring, and Observability
4. Error Tracking and Alerting
5. Error Handling and Fault Tolerance
6. Timeouts and Retries
7. Graceful Shutdown
8. Health Checks and Production Readiness
9. Production Incident Debugging
10. The Twelve-Factor App

Priority: High before production launch.

You should be able to determine why a request failed, investigate a production error, configure your application safely, and deploy updates without unnecessarily disrupting users.

05 — Testing and Quality Assurance

Learn how to verify that your application behaves correctly and remains stable as you add features.

1. Testing Strategies — unit, integration, and end-to-end testing
2. Test Doubles — mocks, stubs, fakes, and fixtures
3. Test Data Management
4. Test Coverage and Test-Driven Development
5. API Testing and Contract Testing
6. Database Testing and Transaction Behavior
7. Authentication and Authorization Testing

Priority: High, but learn testing alongside implementation rather than waiting until the application is complete.

For your MVP, integration tests covering critical API endpoints, database behavior, and access control are particularly valuable.

06 — Deployment, Infrastructure, and DevOps

Learn how to deploy and maintain the application.

1. Web Servers, Application Servers, and Reverse Proxies
2. Containerization and Reproducible Environments — Docker
3. CI/CD and Deployment Automation
4. DNS, Domains, HTTPS, and TLS Termination
5. Application Hosting and Infrastructure Fundamentals
6. Database Backups and Recovery
7. Database Migrations in Production
8. Deployment Strategies and Rollbacks
9. DevOps and Infrastructure Fundamentals

Priority: High before launch.

You don’t need an elaborate infrastructure setup. Focus on deploying your application reliably, securing its configuration, backing up important data, and recovering from failures.

07 — Performance, Caching, and Asynchronous Processing

Learn how to improve performance and handle work that shouldn’t block a user’s request.

1. Latency, Throughput, and Performance Fundamentals
2. Performance Profiling and Bottleneck Diagnosis
3. Database Query Optimization
4. Caching Fundamentals and Invalidation
5. Background Jobs and Task Queues
6. Messaging and Event-Driven Systems
7. Reliable Asynchronous Processing — idempotency, retries, and duplicate delivery
8. Concurrency Limits and Backpressure
9. Storage Systems and Storage Trade-offs — relational databases, key-value stores, and object storage
10. Object Storage and Media Delivery
11. Scalability Fundamentals — vertical vs. horizontal scaling

Priority: Learn the fundamentals early, but implement additional infrastructure when a real requirement justifies it.

For example, uploading listing images is an MVP requirement, but introducing Redis and a distributed task queue may not be. Don’t introduce them simply because they’re common in production systems.

08 — Advanced Backend Engineering and Distributed Systems

Build deeper knowledge after the MVP’s core functionality and production foundations are in place.

1. Distributed Systems Concepts
2. Consistency and Availability
3. CAP Theorem
4. Replication and Data Consistency
5. Partitioning and Sharding
6. Distributed Communication and Coordination
7. Redundancy and High Availability
8. Load Balancing and Traffic Distribution
9. Capacity Planning and Resource Management
10. Advanced Concurrency and Parallelism
11. Scalability Patterns and Architectural Trade-offs
12. Full-Text Search Using Elasticsearch for Blazingly Fast Search

Priority: Mostly post-MVP, except when a specific requirement demands earlier study.

PostgreSQL may be sufficient for your initial search functionality. You don’t need Elasticsearch until your search requirements justify operating an additional system.

09 — System Design in Depth

Keep this as a title for now. Add its detailed topics when you’re ready to begin studying system design.

⸻

Resources: https://www.youtube.com/watch?v=IrmPU7HRK0o&t=168s https://www.youtube.com/watch?v=Ru54dxzCyD0&t=3s https://github.com/joesef1/bulletproof-backend/blob/master/main.md
