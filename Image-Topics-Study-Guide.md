# Image Topics Study Guide

This guide organizes the topics from the image into the current IT folder system. Each topic folder contains a `Theory` folder for concepts, explanations, diagrams, notes, tradeoffs, and interview-style understanding, plus a `Practical/Code` folder for hands-on scripts, examples, configs, commands, demos, and exercises.

The image text appears to say `SOS`, but because it is listed next to AWS topics like `S3`, `Lambda`, and `DynamoDB`, this structure treats it as `SQS`.

## Top-Level Structure

- `Backend`: application servers, APIs, databases, messaging, performance, reliability, and backend architecture.
- `DevOps`: containers, cloud, AWS, CI/CD, deployment workflow, infrastructure, security, and networking operations.
- `Data`: data-oriented systems, ML frameworks, and cloud databases.
- `Tools`: development tools such as IDEs.
- `FrontEnd`: existing folder kept untouched because the image topics are mostly backend, cloud, and infrastructure.

AWS now has its own detailed section at `DevOps/AWS`, including Route 53, VPC management, EKS, CloudFormation templates, EC2, IAM, CloudWatch, S3, SQS, Lambda, DynamoDB, and other AWS platform services.

## Backend

### APIs

#### WebSockets

Folder: `Backend/APIs/WebSockets`

Theory: WebSockets are persistent, full-duplex connections between client and server. Unlike normal HTTP request-response traffic, either side can send messages at any time after the handshake. They are useful for chat, live notifications, collaborative editing, dashboards, multiplayer state, and streaming updates. Important theory topics include connection lifecycle, ping/pong heartbeats, reconnection strategy, message ordering, backpressure, authentication, horizontal scaling, and how WebSockets interact with load balancers.

Practical: Build a small chat or live counter service. Add reconnect behavior, heartbeat checks, authentication during the upgrade request, and a Redis pub/sub adapter so multiple server instances can broadcast messages.

#### RPC

Folder: `Backend/APIs/RPC`

Theory: RPC, or Remote Procedure Call, models network communication as calling a function on another service. Common forms include gRPC, JSON-RPC, XML-RPC, and internal service clients. RPC works well for service-to-service communication when contracts are clear and latency matters. Learn request schemas, generated clients, deadlines, retries, idempotency, error mapping, versioning, streaming RPC, and how RPC differs from REST.

Practical: Create a small gRPC service with one unary call and one streaming call. Add request validation, deadlines, structured errors, and a client script that retries only safe operations.

#### Long-Short-Polling

Folder: `Backend/APIs/Long-Short-Polling`

Theory: Short polling repeatedly asks the server for updates at a fixed interval. Long polling keeps the request open until data is available or a timeout occurs. Polling is simpler than WebSockets and can work through restrictive proxies, but it can waste resources if the polling interval is too aggressive. Learn timeout behavior, connection limits, duplicate delivery, cache headers, retry intervals, and when polling is still the right choice.

Practical: Implement a notification endpoint using short polling, then convert it to long polling. Measure request count, average latency, and server resource usage.

### Runtime Servers

#### Servers

Folder: `Backend/Runtime-Servers/Servers`

Theory: A server receives requests, runs application logic, manages resources, and returns responses. Backend server design includes process models, worker pools, threading, async I/O, request routing, middleware, memory limits, connection pools, graceful shutdown, health checks, and observability. A production server must handle correctness, latency, throughput, failures, and operational visibility.

Practical: Build a simple HTTP API with health checks, graceful shutdown, request logging, and database connection pooling. Load test it and observe CPU, memory, and latency.

#### Proxy

Folder: `Backend/Runtime-Servers/Proxy`

Theory: A proxy sits between clients and servers or between internal services. It can forward traffic, terminate TLS, route requests, cache responses, enforce authentication, rate limit, rewrite headers, and hide internal network details. Reverse proxies such as Nginx, Envoy, HAProxy, and cloud API gateways are common in production systems.

Practical: Configure a reverse proxy in front of a local API. Add path-based routing, TLS termination if possible, request logging, and a rule that forwards `/api` to one backend and `/static` to another.

### Messaging

#### Kafka-RabbitMQ

Folder: `Backend/Messaging/Kafka-RabbitMQ`

