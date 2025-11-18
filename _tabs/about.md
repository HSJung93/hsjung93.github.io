---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

As a backend engineer, I contribute directly to business growth by solving complex technical challenges, including an initiative that onboarded 200,000+ new users(2% of total users). I specialize in building stable and efficient systems, with strong expertise in managing large-scale traffic, modernizing legacy systems, and enhancing security.

## Key Achievements

### 1. USIM Reservation System Development

- Implemented a robust Rate Limiter to handle a massive surge in identity requests, comparing bucket4j, Resilience4j, and a custom Redis ZSET solution, validating with k6 load testing. Post-launch, I monitored Grafana/Prometheus dashboards to adhere to quotas and ensure service stability.

- Met a 3-day deadline for a partner admin login using Spring Session secured by an IP whitelist. Continuously hardened the system to pass all penetration tests, including adding Brute Force mitigation.

### 2. Calendar Service Development

- Solved user growth stagnation by developing an Outlook calendar sync, converting its proprietary API to the standard CalDAV protocol. This integration successfully onboarded over 200,000 new users.

- Addressed severe maintainability issues (circular dependencies, 2000+ unit tests) by migrating batch jobs to Spring Batch. I then refactored the main application to Hexagonal Architecture, separating adapters and resolving the dependency issues.

### 3. Development of a New Member & Authentication System (Legacy Replacement)

- Secured bloated JWTs by implementing JWE (JSON Web Encryption) for token encryption. Used k6 to proactively measure the performance impact, ensuring a balance between security and latency.

- Resolved a critical data integrity issue of duplicate user CIs post-migration. I coordinated a Soft Delete for duplicates and worked with a DBA to add a DB Unique constraint to the CI column, permanently fixing the problem.

### 4. Gateway Development for Web Services

- Architected and built a new, dedicated Web Service Gateway using Spring Cloud Gateway to handle overload and support a new SSE (Server-Sent Events) requirement. The gateway manages SSL, token authentication, and routing.

- Implemented full observability by using Ansible to configure the OpenTelemetry Agent for trace-id propagation in a Reactive environment. I also built a new Grafana dashboard using Spring Cloud Gateway's native metrics for accurate reporting.
