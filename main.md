# Backend Engineering Fundamentals: The Master Checklist

## Pillar 1: The Internet & Communication

The foundational rules of how data moves across the web.

- **HTTP/HTTPS Lifecycle**
  - **DNS Resolution:** How a domain name (like `google.com`) is translated into an IP address.
  - **TCP Handshake:** The 3-way handshake (SYN, SYN-ACK, ACK) that establishes a reliable connection.
  - **TLS/SSL Handshake:** How HTTPS encrypts data in transit to prevent packet sniffing.
  - **The Request/Response Cycle:** The anatomy of an HTTP request (Method, Headers, Body) and response (Status Code, Headers, Body).
- **RESTful API Design**
  - **HTTP Verbs:** Strict adherence to `GET` (read), `POST` (create), `PUT` (replace), `PATCH` (partial update), and `DELETE` (remove).
  - **Idempotency:** Understanding which methods can be called multiple times without changing the result beyond the initial application (e.g., `PUT`, `DELETE`, `GET`).
  - **Resource-Based URLs:** Designing endpoints around nouns, not verbs (e.g., `/users/123/orders` instead of `/getOrdersForUser?id=123`).
  - **Status Codes:** Mastery of the 5 categories (1xx Info, 2xx Success, 3xx Redirect, 4xx Client Error, 5xx Server Error).
- **Real-Time vs. Asynchronous Communication**
  - **Webhooks:** Server-to-server HTTP callbacks pushed when an event occurs (Event-Driven).
  - **Polling (Short & Long):** The client repeatedly asking the server if there is new data.
  - **WebSockets:** A persistent, bi-directional, full-duplex TCP connection for real-time apps (like chat).
- **Data Serialization**
  - **JSON parsing:** How native language objects (like Python dicts) are serialized into strings for transport.
  - **Data Validation:** Enforcing strict schemas on incoming JSON payloads (e.g., using Pydantic).
- **CORS (Cross-Origin Resource Sharing)**
  - **What it is:** A browser security mechanism that blocks frontend requests to a different domain/port unless the backend explicitly allows it.
  - **Why it matters:** Every Django + React/Next.js project hits this. The browser sends a preflight `OPTIONS` request before the actual request.
  - **How to handle it:** Using `django-cors-headers` middleware to whitelist frontend origins.
- **API Versioning**
  - **What it is:** Strategies for evolving your API without breaking existing clients.
  - **Common patterns:** URL versioning (`/api/v1/`), header versioning (`Accept: application/vnd.api+json;version=1`), and query param versioning (`?version=1`).
- **HTTP/2 and HTTP/3**
  - **HTTP/2:** Multiplexing (multiple requests over a single TCP connection), header compression, server push.
  - **HTTP/3:** Built on QUIC (UDP-based), eliminates TCP head-of-line blocking.

## Pillar 2: Databases & Data Management

How state is stored, retrieved, and structured safely.

- **Relational Databases (SQL)**
  - **ACID Properties:** Atomicity, Consistency, Isolation, Durability (guarantees that database transactions are processed reliably).
  - **Normalization:** Organizing data to reduce redundancy (1NF, 2NF, 3NF).
  - **Joins:** Deep understanding of `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL OUTER JOIN`.
  - **Transactions:** Grouping multiple SQL statements into a single unit of work (`COMMIT` or `ROLLBACK`).
- **NoSQL Databases**
  - **Key-Value Stores (Redis):** In-memory data structure stores, perfect for caching and queues.
  - **Document Stores (MongoDB):** Storing JSON-like documents, ideal for flexible, unstructured data.
  - **When to choose NoSQL over SQL:** Understanding the CAP Theorem (Consistency, Availability, Partition Tolerance).
- **Database Indexing**
  - **What it is:** A data structure (usually a B-Tree) that improves the speed of data retrieval at the cost of slower writes and more storage.
  - **Primary vs. Foreign Keys:** How tables relate to one another.
  - **Composite Indexes:** Indexing multiple columns together.
- **ORMs (Object-Relational Mapping)**
  - **Abstraction:** How Django ORM or SQLAlchemy convert Python classes into SQL tables.
  - **The N+1 Query Problem:** The most common ORM performance killer (accidentally querying the database inside a loop) and how to fix it (using `select_related` or `prefetch_related`).
- **Database Migrations**
  - **What it is:** Version control for your database schema. Django's `makemigrations` generates Python files that describe schema changes; `migrate` applies them.
  - **Why it matters:** Understanding migration dependencies, squashing migrations, and handling data migrations (not just schema migrations) is critical in production.
- **Connection Pooling**
  - **What it is:** Reusing database connections instead of opening/closing one per request.
  - **Why it matters:** Without pooling, a production Django app under load will exhaust database connections and crash.
  - **Tools:** `pgbouncer` (external) or Django 5.1+ built-in persistent connections (`CONN_MAX_AGE`, `CONN_HEALTH_CHECKS`).

## Pillar 3: Security & Identity

Defending the application and verifying users.