Theory: Kafka and RabbitMQ both move messages between systems, but their mental models differ. Kafka is an append-only distributed log optimized for high-throughput event streams, replay, partitions, and consumer groups. RabbitMQ is a broker optimized for routing, queues, acknowledgements, exchanges, and flexible delivery patterns. Learn producers, consumers, topics, queues, partitions, ordering, acknowledgements, retries, dead-letter queues, idempotent consumers, and exactly-once claims.

Practical: Build an order event flow. Publish events, consume them in a worker, retry failed messages, and send poison messages to a dead-letter queue. Compare Kafka partition ordering with RabbitMQ routing.

### Data Storage

#### Database

Folder: `Backend/Data-Storage/Database`

Theory: Databases persist, query, and protect application data. Core theory includes relational modeling, normalization, indexes, transactions, isolation levels, locks, query planning, replication, backups, schema migrations, and tradeoffs between SQL, document, key-value, graph, and time-series systems. A good backend engineer understands both data modeling and operational behavior under load.

Practical: Design a schema for users, orders, and payments. Add indexes, write queries, inspect query plans, create migrations, and test transactional behavior during concurrent updates.

#### Embedded-Database

Folder: `Backend/Data-Storage/Embedded-Database`

Theory: An embedded database runs inside or beside the application instead of as a separate database server. SQLite, DuckDB, RocksDB, LevelDB, and LMDB are common examples. They are useful for local apps, edge systems, tests, analytics, caches, and single-node services. Learn file locking, durability settings, write concurrency, backups, migration strategy, and when embedded storage is inappropriate.

Practical: Build a small app backed by SQLite. Add migrations, indexes, transactions, and a backup command. Compare it with a client-server database for concurrent writes.

#### Sharding

Folder: `Backend/Data-Storage/Sharding`

Theory: Sharding splits data across multiple machines or logical partitions so one database does not carry all data or traffic. Common shard keys include user ID, tenant ID, region, or hash of an entity ID. The hard parts are choosing a good shard key, avoiding hot shards, resharding, cross-shard queries, cross-shard transactions, and operational tooling.

Practical: Simulate sharding users by hash across multiple local databases. Add routing logic, then test what happens when one shard receives disproportionate traffic.

#### Partitioning

Folder: `Backend/Data-Storage/Partitioning`

Theory: Partitioning splits large tables or datasets into smaller pieces, often inside the same database system. It can improve manageability and query performance when queries filter by partition key, such as time, tenant, or region. Learn range, list, hash, and composite partitioning, plus pruning, retention, archive strategy, and partition maintenance.

Practical: Create a time-partitioned events table. Insert data across months, compare query plans with and without partition filters, and practice dropping old partitions.

### Performance Reliability

#### Optimisation

Folder: `Backend/Performance-Reliability/Optimisation`

Theory: Optimization means improving performance, cost, reliability, or developer efficiency without breaking correctness. Start with measurement, not guesses. Learn profiling, latency budgets, bottlenecks, algorithmic complexity, database query tuning, caching, batching, concurrency, memory pressure, and tradeoffs between speed, complexity, and maintainability.

Practical: Profile a slow endpoint. Identify whether time is spent in CPU, database, network, serialization, or external services. Apply one improvement at a time and measure before and after.

#### Rate-Limiting

Folder: `Backend/Performance-Reliability/Rate-Limiting`

Theory: Rate limiting controls how often a user, client, IP, token, or service can perform an action. It protects systems from abuse, accidental traffic spikes, brute force attempts, and noisy clients. Common algorithms include fixed window, sliding window, token bucket, and leaky bucket. Learn distributed counters, fairness, burst handling, headers, retry-after behavior, and user experience.

Practical: Implement token bucket rate limiting for an API. Store counters in memory first, then Redis. Return standard rate limit headers and test burst behavior.

#### Error-Logging

Folder: `Backend/Performance-Reliability/Error-Logging`

Theory: Error logging records failures in a way that helps humans debug production issues. Useful logs are structured, searchable, correlated, and privacy-aware. Learn log levels, stack traces, request IDs, trace IDs, redaction, sampling, alerting, error grouping, and the difference between logs, metrics, and traces.

Practical: Add structured JSON logging to an API. Include request ID, user ID where safe, route, status code, duration, and error stack. Send unexpected errors to an error tracking tool or local collector.

#### QPS

