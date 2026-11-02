---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

Data engineer with backend roots (Spring). Since early 2026 I have worked as a forward deployed engineer for internal teams in a telecom marketing division: embedded with non-engineering teams to ship data pipelines, automation systems, and AI coding practices. Before that, four years of backend engineering on auth, calendar, and gateway services, including an integration that onboarded 200,000+ users (2% of total users).

## 2026 — Data Engineering & Forward Deployed Engineering (internal)

1. **Settlement-collection automation, shipped in 2 months with non-engineers.** Built the backend (FastAPI) and the whole dev/deploy environment (React, PostgreSQL, Kubernetes PaaS, nginx ingress) with two non-engineer planners, replacing a 4-month outsourcing estimate. Designed LLM-drafted reminder mail behind a batch-send policy, a master kill switch, and 3-tier access so mis-sends became an operational non-issue. Owner's analysis time down 75%, mail drafting down 50%.
   <!-- TODO post: [Read more](/posts/vibe-coding-two-months-with-non-engineers/) -->

2. **Team pipeline migration to dbt + Airflow.** Led the move from four ad-hoc styles (notebooks, raw SQL, shell, Python runners) to one standard. Built consensus with a repository diagnosis doc, then cut over in a single day using reverse-substitution checks and two-way EXCEPT tests. Protected branches, static checks, and MR conventions became the team norm; most MR reviews on the team.
   <!-- TODO post: [Read more](/posts/migrating-notebook-sql-to-dbt-safely/) -->

3. **Monthly Excel statistics → dbt mart + React dashboard (7-week MVP).** "Axes in the pipeline, combinations in the UI": the mart stores only the finest-grained axes, so most filter requests ship as UI-only changes. Golden tests and a dbt run-status page made the numbers trustworthy to non-engineers. 500 Vitest cases behind a coverage gate.
   <!-- TODO post: [Read more](/posts/axes-in-pipeline-combinations-in-ui/) -->

4. **Two 1M/1M randomized uplift experiments, and an upstream fix to uber/causalml.** Hash-based reproducible assignment instead of Athena's non-seedable `rand(n)`; corrected an apparent negative effect by re-aggregating on an ITT basis. Found and fixed a build defect in causalml's Causal Forest (Cython `linetrace` compiled into the release wheel → GIL contention), merged in v0.17.0: training 27 min → 2.6 min. [Read more](/posts/causalml-cython-linetrace-perf-en/)
   <!-- TODO post: [rand(n)](/posts/athena-rand-is-not-a-seed/) [ITT](/posts/itt-when-denominators-differ/) -->

5. **Division-wide internal AI adoption.** Designed and taught a 3-round vibe-coding and deployment curriculum for non-engineers across 14 teams. As a full-time member of the division's AI task force, analyzed workflow data from 147 staff, assessed 8 projects, and measured 2.4 FTE/yr of realized savings. Ran a Claude Code skill marketplace and repository rules as the team standard.
   <!-- TODO post: [teaching](/posts/teaching-git-and-nginx-to-marketers/) [team standard](/posts/claude-code-team-standard-mistakes/) -->

## 2022–2025 — Backend Engineering

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