- **Authentication vs. Authorization**
  - **Authentication (AuthN):** "Are you who you say you are?" (Verifying identity).
  - **Authorization (AuthZ):** "Are you allowed to do this?" (Checking permissions/roles).
- **Session-Based Auth (Stateful)**
  - How the server creates a session, stores the ID in the database, and sets an `HttpOnly` cookie in the client's browser.
- **Token-Based Auth (Stateless / JWT)**
  - The anatomy of a JSON Web Token (Header, Payload, Signature).
  - Why JWTs do not require a database lookup to verify identity.
  - The difference between Access Tokens (short-lived) and Refresh Tokens (long-lived).
- **Cryptography Basics**
  - **Hashing (Bcrypt / Argon2):** One-way scrambling used exclusively for passwords.
  - **Encryption (AES / RSA):** Two-way scrambling requiring a key, used for sensitive data (like credit cards).
  - **Encoding (Base64):** Translating data for safe transport (NOT for security).
- **The Big 3 Vulnerabilities**
  - **XSS (Cross-Site Scripting):** Attackers injecting malicious JavaScript into the frontend. Prevented by escaping user input and using `HttpOnly` cookies.
  - **CSRF (Cross-Site Request Forgery):** Attackers tricking a user's browser into executing unwanted actions on another site. Prevented by anti-CSRF tokens.
  - **SQL Injection:** Attackers inputting SQL commands into form fields. Prevented by using parameterized queries (which ORMs do automatically).
- **OAuth 2.0 / OpenID Connect**
  - **OAuth 2.0:** An authorization framework that lets third-party apps access a user's resources without sharing passwords (e.g., "Login with Google").
  - **OpenID Connect (OIDC):** An authentication layer on top of OAuth 2.0 that verifies user identity.
  - **Flows:** Authorization Code Flow (most common for server-side apps), Client Credentials (machine-to-machine).
- **Rate Limiting & Throttling**
  - **What it is:** Restricting how many API requests a client can make in a given time window.
  - **Why it matters:** Prevents abuse, brute-force attacks, and protects server resources.
  - **In DRF:** Built-in throttle classes — `AnonRateThrottle`, `UserRateThrottle`, and custom throttles.

## Pillar 4: Performance & Scalability

Handling massive amounts of traffic without crashing.

- **Caching Strategies**
  - Storing frequently requested, computationally heavy data in memory (Redis/Memcached).
  - **Cache Invalidation:** The hardest problem in computer science (knowing exactly when to delete outdated cached data).
- **Asynchronous Task Queues**
  - Decoupling the main web server from heavy workloads.
  - The architecture of a message broker (Redis/RabbitMQ) and worker nodes (Celery).
- **Scaling Systems**
  - **Vertical Scaling (Scale-Up):** Adding more CPU/RAM to a single server (easy, but has a hard limit and creates a single point of failure).
  - **Horizontal Scaling (Scale-Out):** Adding more servers to the pool (requires a load balancer and stateless architecture).
- **Load Balancing**
  - How tools like Nginx or AWS ALB distribute incoming traffic evenly across multiple server instances.
- **Pagination**
  - **Offset Pagination:** `LIMIT 10 OFFSET 20` — simple but slow on large datasets (database still scans skipped rows).
  - **Cursor Pagination:** Uses a pointer to the last item seen — fast and scales well for infinite scroll and large datasets.

## Pillar 5: Deployment & Infrastructure

Getting code from your laptop to the public internet reliably.

- **Containerization (Docker)**
  - **Images vs. Containers:** An image is the blueprint (Dockerfile); a container is the running instance.
  - **Why we use it:** Eliminates the "it works on my machine" problem by packaging the OS, dependencies, and code into one isolated unit.
- **CI/CD (Continuous Integration / Continuous Deployment)**
  - **CI:** Automated testing and linting that runs every time code is pushed to GitHub.
  - **CD:** Automated deployment of that code to the production server if all tests pass.
- **Environment Variables & Secrets Management**
  - The concept of the `.env` file.
  - Why API keys, database URLs, and secret keys must _never_ be hardcoded or committed to version control.
- **Web Servers vs. App Servers**
  - **Reverse Proxies (Nginx):** Handles raw internet traffic, SSL termination, and static files.
  - **App Servers (Uvicorn / Gunicorn):** Translates the HTTP requests into Python code (WSGI/ASGI) that Django or FastAPI can understand.
- **Logging & Monitoring**
  - **Django Logging:** Configuring loggers, handlers, and formatters in `settings.py` to track application behavior.
  - **Error Tracking:** Tools like Sentry that catch production exceptions automatically and alert you before users report them.
- **Testing**
  - **Unit Tests:** Testing individual functions/methods in isolation to verify they return correct results.
  - **Integration Tests:** Testing how components work together — views, serializers, and the database as a connected system.
  - **Mocking:** Replacing external dependencies (third-party APIs, email services, task queues) with fakes during tests so tests run fast and don't depend on external services.
  - **Fixtures & Factories:** Creating reliable, repeatable test data (e.g., using `factory_boy` instead of manually creating objects in every test).
  - **Test Coverage:** Measuring what percentage of your codebase is exercised by tests (using `coverage.py`).
