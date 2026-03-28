# 02 — Software Development Lifecycle (SDLC)

> "Every piece of software ever written was produced by some version of the SDLC. DevOps didn't replace the SDLC — it turbocharged it."

---

## Table of Contents

1. [What is the SDLC?](#1-what-is-the-sdlc)
2. [SDLC Phases — Deep Dive](#2-sdlc-phases--deep-dive)
3. [SDLC Models](#3-sdlc-models)
4. [Waterfall Model](#4-waterfall-model)
5. [V-Model](#5-v-model)
6. [Iterative Model](#6-iterative-model)
7. [Spiral Model](#7-spiral-model)
8. [Agile Model](#8-agile-model)
9. [DevOps Model — SDLC Evolved](#9-devops-model--sdlc-evolved)
10. [SDLC Model Comparison](#10-sdlc-model-comparison)
11. [How DevOps Transforms Each SDLC Phase](#11-how-devops-transforms-each-sdlc-phase)
12. [Value Stream Mapping](#12-value-stream-mapping)
13. [Summary](#13-summary)

---

## 1. What is the SDLC?

**SDLC (Software Development Lifecycle)** is a structured process that defines the steps, roles, and deliverables used to plan, create, test, deploy, and maintain software.

Think of it as the **blueprint** for how a software idea becomes a running system.

### Why does SDLC matter for a DevOps engineer?

- You need to know WHERE in the lifecycle you plug in automation
- Understanding SDLC helps you map tools to phases correctly
- Interview questions often ask: "Where does CI/CD fit in the SDLC?"
- Knowing the history explains WHY DevOps practices exist

### The Generic SDLC — 7 Phases

```
1. Plan
    ↓
2. Requirements
    ↓
3. Design
    ↓
4. Implementation (Code)
    ↓
5. Testing & QA
    ↓
6. Deployment
    ↓
7. Maintenance & Operations
    ↓
 (back to Plan for next release)
```

Every SDLC model covers some version of these 7 phases — they just differ in **how they sequence them**, **how long each phase takes**, and **how much feedback flows between them**.

---

## 2. SDLC Phases — Deep Dive

### Phase 1: Planning

**What happens:**
- Define project scope, timelines, and resources
- Feasibility analysis: technical, financial, operational
- Risk assessment
- Resource allocation (team size, budget, infrastructure costs)

**Key artifacts:**
- Project charter
- Feasibility study
- Risk register
- Work breakdown structure (WBS)

**DevOps touch:** Feature flags planned from the start. Deployment strategy (blue-green, canary) decided at planning, not at deployment day.

---

### Phase 2: Requirements Analysis

**What happens:**
- Gather functional requirements: what the software must DO
- Gather non-functional requirements: performance, security, scalability, accessibility
- Stakeholder interviews and workshops
- Document requirements formally

**Key artifacts:**
- BRD (Business Requirements Document)
- SRS (Software Requirements Specification)
- User stories (Agile) or use cases (traditional)
- Acceptance criteria

**Types of requirements:**

| Type | Example |
|------|---------|
| Functional | "User can log in with email and password" |
| Non-functional | "Login response time < 200ms at 10,000 concurrent users" |
| Security | "All passwords hashed with bcrypt, min cost factor 12" |
| Compliance | "GDPR — user data deletable on request within 30 days" |
| Operational | "System must be deployable with zero downtime" |

**DevOps touch:** Non-functional requirements (performance, security, reliability) are written as **testable criteria** that gate the CI/CD pipeline.

---

### Phase 3: System Design

**What happens:**
- High-Level Design (HLD): architecture, technology stack, infrastructure, data flow
- Low-Level Design (LLD): database schemas, API contracts, class diagrams, module interfaces
- Security design: threat modeling, authentication architecture
- Infrastructure design: cloud provider, regions, VPCs, networking

**Key artifacts:**
- Architecture diagrams (C4 model, UML)
- API specifications (OpenAPI / Swagger)
- Database ERD
- ADRs (Architecture Decision Records)
- Infrastructure diagrams

**Design considerations a DevOps engineer owns:**
- Is the app containerizable?
- How will configuration be managed? (12-factor app principles)
- What's the deployment unit? (monolith vs microservices)
- How will secrets be managed?
- What does a rollback look like?
- How will we monitor this in production?

**DevOps touch:** Observability designed in from the start — logging format, metric naming conventions, tracing instrumentation.

---

### Phase 4: Implementation (Development / Coding)

**What happens:**
- Developers write application code
- Code reviews via pull requests
- Unit tests written alongside code (TDD ideally)
- Feature branches merged to a shared integration branch

**Key DevOps practices at this phase:**

| Practice | What it means |
|----------|---------------|
| Trunk-Based Development | Small, frequent commits to main branch rather than long-lived feature branches |
| Feature Flags | New features hidden behind toggles so code can be merged but not activated |
| Pre-commit hooks | Linting and formatting checks before code is even committed |
| Pair programming | Two developers on one screen — built-in code review |
| Code ownership | CODEOWNERS file defines who reviews which parts of the codebase |

**Key artifacts:**
- Source code in version control (Git)
- Unit tests
- Pull requests with reviews
- Updated documentation

---

### Phase 5: Testing & QA

**The testing pyramid:**

```
            ┌─────────────┐
            │     E2E /   │  ← Few, slow, expensive
            │   UI Tests  │     (Selenium, Playwright, Cypress)
           ─┼─────────────┼─
          ╱ │ Integration │  ← Moderate number
         ╱  │    Tests    │     (API tests, database tests)
        ╱   └─────────────┘
       ╱                          
   ───────────────────────────
   │        Unit Tests        │  ← Many, fast, cheap
   └───────────────────────────    (Jest, JUnit, pytest, Go test)
```

**Types of testing in a DevOps pipeline:**

| Test Type | What it checks | When it runs | Tool examples |
|-----------|---------------|--------------|---------------|
| Unit tests | Individual functions/methods | Every commit (CI) | Jest, JUnit, pytest, Go test |
| Integration tests | Services talking to each other | PR + main branch CI | Postman, RestAssured, Testcontainers |
| Contract tests | API contracts between services | PR CI | Pact |
| Performance tests | Load, stress, spike behavior | Scheduled nightly or pre-release | k6, Gatling, JMeter, Locust |
| Security tests (SAST) | Static code vulnerabilities | Every commit | SonarQube, Semgrep, Checkov |
| Security tests (DAST) | Running app vulnerabilities | Staging environment | OWASP ZAP, Burp Suite |
| Dependency scanning | Known CVEs in libraries | Every commit | Dependabot, Snyk, Trivy |
| Infrastructure tests | IaC correctness | PR CI | Terratest, Kitchen-Terraform |
| Smoke tests | Basic production health | After every deployment | Custom scripts, synthetic monitors |
| User Acceptance Tests (UAT) | Business logic validation | Staging before production | Manual or Selenium |

**Key principle — Test Environments:**

```
Local (developer laptop)  →  Dev  →  Staging  →  Production
        ↑ fast, cheap              ↑ mirrors prod    ↑ real users
```

**DevOps touch:** Testing shifts left into CI. Security scanning is automated in the pipeline. Test results gate deployments — a failing test blocks the pipeline.

---

### Phase 6: Deployment

**What happens:**
- Application is packaged and deployed to production
- Configuration management ensures correct settings
- Database migrations applied
- Traffic switched to new version

**Deployment strategies:**

| Strategy | Description | Risk | Downtime |
|----------|-------------|------|----------|
| **Recreate** | Shutdown old, start new | High | Yes |
| **Rolling** | Replace instances one by one | Medium | Minimal |
| **Blue-Green** | Two identical environments; switch traffic | Low | Zero |
| **Canary** | Route small % of traffic to new version first | Very Low | Zero |
| **A/B Testing** | Different versions for different user segments | Low | Zero |
| **Shadow** | New version receives traffic but responses discarded | Zero | Zero |
| **Feature Flags** | Deploy code, toggle feature on/off in config | Zero | Zero |

**Blue-Green Deployment visualized:**

```
BEFORE DEPLOYMENT:
  Load Balancer ──100%──→ Blue (v1.0) ← LIVE
                          Green (v1.1) ← IDLE

DEPLOYMENT:
  Load Balancer ──0%───→ Blue (v1.0)
                ──100%──→ Green (v1.1) ← NOW LIVE

ROLLBACK (instant):
  Load Balancer ──100%──→ Blue (v1.0) ← REVERTED
```

**Canary Deployment visualized:**

```
Stage 1:  99% → v1.0 (old)    1% → v1.1 (canary)
Stage 2:  80% → v1.0          20% → v1.1
Stage 3:  50% → v1.0          50% → v1.1
Stage 4:   0% → v1.0         100% → v1.1
           (old decommissioned after stability confirmed)
```

---

### Phase 7: Maintenance & Operations

**What happens:**
- Monitor application health (uptime, errors, performance)
- Respond to incidents
- Apply security patches and dependency updates
- Gather user feedback
- Manage capacity and scaling

**Key operational duties for a DevOps engineer:**

| Activity | Description |
|----------|-------------|
| Incident Management | Detect, respond, resolve, and post-mortem production issues |
| Capacity Planning | Predict and provision resources ahead of demand |
| Performance Tuning | Identify bottlenecks and optimize |
| Security Patching | Apply CVE patches; rotate credentials |
| On-call | Respond to pages and alerts 24/7 |
| Toil Reduction | Automate repetitive operational tasks |
| Cost Optimization | Right-size infrastructure, spot instances, reserved capacity |

---

## 3. SDLC Models

There are multiple models that describe how teams move through SDLC phases. Understanding all of them is necessary because:
- You'll encounter legacy teams still on Waterfall
- Many enterprises use a hybrid model
- Interviews ask you to compare models

---

## 4. Waterfall Model

**Invented:** 1970 (Winston W. Royce paper)
**Nature:** Linear and sequential — each phase must be 100% complete before the next begins.

```
┌──────────────┐
│  Requirements│ ← Complete requirements document before moving on
└──────┬───────┘
       ↓
┌──────────────┐
│    Design    │ ← Full architecture before writing one line of code
└──────┬───────┘
       ↓
┌──────────────┐
│Implementation│ ← Code the entire system
└──────┬───────┘
       ↓
┌──────────────┐
│   Testing    │ ← Test after all code is written
└──────┬───────┘
       ↓
┌──────────────┐
│  Deployment  │
└──────┬───────┘
       ↓
┌──────────────┐
│ Maintenance  │
└──────────────┘
```

### Advantages of Waterfall

- Simple to understand and manage
- Clear milestones and deliverables
- Works well for projects with very stable, well-understood requirements (e.g., NASA spacecraft, embedded firmware)
- Documentation is comprehensive

### Disadvantages of Waterfall

- Requirements change mid-project → entire plan breaks
- No working software until very late in the process
- Testing happens too late — bugs are expensive to fix
- Customer doesn't see anything until the end
- Risk accumulates — a wrong assumption in requirements affects all downstream phases

**Real-world failure:** UK National Program for IT (NPfIT) — £12.7 billion NHS project cancelled after 10 years. Classic Waterfall failure: requirements defined upfront for a decade-long project in a fast-changing environment.

### When to use Waterfall today

- Government/defense contracts with strict documentation requirements
- Projects with legally mandated phases (FDA medical device software)
- Short, simple projects with fully known requirements

---

## 5. V-Model

Extension of Waterfall that adds a testing phase corresponding to each development phase.

```
Requirements ────────────────────────────→ Acceptance Testing
     ↓                                            ↑
 System Design ────────────────────→ System Testing
     ↓                                     ↑
 Architecture Design ──────→ Integration Testing
     ↓                              ↑
  Module Design ──────→ Unit Testing
           ↓         ↑
           Implementation
```

**Key idea:** For every "left side" (development) phase, there is a corresponding "right side" (testing) phase defined at the same time.

**Advantage over Waterfall:** Testing is planned from the start, not as an afterthought.

**Still has Waterfall's main problem:** No working software until late; feedback loops are slow.

---

## 6. Iterative Model

Build the system in small, repeated iterations. Each iteration produces a working (partial) system.

```
Iteration 1:  Plan → Design → Build → Test → Deploy (basic version)
Iteration 2:  Plan → Design → Build → Test → Deploy (add features)
Iteration 3:  Plan → Design → Build → Test → Deploy (add more features)
...
```

**Advantage:** Working software exists early. Requirements can evolve between iterations.

**Disadvantage:** Can lead to scope creep. Architecture can become inconsistent if not carefully managed.

---

## 7. Spiral Model

Combines iterative development with Waterfall's risk management focus. Each "spiral" is:

1. Determine objectives
2. Identify and resolve risks
3. Development and testing
4. Plan next iteration

**Best for:** Large, high-risk, complex projects (aerospace, large enterprise systems).

**Disadvantage:** Expensive; requires risk management expertise; hard to apply to small teams.

---

## 8. Agile Model

**Published:** 2001 (Agile Manifesto)
**Nature:** Iterative, incremental, collaborative. Work is divided into short **sprints** (2–4 weeks).

### The Agile Manifesto Values:

```
Individuals and interactions     OVER  processes and tools
Working software                 OVER  comprehensive documentation
Customer collaboration           OVER  contract negotiation
Responding to change             OVER  following a plan
```

Note: Items on the right have value — but items on the left are valued MORE.

### Agile Principles (12 Principles):

1. Deliver valuable software early and continuously
2. Welcome changing requirements, even late in the project
3. Deliver working software frequently (weeks, not months)
4. Business and developers work together daily
5. Build projects around motivated individuals; trust them
6. Face-to-face conversation is the most efficient communication
7. Working software is the primary measure of progress
8. Sustainable pace — dev teams should be able to maintain indefinitely
9. Continuous attention to technical excellence
10. Simplicity — maximize the work not done
11. Self-organizing teams produce the best architectures
12. Reflect regularly and adjust behavior

### Scrum Framework (most popular Agile implementation)

```
PRODUCT BACKLOG
(prioritized list of all features)
         ↓
  Sprint Planning
  (select items for next sprint)
         ↓
  SPRINT BACKLOG
  (items committed for this sprint)
         ↓
  ┌────────────────────────────────┐
  │  SPRINT (2–4 weeks)           │
  │                                │
  │  Daily Standups (15 min)      │
  │  Development + Testing         │
  │  Sprint Review                 │
  │  Sprint Retrospective          │
  └────────────────────────────────┘
         ↓
  POTENTIALLY SHIPPABLE INCREMENT
  (Working software at end of sprint)
```

**Scrum Roles:**

| Role | Responsibility |
|------|---------------|
| Product Owner | Owns the backlog, prioritizes features, represents the business |
| Scrum Master | Removes blockers, facilitates ceremonies, protects the team |
| Development Team | Cross-functional team that builds the product |

**Scrum Artifacts:**

| Artifact | Description |
|----------|-------------|
| Product Backlog | Ordered list of all desired features (written as user stories) |
| Sprint Backlog | Subset of backlog committed to for the current sprint |
| Increment | Working product produced at end of each sprint |
| Definition of Done (DoD) | Checklist that defines "done" (unit tested, reviewed, deployed to staging, docs updated) |

### Kanban

A visual workflow management method:

```
┌──────────┬──────────┬──────────┬──────────┐
│  BACKLOG │  TO DO   │  IN PROG │   DONE   │
│          │          │ (WIP: 3) │          │
├──────────┼──────────┼──────────┼──────────┤
│  Card A  │  Card D  │  Card F  │  Card H  │
│  Card B  │  Card E  │  Card G  │  Card I  │
│  Card C  │          │          │  Card J  │
└──────────┴──────────┴──────────┴──────────┘
```

Key Kanban principle: **WIP Limits** — you limit how many items can be "In Progress" at once to reduce multitasking and increase throughput.

### Agile Limitations that DevOps addresses

- Agile sprint ends with "code is ready" — but deployment is still a separate, slow process
- QA happens inside the sprint but deployment can still take weeks
- Agile didn't address infrastructure, operations, or monitoring
- "Done" in Agile often means "done in staging" — not "in users' hands"

---

## 9. DevOps Model — SDLC Evolved

DevOps takes the Agile sprint and extends it all the way through deployment and operations.

```
AGILE:
Plan → Code → Test → [DONE IN SPRINT] ← → → → Deploy (separately, slowly)

DEVOPS:
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ↑                                                              ↓
  └──────────────────── Continuous Feedback ─────────────────────┘
```

**Key differences DevOps adds to Agile SDLC:**

| Stage Added | What it brings |
|-------------|---------------|
| Build automation | Every commit triggers automated build — no manual build steps |
| Automated gate testing | Unit/integration/security tests run automatically; failures block release |
| Release automation | Versioning, changelog generation, artifact publishing automated |
| Deployment automation | CI/CD pipeline handles all deployments — no manual SSH and scripts |
| Operate phase | Infrastructure as Code — infrastructure is part of the sprint |
| Monitor phase | Dashboards, alerts, and SLOs are delivered as part of the feature |
| Feedback loop | Production metrics feed back into product backlog — data-driven prioritization |

---

## 10. SDLC Model Comparison

| Attribute | Waterfall | V-Model | Agile | DevOps |
|-----------|-----------|---------|-------|--------|
| Flexibility | Very Low | Low | High | Very High |
| Customer involvement | Beginning & end | Beginning & end | Continuous | Continuous |
| Release frequency | Once | Once | Per sprint (2–4 wks) | Continuous / daily |
| Testing | After development | Planned with each phase | Inside each sprint | Automated throughout |
| Risk | High (late feedback) | Medium | Low | Very Low |
| Team structure | Siloed | Siloed | Cross-functional | Cross-functional + Ops |
| Works best for | Stable requirements, regulated industries | Safety-critical systems | Most modern software | Most modern software |
| Feedback loops | Very slow (months) | Slow (months) | Sprint cadence (2-4 wks) | Real-time (hours) |
| Documentation | Extensive upfront | Extensive upfront | Lightweight, just enough | Automated (living docs) |

---

## 11. How DevOps Transforms Each SDLC Phase

| SDLC Phase | Traditional | DevOps |
|------------|------------|--------|
| **Planning** | Requirements frozen for months | Story grooming; requirements evolve; capacity planned with metrics |
| **Requirements** | 100-page BRD document | Lightweight user stories with testable acceptance criteria |
| **Design** | Big upfront architecture | Evolutionary architecture; ADRs tracked in Git |
| **Implementation** | Long-lived feature branches, big merges | Trunk-based development; feature flags; short-lived branches |
| **Testing** | Manual QA phase at end | Automated pyramid; testing in the pipeline; shift left |
| **Deployment** | Manual, scripted, once per quarter | Automated CI/CD pipeline; multiple deploys per day |
| **Operations** | Ops team alone; tribal knowledge | Shared runbooks; IaC; developer on-call; self-service |
| **Monitoring** | Ops-only dashboards; reactive | Shared dashboards; proactive alerting; blameless post-mortems |

---

## 12. Value Stream Mapping

**Value Stream Mapping (VSM)** is a Lean technique borrowed by DevOps to identify waste in the software delivery process.

**How to do it:**
1. Map every step from "feature idea" to "feature live in production"
2. Record the time each step takes
3. Identify value-added steps vs wait time / waste

**Example VSM (before DevOps):**

```
Feature Request  →  Backlog Grooming  →  Development  →  Code Review  →  QA  →  Release Approval  →  Deployment
   3 days wait        2 days               5 days          2 days       7 days     10 days             2 days

Total lead time: ~31 days
Value-added time: ~7 days (development + deployment)
WASTE: ~24 days (waiting)
```

**After DevOps:**

```
Feature Request  →  Backlog Grooming  →  Development  →  PR Review  →  CI Pipeline  →  Auto-Deploy
   1 day              0.5 days             3 days         4 hours       30 minutes       10 minutes

Total lead time: ~5 days
Value-added time: ~4 days
```

**Types of waste to eliminate (the 8 Lean Wastes):**

| Waste | Software Example |
|-------|-----------------|
| Defects | Bugs found in production |
| Overproduction | Features built that no one uses |
| Waiting | Code waiting in review queue; environment waiting to be provisioned |
| Non-utilized talent | Developer spending 4 hrs/day on manual deployments |
| Transportation | Handing tickets between teams |
| Inventory | Unfinished features in sprint backlog |
| Motion | Context switching between many tasks |
| Extra-processing | Manual approval gates; compliance reports generated by hand |

---

## 13. Summary

| Concept | Key Takeaway |
|---------|-------------|
| SDLC | The structured process for building software — every team uses some version of it |
| Waterfall | Sequential phases; good for stable requirements; fails in fast-changing environments |
| Agile | Iterative sprints; customer collaboration; revolutionized development but not operations |
| DevOps SDLC | Extends Agile to include Build, Deploy, Operate, Monitor phases with automation |
| Value Stream | Map the flow from idea to production; eliminate wait time |
| Shift Left | Testing and security as early as possible = cheaper, faster feedback |
| Continuous integration | Coding phase extended by automation |
| The real insight | DevOps isn't a new SDLC — it's what happens when you apply Lean and Agile principles to the entire software value stream, including infrastructure and operations |

---

**Previous:** [01 — What is DevOps?](01-what-is-devops.md)
**Next:** [03 — DevOps Lifecycle](03-devops-lifecycle.md)