Folder: `Backend/Performance-Reliability/QPS`

Theory: QPS means queries per second or requests per second, depending on context. It measures traffic rate, not latency. A service with high QPS still needs acceptable latency, error rate, and resource usage. Learn average versus peak QPS, capacity planning, load testing, saturation, concurrency, percentiles, and how QPS relates to throughput.

Practical: Load test a simple endpoint at increasing QPS. Record p50, p95, and p99 latency, error rate, CPU, memory, and database load.

#### Load-Balancer

Folder: `Backend/Performance-Reliability/Load-Balancer`

Theory: A load balancer distributes traffic across multiple backend instances. It improves availability, scaling, and operational flexibility. Learn layer 4 versus layer 7 balancing, health checks, round robin, least connections, weighted routing, sticky sessions, TLS termination, connection draining, and failure behavior.

Practical: Run two API instances and place Nginx, HAProxy, or a cloud load balancer in front. Add health checks and observe traffic distribution when one instance fails.

#### Caching

Folder: `Backend/Performance-Reliability/Caching`

Theory: Caching stores reusable data closer to where it is needed. It reduces latency and load, but introduces consistency problems. Learn cache-aside, write-through, write-behind, TTLs, invalidation, stampede protection, negative caching, HTTP caching, CDN caching, Redis, local memory caches, and when not to cache.

Practical: Add Redis caching to a slow read endpoint. Use TTLs, cache keys, invalidation on writes, and a lock or single-flight mechanism to prevent stampedes.

#### Availability

Folder: `Backend/Performance-Reliability/Availability`

Theory: Availability is the percentage of time a system is usable. It depends on redundancy, failover, graceful degradation, deployment safety, dependency management, monitoring, and incident response. Learn SLAs, SLOs, SLIs, error budgets, multi-zone design, retries, circuit breakers, bulkheads, and single points of failure.

Practical: Draw the dependency graph for a small service and identify single points of failure. Add health checks, timeouts, fallback behavior, and a basic uptime monitor.

#### Throughput

Folder: `Backend/Performance-Reliability/Throughput`

Theory: Throughput is the amount of work completed per unit of time, such as requests, messages, jobs, or bytes. It is related to QPS but broader. A system can increase throughput through batching, parallelism, async processing, efficient I/O, better queries, horizontal scaling, and reducing contention. Throughput must be balanced against latency and correctness.

Practical: Build a worker that processes queued jobs. Measure jobs per second, then test batching, multiple workers, and database write batching.

## DevOps

### Containers Orchestration

#### Kubernetes

Folder: `DevOps/Containers-Orchestration/Kubernetes`

Theory: Kubernetes orchestrates containers across a cluster. It schedules pods, maintains desired state, exposes services, handles rolling updates, manages configuration, and integrates with storage and networking. Learn pods, deployments, replicasets, services, ingress, config maps, secrets, namespaces, probes, resource requests and limits, autoscaling, and cluster observability.

Practical: Containerize an API and deploy it to a local Kubernetes cluster. Add readiness and liveness probes, environment config, a service, an ingress, and a rolling update.

#### Docker

Folder: `DevOps/Containers-Orchestration/Docker`

Theory: Docker packages applications with their runtime dependencies into images that run as containers. Core topics include images, layers, Dockerfiles, build context, volumes, networks, environment variables, multi-stage builds, registries, and container lifecycle. Docker is about packaging and running containers; Kubernetes is about orchestrating them.

Practical: Write a multi-stage Dockerfile for a backend service. Add a `docker-compose.yml` with an app, database, and Redis. Practice logs, exec, networks, volumes, and rebuilds.

#### Containerisation

Folder: `DevOps/Containers-Orchestration/Containerisation`

Theory: Containerisation packages an application and its dependencies into an isolated runtime unit. It is broader than Docker: Docker is one common tool, while containerisation is the general operating-system-level virtualization concept. Learn images, containers, namespaces, cgroups, isolation limits, immutable builds, environment parity, image registries, dependency pinning, and security scanning.

Practical: Take one app and make it run consistently on a clean machine using a container image. Keep the image small, avoid storing secrets in the image, run as a non-root user, and document the build and run commands.

### Environments Release

#### Staging

Folder: `DevOps/Environments-Release/Staging`

