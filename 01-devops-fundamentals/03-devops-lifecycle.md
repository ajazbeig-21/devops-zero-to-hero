# 03 — The DevOps Lifecycle

> "The DevOps lifecycle is not a one-time process — it is an infinite loop. Every deployment feeds information back into planning the next one."

---

## Table of Contents

1. [The DevOps Infinity Loop](#1-the-devops-infinity-loop)
2. [Stage 1 — Plan](#2-stage-1--plan)
3. [Stage 2 — Code](#3-stage-2--code)
4. [Stage 3 — Build](#4-stage-3--build)
5. [Stage 4 — Test](#5-stage-4--test)
6. [Stage 5 — Release](#6-stage-5--release)
7. [Stage 6 — Deploy](#7-stage-6--deploy)
8. [Stage 7 — Operate](#8-stage-7--operate)
9. [Stage 8 — Monitor](#9-stage-8--monitor)
10. [The Feedback Loop — The Most Important Stage](#10-the-feedback-loop--the-most-important-stage)
11. [CI vs CD vs Continuous Deployment](#11-ci-vs-cd-vs-continuous-deployment)
12. [A Day in the Life — Following a Feature Through the Lifecycle](#12-a-day-in-the-life--following-a-feature-through-the-lifecycle)
13. [DevOps Lifecycle Anti-Patterns](#13-devops-lifecycle-anti-patterns)
14. [Summary](#14-summary)

---

## 1. The DevOps Infinity Loop

The iconic symbol of DevOps is the **∞ (infinity) loop** or Möbius strip. It represents:

- There is no "end" to software — it evolves continuously
- Development (left loop) and Operations (right loop) are equally important
- Feedback from the right loop (operations) informs the left loop (development)

```
         ┌────────────────────────────────────────────────────┐
         │                                                     │
    ┌────┴─────┐                                         ┌────┴─────┐
    │  PLAN    │                                         │  MONITOR │
    └────┬─────┘                                         └────┬─────┘
         │                  DEVELOPMENT                       │
    ┌────┴─────┐         ◄──────────────►          ┌──────────┴────┐
    │   CODE   │                                   │    OPERATE    │
    └────┬─────┘                                   └──────────┬────┘
         │                                                     │
    ┌────┴─────┐                                         ┌─────┴────┐
    │   BUILD  │                                         │  DEPLOY  │
    └────┬─────┘                                         └─────┬────┘
         │                  OPERATIONS                         │
    ┌────┴─────┐         ◄──────────────►          ┌─────┴────┐
    │   TEST   │                                   │  RELEASE │
    └──────────┘                                   └──────────┘
```

**The 8 stages:**

| # | Stage | Primary Owner | Core Question |
|---|-------|--------------|---------------|
| 1 | Plan | Product + Dev | What do we build next? |
| 2 | Code | Development | How do we build it? |
| 3 | Build | Dev + DevOps | Does it compile and package? |
| 4 | Test | QA + Dev + DevOps | Does it work correctly and safely? |
| 5 | Release | DevOps | Is it ready for production? |
| 6 | Deploy | DevOps | Is it live in production? |
| 7 | Operate | Ops + SRE + Dev | Is it running correctly? |
| 8 | Monitor | DevOps + SRE | Is it performing as expected? |

---

## 2. Stage 1 — Plan

### Purpose

Define WHAT to build, WHY to build it, and WHEN.

### Activities

**Product Planning:**
- Product roadmap refinement
- Feature prioritization (MoSCoW, RICE scoring, OKRs)
- Sprint planning and backlog grooming
- Capacity planning for the development team

**Technical Planning:**
- Architecture discussions (new service? new database? new API?)
- Dependency identification (does this feature block another?)
- Infrastructure capacity planning (will we need more compute?)
- Rollout strategy (what's the deployment plan? canary or blue-green?)
- Rollback plan (what do we do if it fails?)
- Feature flag design

**Tooling used:**

| Tool Category | Examples |
|--------------|----------|
| Project management | Jira, Linear, GitHub Issues, Asana, Notion |
| Roadmap planning | Productboard, Aha!, JIRA Portfolio |
| Collaboration | Confluence, Notion, Google Docs, Miro |
| OKR tracking | Lattice, Weekdone, Ally.io |

### DevOps deliverables from Plan stage

- Architecture Decision Records (ADRs) committed to the repo
- Acceptance criteria written with testable, automatable conditions
- Deployment strategy documented
- Feature flag names established
- SLO targets defined for new features
- Monitoring plan: which metrics to add, which alerts to create

### Why Plan matters in DevOps

Traditional teams plan once for a 6-month release. DevOps teams plan in 2-week sprints — shorter planning cycles mean faster response to market feedback.

---

## 3. Stage 2 — Code

### Purpose

Implement the planned feature or fix through well-crafted, tested code.

### Activities

- Write application code
- Write unit tests alongside the code (TDD recommended)
- Write infrastructure code (Terraform, Kubernetes manifests)
- Code review via pull requests
- Keep changes small and focused

### Branching Strategies

**Gitflow (traditional):**

```
main ─────────────────────────────────────────────────────→
      ↑                                               ↑
    release/1.0                                  release/2.0
         ↑                                          ↑
      develop ──────────────────────────────────────┘
         ↑         ↑
     feature/A  feature/B
```

Problems: Long-lived branches cause massive merge conflicts; deploy cycles are slow.

**Trunk-Based Development (preferred in DevOps):**

```
main ──────────────────────────────────────────────────────→
  ↑       ↑          ↑           ↑           ↑
feat/A  feat/B      feat/C      fix/D       feat/E
(1 day) (2 days)   (1 day)     (4 hrs)     (3 days)
  ↑ small commits merged frequently to main
```

Benefits:
- No long-lived branches = no big-bang merges
- CI runs on every merge to main
- Feature flags hide incomplete features
- Everyone works from the same codebase

**Branch naming conventions (common):**

```
feature/JIRA-123-user-authentication
fix/JIRA-456-login-null-pointer
hotfix/JIRA-789-payment-crash
chore/update-dependencies
docs/update-api-spec
```

### Code Quality Practices

**Pre-commit hooks (runs before your commit is saved):**

```bash
# .pre-commit-config.yaml example
repos:
  - repo: https://github.com/psf/black
    hooks:
      - id: black          # Python formatting
  - repo: https://github.com/pre-commit/pre-commit-hooks
    hooks:
      - id: trailing-whitespace
      - id: check-yaml
      - id: check-merge-conflict
  - repo: https://github.com/pycqa/flake8
    hooks:
      - id: flake8         # Python linting
```

**Pull Request best practices:**
- Small PRs (< 400 lines changed)
- Clear description: what, why, how to test
- Linked to a ticket/issue
- At least 1 reviewer who understands the code
- All CI checks passing before merge
- Screenshots/recordings for UI changes
- CODEOWNERS file auto-assigns reviewers

**Code quality metrics:**
- Code coverage (% of code exercised by tests)
- Cyclomatic complexity (how complex is the logic)
- Technical debt (SonarQube quantifies this)
- Duplication (DRY violations)

### Tooling used

| Tool Category | Examples |
|--------------|----------|
| Version control | Git |
| Hosting | GitHub, GitLab, Bitbucket, Azure DevOps |
| IDE | VS Code, IntelliJ, Vim, Neovim |
| Code review | GitHub PRs, GitLab MRs, Gerrit |
| Static analysis | SonarQube, ESLint, Pylint, RuboCop |
| Pre-commit | pre-commit, husky (Node.js) |
| Secrets detection | git-secrets, gitleaks, truffleHog |

---

## 4. Stage 3 — Build

### Purpose

Take the source code and transform it into a deployable artifact (Docker image, JAR, binary, zip, etc.).

### What happens in a build?

```
Source Code + Dependencies
         ↓
  Dependency Resolution
  (npm install / pip install / mvn download)
         ↓
  Compilation (if compiled language)
  (javac / go build / dotnet build)
         ↓
  Packaging
  (jar / docker build / zip)
         ↓
  Artifact Published to Registry
  (Docker Hub, ECR, Nexus, Artifactory, S3)
```

### Build Systems

| Language | Build Tool | Config file |
|----------|-----------|------------|
| Java | Maven, Gradle | pom.xml, build.gradle |
| JavaScript/TypeScript | npm, yarn, pnpm, Webpack, Vite | package.json |
| Python | pip, Poetry, setuptools | pyproject.toml, requirements.txt |
| Go | go build (native) | go.mod |
| .NET | MSBuild, dotnet CLI | .csproj |
| C/C++ | Make, CMake, Ninja | Makefile |

### Docker Build — The most important build concept in DevOps

```dockerfile
# Example Dockerfile
FROM node:20-alpine AS builder       # Multi-stage: build stage
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production         # Install only prod deps
COPY . .
RUN npm run build

FROM node:20-alpine AS runtime       # Multi-stage: runtime stage
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node                            # Security: non-root user
CMD ["node", "dist/index.js"]
```

**Docker best practices:**
- Use multi-stage builds to keep images small
- Use specific version tags, not `latest` in production
- Run as non-root user
- Use `.dockerignore` to exclude unnecessary files
- Scan images for vulnerabilities (Trivy, Snyk)
- Layer caching: order Dockerfile instructions from least-to-most frequently changing

### Artifact Management

Artifacts are the output of builds. They must be:

- **Versioned:** Every artifact has a unique version (e.g., `myapp:1.2.3-abc1234`)
- **Immutable:** Once published, an artifact never changes
- **Signed:** Cryptographically signed to verify authenticity
- **Scanned:** Checked for known vulnerabilities before deployment

**Artifact registries:**

| Type | Examples |
|------|----------|
| Docker images | Docker Hub, AWS ECR, GCP Artifact Registry, Azure ACR, Harbor |
| Maven/Gradle JARs | Nexus Repository, JFrog Artifactory, GitHub Packages |
| npm packages | npm Registry, GitHub Packages, Verdaccio |
| Python packages | PyPI, Nexus, Artifactory |
| Generic artifacts | S3, Azure Blob Storage, GCS |

### Versioning

**Semantic Versioning (SemVer):** `MAJOR.MINOR.PATCH`

```
1.0.0   → Initial release
1.0.1   → Bug fix (patch: backward compatible)
1.1.0   → New feature (minor: backward compatible)
2.0.0   → Breaking change (major: NOT backward compatible)

Pre-release: 1.0.0-alpha.1, 1.0.0-rc.2
Build metadata: 1.0.0+git.abc1234
```

---

## 5. Stage 4 — Test

### Purpose

Verify that the artifact is correct, performant, and secure before it reaches users.

### The Testing Pyramid (Revisited)

```
                    ╔═══════════════╗
                    ║   E2E Tests   ║   Slow, expensive, test full flow
                    ╚═══════════════╝
               ╔═════════════════════════╗
               ║   Integration Tests     ║   Services talking together
               ╚═════════════════════════╝
          ╔═════════════════════════════════╗
          ║         Unit Tests              ║   Fast, isolated, granular
          ╚═════════════════════════════════╝
```

**Rule of thumb:** 70% unit, 20% integration, 10% E2E.

### Test Execution in CI Pipeline

```yaml
# GitHub Actions example
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: SAST - SonarQube scan
        run: sonar-scanner
      
      - name: Dependency vulnerability scan
        run: npm audit --audit-level=high
      
      - name: Container image scan
        run: trivy image myapp:${{ github.sha }}
```

### Test Environment Strategy

**Environment tiers:**

```
Local  →  Dev (CI)  →  Staging  →  Production
  ↑           ↑             ↑              ↑
Developer    Every        PR merge    Production
  machine   commit         gate        release
                
Fast loop    Full tests   UAT, perf   Smoke +
Unit only    + security   load test   synthetic
```

**Test data management:**
- Never use real production data in testing
- Use factories/fixtures for test data
- Anonymize/mask sensitive data in staging
- Database seeding scripts in version control

---

## 6. Stage 5 — Release

### Purpose

Prepare a tested, versioned artifact for deployment. This stage answers: "Is this artifact ready to go to production?"

### Release vs Deploy

This distinction confuses many people:

```
RELEASE:  "This artifact is approved and ready for production."
         (Semantic versioning, changelog, approval gates)

DEPLOY:   "This artifact is now running in production."
         (Infrastructure change — traffic goes to new version)
```

In **Continuous Delivery**: Release is automated up to an approval gate — a human approves the final step.
In **Continuous Deployment**: Release and Deploy are fully automated. No human gate.

### Release Activities

1. **Semantic version tagging:** `git tag v1.2.3`
2. **Changelog generation:** From commit messages (Conventional Commits → auto-changelog)
3. **Release notes:** Human-readable description of what changed
4. **Artifact promotion:** Move artifact from dev registry → prod registry
5. **Approval gates:** Required sign-offs (security, QA, product owner)
6. **Deployment plan:** Which environments, in which order, with what rollback plan

### Conventional Commits (enables automated changelogs)

```
feat: add user authentication via OAuth2
fix: resolve null pointer in payment processing
docs: update API documentation for v2 endpoints
chore: upgrade dependencies to latest versions
refactor: extract authentication logic into separate module
perf: add database indexing to speed up user queries
BREAKING CHANGE: remove deprecated /v1/users endpoint
```

**Changelog auto-generated from these commits:**

```markdown
## v2.1.0 (2026-03-28)

### Features
- add user authentication via OAuth2

### Bug Fixes
- resolve null pointer in payment processing

### Performance
- add database indexing to speed up user queries

### BREAKING CHANGES
- remove deprecated /v1/users endpoint
```

### Release Approval Strategies

| Approach | Description | When to use |
|----------|-------------|-------------|
| Fully automated | All gates are automated checks | Mature CI/CD, high test coverage |
| Automated + 1 human | Automated checks + 1 human approves | Most common in production |
| Change Advisory Board (CAB) | Committee approves all production changes | Regulated industries (finance, healthcare) |
| Peer approval | Teammate approves deploy | Engineering team discretion |

---

## 7. Stage 6 — Deploy

### Purpose

Get the verified artifact running in the target environment (production, staging, etc.).

### Deployment Pipeline

```
Staging Deploy → Smoke Tests → Canary 5% → Monitor 15min → 
Canary 25% → Monitor 15min → Full Deploy 100% → Monitor
```

### Deployment Strategies (Deep Dive)

#### Recreate Deployment
```
Old: [v1][v1][v1]
            ↓ shutdown all
            [    ][    ][    ]
            ↓ start new
New: [v2][v2][v2]

Downtime: Yes
Risk: High if v2 has issues
Use case: Local dev, non-prod environments
```

#### Rolling Deployment
```
Round 1:  [v2][v1][v1][v1]
Round 2:  [v2][v2][v1][v1]
Round 3:  [v2][v2][v2][v1]
Round 4:  [v2][v2][v2][v2]

Downtime: Zero
Risk: Both versions live briefly (must be backward compatible)
Use case: Default in Kubernetes Deployments
```

#### Blue-Green Deployment
```
LIVE:    Blue  env: [v1][v1][v1]  ←── 100% traffic
STANDBY: Green env: [v2][v2][v2]  ←── 0% traffic

Switch:
LIVE:    Green env: [v2][v2][v2]  ←── 100% traffic
STANDBY: Blue  env: [v1][v1][v1]  ←── 0% (instant rollback available)

Downtime: Zero
Risk: Low — instant rollback
Cost: 2× infrastructure during transition
Use case: Critical systems needing instant rollback
```

#### Canary Deployment
```
Stage 1:  [v2]  1%  of traffic    [v1]  99% of traffic
Stage 2:  [v2]  10% of traffic    [v1]  90% of traffic
Stage 3:  [v2]  50% of traffic    [v1]  50% of traffic
Stage 4:  [v2] 100% of traffic
          (v1 decommissioned)

Downtime: Zero
Risk: Very low — only small % of users affected if v2 breaks
Use case: High-traffic services where risk of new version is unknown
Tools: Argo Rollouts, Flagger, Istio, AWS CodeDeploy
```

### Kubernetes Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Allow 1 extra pod during update
      maxUnavailable: 0  # Don't take down any pods — zero downtime
  selector:
    matchLabels:
      app: myapp
  template:
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:1.2.3
        readinessProbe:              # K8s won't route traffic until this passes
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

### Deployment Checklist

- [ ] All tests passing in CI
- [ ] Artifact version tagged and signed
- [ ] Database migrations backward compatible
- [ ] Feature flags configured correctly
- [ ] Monitoring alerts updated for new endpoints/metrics
- [ ] Runbook updated if new operational procedures needed
- [ ] Rollback plan documented and tested
- [ ] Communication sent to stakeholders
- [ ] On-call rotation reviewed (who is responsible post-deploy?)

---

## 8. Stage 7 — Operate

### Purpose

Keep the application running reliably in production. Handle incidents, scale infrastructure, manage configurations.

### Operational Responsibilities

**Incident Management Process:**

```
Alert fires
    ↓
On-call engineer acknowledges (< 5 min)
    ↓
Initial triage – What is the impact? Who is affected?
    ↓
Incident declared: P1/P2/P3/P4
    ↓
Incident commander assigned (for P1/P2)
    ↓
Diagnosis – Find root cause or workaround
    ↓
Mitigation – Stop the bleeding (rollback, feature flag off, etc.)
    ↓
Resolution – Full fix deployed
    ↓
Post-mortem – Blameless analysis within 48 hrs
```

**Incident severity levels:**

| Severity | Definition | SLA Response |
|----------|-----------|-------------|
| P1 (Critical) | Production down, all users affected | 5 minutes |
| P2 (High) | Major feature broken, most users affected | 15 minutes |
| P3 (Medium) | Minor feature broken, some users affected | 1 hour |
| P4 (Low) | Cosmetic issue, workaround available | Next business day |

### Infrastructure Operations

**Toil** (term from SRE):
> Manual, repetitive, tactical work that grows with service size and provides no enduring value.

Examples of toil:
- Manually restarting pods
- Manually rotating certificates
- Manually resizing volumes
- Manually running database backups
- Manually assigning permissions in a console

**Toil elimination through automation:**

| Toil Activity | Automation Solution |
|---------------|---------------------|
| Pod crashes need manual restart | Kubernetes liveness probes + auto-restart |
| SSL cert renewal every 90 days | cert-manager (auto-renewal) |
| Database backup | Cron job + cloud snapshot |
| User provisioning | Terraform + SCIM provisioning |
| Scaling on traffic spikes | HPA (Horizontal Pod Autoscaler) |
| Certificate rotation | HashiCorp Vault + auto-rotation |

### Configuration Management

**The 12-Factor App — Factor 3: Config**

> Store config in the environment. Never hardcode configuration values in the application.

```
BAD:
const db = new Database('10.0.0.1', 'admin', 'password123')

GOOD:
const db = new Database(
  process.env.DB_HOST,
  process.env.DB_USER,
  process.env.DB_PASSWORD
)
```

**Config management tools:**

| Scope | Tool | Use case |
|-------|------|----------|
| App config | Environment variables | Twelve-factor apps |
| Secrets | HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager | API keys, passwords, certs |
| Infrastructure config | Terraform variables, Ansible vars | Infrastructure settings |
| K8s config | ConfigMaps, Secrets | Kubernetes application config |
| Feature flags | LaunchDarkly, Flagsmith, Unleash | Runtime feature toggles |

### Scaling

**Vertical scaling (scale up):** Increase the size of the machine (more CPU/RAM).
- Simple but has limits
- Requires downtime in many cases
- More expensive proportionally

**Horizontal scaling (scale out):** Add more instances of the service.
- Requires stateless services (session data in Redis, not in-memory)
- Theoretically unlimited
- More complex but more resilient

**Auto-scaling in Kubernetes:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    name: myapp
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70     # Scale when CPU > 70%
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80     # Scale when memory > 80%
```

---

## 9. Stage 8 — Monitor

### Purpose

Continuously observe the system state and emit actionable signals when something is wrong.

### The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────┐
│                 OBSERVABILITY                           │
│                                                         │
│  ┌─────────┐      ┌─────────┐      ┌─────────────────┐ │
│  │  LOGS   │      │METRICS  │      │    TRACES       │ │
│  │         │      │         │      │                 │ │
│  │"What    │      │"How     │      │"Where did       │ │
│  │happened"│      │fast/how │      │this request     │ │
│  │         │      │many?"   │      │go?"             │ │
│  └─────────┘      └─────────┘      └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

**Logs:** Timestamped records of events that occurred.

```json
{
  "timestamp": "2026-03-28T14:32:01Z",
  "level": "ERROR",
  "service": "payment-service",
  "traceId": "abc123",
  "message": "Payment failed",
  "userId": "usr_456",
  "errorCode": "INSUFFICIENT_FUNDS",
  "duration_ms": 245
}
```

**Metrics:** Numerical measurements over time.
- `http_request_duration_seconds{method="GET", path="/checkout", status="200"} 0.156`
- `jvm_memory_used_bytes{area="heap"} 524288000`
- `deployment_count{env="production", app="myapp"} 42`

**Traces:** Records the journey of a request across multiple services.

```
User Request
     ↓
  API Gateway (5ms)
     ↓
  Auth Service (12ms)
     ↓
  Product Service (45ms)
     │
     ├── Database Query (32ms)
     └── Cache Hit (2ms)
     ↓
  Payment Service (78ms)
     │
     └── Stripe API Call (65ms)

Total: 140ms
```

### Monitoring Tools Stack

| Layer | Tools |
|-------|-------|
| Metrics collection | Prometheus, Datadog, CloudWatch, Dynatrace |
| Metrics visualization | Grafana |
| Log aggregation | ELK Stack (Elasticsearch + Logstash + Kibana), Loki + Grafana |
| Distributed tracing | Jaeger, Zipkin, AWS X-Ray, Tempo |
| APM (all-in-one) | Datadog, New Relic, Dynatrace, AppDynamics |
| Alerting | PagerDuty, OpsGenie, Prometheus Alertmanager, Grafana Alerting |
| Synthetic monitoring | Checkly, Pingdom, Datadog Synthetics |
| Error tracking | Sentry, Rollbar, Bugsnag |

### The RED Method (for services)

| Letter | Metric |
|--------|--------|
| R — Rate | Requests per second (throughput) |
| E — Errors | % of requests that failed |
| D — Duration | Time each request takes (latency) |

### The USE Method (for infrastructure)

| Letter | Metric |
|--------|--------|
| U — Utilization | % time resource is busy (e.g., CPU at 80%) |
| S — Saturation | Queue depth / backlog (overflow of resource) |
| E — Errors | Error events from the resource |

### The Four Golden Signals (Google SRE)

| Signal | Description |
|--------|-------------|
| Latency | Time to serve a request |
| Traffic | Demand on the system (requests/sec) |
| Errors | Rate of failed requests |
| Saturation | How "full" the service is (queue depth, CPU %) |

### SLO Alerting

```yaml
# Example SLO alert
# Alert if error rate exceeds 1% (0.999 SLO)
- alert: ErrorRateTooHigh
  expr: |
    (
      sum(rate(http_requests_total{status=~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m]))
    ) > 0.01
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Error rate above SLO threshold"
    description: "Error rate is {{ $value | humanizePercentage }} — SLO is 99.9%"
```

### Observability Best Practices

1. **Structured logging** — JSON format with consistent fields. Never free-text logs in production.
2. **Correlation IDs** — Pass a trace ID through all services so you can connect logs across them.
3. **Cardinality awareness** — Don't put high-cardinality values (user IDs, request IDs) in metric labels — use traces instead.
4. **Alerting on symptoms, not causes** — Alert "users can't check out" not "CPU at 90%".
5. **Keep runbooks linked to alerts** — Every alert should have a link to a documented response procedure.
6. **Dashboard-as-code** — Store Grafana dashboards as JSON in Git.

---

## 10. The Feedback Loop — The Most Important Stage

The feedback loop is what makes DevOps fundamentally different from Waterfall.

```
Monitor → Insights → Backlog → Plan → Code → Build → Test → Deploy → Monitor
   ↑                                                                      ↓
   └──────────────────────── FEEDBACK LOOP ──────────────────────────────┘
```

**Types of feedback:**

| Type | Source | How it feeds back |
|------|--------|-------------------|
| Error rate spike | Monitoring | Creates bug ticket in sprint backlog |
| Slow endpoint | Trace data | Creates performance task |
| User abandoning checkout | Analytics | Product decision to investigate UX |
| High CPU on a service | Metrics | Architecture discussion (scale out? optimize?) |
| Failed deployment | CI/CD pipeline | Immediate notification to developer |
| Security vulnerability | SAST/dependency scan | Auto-created security ticket |
| Post-mortem action items | Incident | Tasks added to sprint |

**The faster the feedback, the cheaper the fix:**

```
Cost to fix a bug:
  ┌─────────────────────────────────────────────────────┐
  │ In development (pre-commit)    →  $1                 │
  │ In CI pipeline                 →  $10                │
  │ In staging                     →  $100               │
  │ Caught in production           →  $1,000+            │
  │ Customer-reported in prod      →  $10,000+ (+ brand) │
  └─────────────────────────────────────────────────────┘
```

---

## 11. CI vs CD vs Continuous Deployment

This is one of the most commonly confused topics in DevOps.

### Continuous Integration (CI)

**Definition:** The practice of developers merging code to a shared branch frequently (daily or more), where each merge triggers an automated build and test suite.

```
Developer commits code
         ↓
  Git push to feature branch
         ↓
  Pull Request opened
         ↓
  CI pipeline triggered automatically:
    ✓ Build
    ✓ Lint / format check
    ✓ Unit tests
    ✓ Integration tests
    ✓ Security scan (SAST)
    ✓ Dependency scan
    ✓ Docker build + image scan
         ↓
  All green → available to merge
  Any red   → PR blocked, developer notified immediately
```

**Goal of CI:** Catch integration problems early. "If it's not tested, it's broken."

### Continuous Delivery (CD)

**Definition:** Every code change that passes CI is automatically deployed to a staging/pre-production environment. A human then approves the final production deployment.

```
CI Pipeline passes
         ↓
  Auto-deploy to Staging
         ↓
  Automated smoke tests on staging
         ↓
  Manual approval gate (product owner, QA lead, etc.)
         ↓
  Production deployment
```

**Key:** The software is ALWAYS in a deployable state. Production deployment is a business decision, not a technical event.

### Continuous Deployment (No manual gate)

**Definition:** Every change that passes the entire automated pipeline is automatically deployed to production without any human approval.

```
CI Pipeline passes
         ↓
  Auto-deploy to Staging
         ↓
  Automated tests pass
         ↓
  Auto-deploy to Production
         ↓
  Automated smoke tests pass
         ↓
  (New version now live — zero human intervention)
```

**Who uses this:** Companies with extremely high confidence in their test suite and monitoring. Amazon, Netflix, Etsy, Facebook. Requires mature automation and near-perfect test coverage.

### Visual Summary

```
Commit → Build → Test → Staging → [GATE] → Production

CI:              Stops here ↑
CD:                         Auto-deploys here, human gate → Production
Continuous Deployment:      Fully automated, no human gate at all ──────→
```

---

## 12. A Day in the Life — Following a Feature Through the Lifecycle

Let's walk through a real feature: "Add dark mode to the user dashboard."

### Plan (Day 1, 30 minutes)

- Product owner writes user story: "As a user, I want a dark mode toggle in Settings so that I can reduce eye strain."
- Acceptance criteria: "Toggle persists across sessions. Available on web and mobile. Complies with WCAG AA contrast ratios."
- Dev estimates: 3 days of work
- ADR: Store preference in user.settings table (no new service needed)

### Code (Day 1–3)

- Developer creates branch: `feature/JIRA-234-dark-mode`
- Commits daily to the branch
- Unit tests for the preference storage logic
- Component tests for the toggle UI
- `git push` triggers pre-commit hooks (linting, formatting)
- Opens PR on Day 3

### Build (Day 3, automated, 4 minutes)

CI pipeline triggers on PR:
- `npm install` — resolves dependencies
- `npm run build` — TypeScript compiled, bundles created
- Docker image built: `myapp:feature-jira-234-abc1234`
- Image pushed to staging registry

### Test (Day 3, automated, 12 minutes)

- Unit tests: 847 tests pass ✓
- Integration tests: API tests pass ✓
- SAST: No critical vulnerabilities ✓
- Dependency scan: No high CVEs ✓
- Image scan: Clean ✓
- Accessibility check: WCAG AA passes ✓
- PR status turns green — reviewer notified

### Review + Merge (Day 3–4)

- Fellow developer reviews, leaves 2 comments
- Developer addresses feedback
- PR approved and merged to `main`

### Release (Day 4, automated, 2 minutes)

- Merge to `main` triggers release pipeline
- Version bumped: `1.47.0` (minor feature bump)
- Changelog auto-generated from conventional commits
- Artifact promoted to production registry

### Deploy (Day 4, automated, 8 minutes)

- Canary deployment: 5% of users see dark mode
- Automated smoke tests on production: pass ✓
- Metrics check: no error rate increase for canary users
- After 30 minutes with healthy metrics: 100% rollout
- Total deploy time: 38 minutes from PR merge

### Operate (Day 4 onwards)

- Feature flag `dark_mode_enabled = true` (was the kill-switch, now permanent)
- No additional operational burden — preference stored in existing DB

### Monitor (Day 4 onwards)

- `dark_mode_toggle_clicked` event tracked in analytics
- Dashboard shows 34% of users enabled dark mode on Day 1
- No increase in error rates
- No change in session duration (positive signal)

### Feedback (1 week later)

- Data shows users on OLED phones use dark mode 94% of the time
- Product team creates follow-up ticket: "High contrast mode for accessibility users"

**Total time from idea to production: 4 days.**

---

## 13. DevOps Lifecycle Anti-Patterns

Knowing what NOT to do is as important as knowing what to do.

| Anti-Pattern | Description | Fix |
|-------------|-------------|-----|
| **Big-bang releases** | Accumulate 3 months of changes and release everything at once | Small, frequent releases |
| **Long-lived branches** | Feature branches that live for weeks | Trunk-based development + feature flags |
| **Manual deployments** | "Bob knows how to deploy, ask Bob" | Document + automate everything |
| **Skipping staging** | Deploying directly from CI to production | Always deploy to staging first |
| **Flaky tests** | Tests that randomly pass/fail — teams start ignoring them | Fix or quarantine flaky tests; never ignore |
| **No rollback plan** | "We'll figure it out if it breaks" | Every deployment has a documented rollback procedure |
| **Alert fatigue** | So many alerts that on-call ignores them | Only alert on symptoms; reduce noise; set SLOs |
| **Security at the end** | "We'll do the security review before release" | Shift left: security scanning in every CI run |
| **Pets vs Cattle** | Treating servers like unique snowflakes that can't be replaced | Immutable infrastructure; servers are disposable |
| **Works on my machine** | No parity between local and production | Docker + docker-compose for local dev |

---

## 14. Summary

| Stage | Core Activity | Key Tools | Key Metric |
|-------|--------------|-----------|------------|
| Plan | Define what to build | Jira, Linear, Confluence | Sprint velocity |
| Code | Write tested code | Git, GitHub, pre-commit | PR merge time |
| Build | Package the artifact | Docker, Maven, npm | Build duration |
| Test | Validate correctness and security | Jest/pytest/JUnit, Trivy, SonarQube | Test coverage, CVE count |
| Release | Version and approve | GitHub Releases, semantic-release | Lead time for changes |
| Deploy | Get it live | Kubernetes, ArgoCD, Terraform | Deployment frequency |
| Operate | Keep it running | Kubernetes, Ansible, Vault | MTTR, toil hours |
| Monitor | Observe and alert | Prometheus, Grafana, PagerDuty | SLO compliance, DORA metrics |

**The most important insight:** The feedback loop from Monitor back to Plan is what distinguishes a DevOps team from a team that just has CI/CD pipelines.

---

**Previous:** [02 — SDLC](02-sdlc.md)
**Next:** [04 — Culture & Mindset](04-culture-and-mindset.md)