Theory: Staging is a pre-production environment used to test releases in conditions close to production. It catches deployment, integration, configuration, migration, and environment-specific issues. Learn environment parity, seed data, feature flags, secrets separation, smoke tests, test accounts, and why staging should be similar to production but not contain production secrets.

Practical: Define a staging deployment checklist. Add smoke tests that verify health, database connectivity, authentication, and one critical user flow after deployment.

#### Deployments

Folder: `DevOps/Environments-Release/Deployments`

Theory: Deployment is the process of safely releasing code or infrastructure changes. Strategies include rolling, blue-green, canary, recreate, feature flags, and progressive delivery. Good deployments are repeatable, observable, reversible, and automated. Learn migrations, rollback plans, config changes, health checks, and deployment approvals.

Practical: Create a deployment pipeline for a simple app. Add build, test, image publish, migration, deployment, smoke test, and rollback steps.

#### Cherry-Pick

Folder: `DevOps/Environments-Release/Cherry-Pick`

Theory: Cherry-picking applies a specific Git commit from one branch onto another. It is useful for hotfixes or selective backports, but it can create duplicate commits and conflict risk if overused. Learn commit hashes, conflict resolution, release branches, hotfix branches, and when merge or rebase is more appropriate.

Practical: Create a test repository with main, release, and feature branches. Cherry-pick one fix into the release branch, resolve a conflict, and document the release note.

#### Git-GitHub

Folder: `DevOps/Environments-Release/Git-GitHub`

Theory: Git is the version control system; GitHub is a platform built around Git with collaboration features. Learn commits, branches, merges, rebases, remotes, pull requests, code review, issues, actions, tags, releases, protected branches, and signed commits. The goal is traceable, reviewable, reversible change.

Practical: Practice branch workflows, pull requests, reviews, resolving merge conflicts, tagging releases, and setting up a GitHub Actions workflow.

### CI CD

#### CI-CD

Folder: `DevOps/CI-CD/CI-CD`

Theory: CI/CD automates integration, testing, packaging, and delivery. Continuous Integration validates changes frequently. Continuous Delivery keeps software releasable. Continuous Deployment releases automatically after checks pass. Learn pipelines, artifacts, test stages, environment promotion, secret management, caching dependencies, approvals, and rollback.

Practical: Build a pipeline that runs linting, unit tests, integration tests, Docker image build, vulnerability scan, and deployment to staging.

### Cloud

#### Cloud

Folder: `DevOps/Cloud/Cloud`

Theory: Cloud computing provides compute, storage, networking, databases, identity, observability, and managed services on demand. Learn regions, availability zones, IAM, VPCs, compute options, managed databases, object storage, queues, serverless, cost management, monitoring, and shared responsibility.

Practical: Create a small cloud architecture diagram for a web API using load balancing, compute, database, object storage, queue, logs, and alerts. Estimate basic cost drivers.

#### S3

Folder: `DevOps/Cloud/S3`

Theory: S3 is AWS object storage. It stores objects in buckets and is commonly used for files, backups, static assets, logs, data lakes, and artifacts. Learn buckets, objects, keys, versioning, lifecycle rules, encryption, IAM policies, pre-signed URLs, public access controls, storage classes, and event notifications.

Practical: Upload and retrieve files, generate pre-signed URLs, enable versioning, add lifecycle expiration, and configure least-privilege IAM access.

#### SQS

Folder: `DevOps/Cloud/SQS`

Theory: SQS is AWS Simple Queue Service, a managed queue for decoupling producers and consumers. It supports standard queues for high throughput and FIFO queues for ordered processing. Learn visibility timeout, message retention, dead-letter queues, long polling, batching, idempotent consumers, and retry behavior.

Practical: Build a producer that sends jobs to SQS and a worker that processes them. Add dead-letter queue handling and test what happens when processing fails.

#### Lambda

Folder: `DevOps/Cloud/Lambda`

Theory: Lambda is AWS serverless compute. You upload code, configure an event source, and AWS runs the function on demand. Learn cold starts, memory and timeout settings, IAM roles, event sources, concurrency, retries, idempotency, logging, local testing, and cost model.

Practical: Create a Lambda that processes S3 upload events or SQS messages. Add structured logs, environment variables, and error handling.

### Security Networking

#### Encryption

Folder: `DevOps/Security-Networking/Encryption`

Theory: Encryption protects data by making it unreadable without the correct key. Data can be encrypted in transit with TLS and at rest with disk, database, object storage, or application-level encryption. Learn symmetric versus asymmetric encryption, hashing versus encryption, key management, certificates, TLS, rotation, envelope encryption, and secret storage.

Practical: Configure HTTPS locally, hash passwords with a modern password hashing function, encrypt a sample payload, and document where keys should be stored.

#### Firewall

Folder: `DevOps/Security-Networking/Firewall`

Theory: A firewall controls network traffic based on rules. It can allow or deny traffic by IP, port, protocol, direction, or application behavior. Learn host firewalls, cloud security groups, network ACLs, web application firewalls, default deny posture, ingress, egress, and rule auditing.

Practical: Define firewall rules for a web app: allow public HTTPS, restrict admin access, allow database only from app servers, and block unnecessary outbound traffic.

#### FTP

Folder: `DevOps/Security-Networking/FTP`

Theory: FTP is an older file transfer protocol. Plain FTP sends credentials and data without encryption, so SFTP or FTPS is usually preferred. Learn active versus passive mode, ports, authentication, file permissions, secure alternatives, automation risks, and when managed object storage is better.

Practical: Compare FTP, FTPS, and SFTP. Set up a local SFTP transfer and script an upload/download flow using keys rather than passwords.

## Data

### Machine Learning

#### TensorFlow

Folder: `Data/Machine-Learning/TensorFlow`

Theory: TensorFlow is a machine learning framework for building, training, and deploying models. It uses tensors, computational graphs, automatic differentiation, layers, optimizers, datasets, and model serving. Learn supervised learning basics, loss functions, training loops, validation, overfitting, saved models, GPU acceleration, and deployment options.

Practical: Train a simple classifier, save the model, reload it for inference, and expose a prediction endpoint. Track accuracy, loss, and inference latency.

### Cloud Databases

#### DynamoDB

Folder: `Data/Cloud-Databases/DynamoDB`

Theory: DynamoDB is AWS managed NoSQL key-value and document storage. It is designed for predictable high-scale access patterns, not ad hoc relational querying. Learn partition keys, sort keys, single-table design, global secondary indexes, local secondary indexes, capacity modes, hot partitions, TTL, streams, conditional writes, and transactions.

Practical: Model an ecommerce access pattern with users, orders, and order items. Design keys for each query, add a GSI, and test conditional writes.

## Tools

### IDEs

#### PyCharm

Folder: `Tools/IDEs/PyCharm`

Theory: PyCharm is a Python IDE focused on code navigation, refactoring, debugging, testing, virtual environments, dependency management, and framework support. Learn interpreter configuration, project structure, run configurations, breakpoints, inspections, formatting, test runners, Git integration, and remote interpreters.

Practical: Create a Python project with a virtual environment, run tests, debug a failing function, configure formatting, and use Git tools from the IDE.

## Suggested Learning Order

1. Start with `Git-GitHub`, `Docker`, `Servers`, and `Database`.
2. Move to `REST-like API thinking`, `WebSockets`, `RPC`, `Polling`, and `Error-Logging`.
3. Add `Caching`, `Rate-Limiting`, `QPS`, `Throughput`, `Load-Balancer`, and `Availability`.
4. Learn deployment flow with `Staging`, `CI-CD`, `Deployments`, and `Kubernetes`.
5. Add cloud building blocks: `Cloud`, `S3`, `SQS`, `Lambda`, and `DynamoDB`.
6. Study scale and data design: `Partitioning`, `Sharding`, `Kafka-RabbitMQ`, and `Embedded-Database`.
7. Round out security and tooling with `Encryption`, `Firewall`, `FTP`, `TensorFlow`, and `PyCharm`.

## Practical Folder Convention

Use each `Practical/Code` folder for runnable assets:

- `README.md`: setup and commands.
- `src/`: source code.
- `tests/`: tests or load tests.
- `docker-compose.yml`: local dependencies when needed.
- `notes.md`: observations, benchmark results, screenshots, and mistakes.

Use each `Theory` folder for learning assets:

- `concepts.md`: core explanation.
- `tradeoffs.md`: when to use and when not to use it.
- `diagrams/`: architecture diagrams.
- `questions.md`: interview or self-check questions.
