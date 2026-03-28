# Interview Questions — DevOps Fundamentals

> **How to use this file:** Go through each question and write your answer before looking at the model answer. Your answer won't be word-for-word identical — that's fine. Focus on hitting the key concepts. Practice answering out loud — interviews are verbal, not written.

---

## Question Categories

- [Conceptual / Behavioral](#conceptual--behavioral-questions) — "Tell me about..."
- [Technical Definitions](#technical-definition-questions) — "What is..."
- [Comparison Questions](#comparison-questions) — "What's the difference between..."
- [Scenario Questions](#scenario-questions) — "How would you..."
- [Culture & Process](#culture--process-questions)
- [DORA Metrics & Measurement](#dora-metrics--measurement-questions)
- [Toolchain](#toolchain-questions)
- [Rapid Fire](#rapid-fire-questions)

---

## Conceptual / Behavioral Questions

---

### Q1. Explain DevOps to a non-technical manager in 2 minutes.

**Model Answer:**

"Traditionally, the team that writes software and the team that deploys and runs it are completely separate. The developers race to ship new features as fast as possible, while the operations team's job is to keep systems stable. These goals clash — more change means more risk.

DevOps is the culture, practices, and tools that unite those two groups into one accountable team. Instead of handing code over a wall, one team owns the entire lifecycle: building the feature, testing it automatically, deploying it safely, and monitoring it in production.

The result is that software teams at companies like Amazon and Netflix can deploy code hundreds of times per day — with lower error rates than companies that deploy once a quarter. The key is automation: every manual step is replaced by a reliable machine. And the cultural change: when the team that writes code is also the team that gets paged when it breaks at 3 AM, they write much more careful code."

---

### Q2. Why did you want to get into DevOps?

**Framework to structure your answer (adapt to your story):**

1. Context: What were you doing before?
2. Pain: What problem did you see that DevOps solves?
3. Discovery: When/how did you learn about DevOps practices?
4. Motivation: What excites you about the combination of development and operations?
5. Action: What have you done to build these skills?

---

### Q3. What does a typical day look like for a DevOps engineer?

**Model Answer:**

"A DevOps engineer's day varies significantly, but common activities include:

**Morning:**
- Check monitoring dashboards and overnight alerts
- Triage any incidents from the previous night in Slack
- Review infrastructure cost report or security scan summary
- Daily standup with the cross-functional team

**Core work (varies by focus):**
- Write or review Terraform for new infrastructure
- Update or debug CI/CD pipelines
- Respond to developer requests ("my pipeline is failing")
- Work on automation to reduce toil
- Conduct or participate in incident post-mortems
- Update runbooks
- Security patching and vulnerability remediation
- On-call duties (if in rotation)

**Collaborative work:**
- Architecture discussions for upcoming features
- Planning sprint work for the next cycle
- Helping developers debug production issues

The balance between reactive work (incidents, tickets) and proactive work (automation, reliability improvements) is something high-performing teams actively manage — aiming for < 50% reactive so there's time to actually improve things."

---

### Q4. Tell me about a time you improved a deployment process.

**STAR Framework (Situation, Task, Action, Result):**

Structure your personal experience story using STAR. Key elements to include:
- What the problem was (slow, manual, risky, failures)
- What your specific contribution was
- The technical approach (what tools, what automation)
- The measurable outcome (deploy time reduced from X to Y, deployments per week from A to B)

**Example story structure:**

> "At [Company], deployments took 2 hours involving 6 people and a shared runbook with 40 manual steps. Failed deployments required a 4-hour rollback procedure. My task was to reduce deployment risk and time.
>
> I built a GitHub Actions pipeline that ran our test suite, built a Docker image, pushed to ECR, and deployed to EKS using Helm. I implemented blue-green deployment so rollback was instant. I parameterized the pipeline so any engineer could trigger a deployment with one button.
>
> Result: Deployment time went from 2 hours with 6 people to 12 minutes fully automated. Rollback time went from 4 hours to 3 minutes. We went from 2 deploys per month to 15 deploys per week. We reclaimed ~80 engineering-hours per month previously spent on manual deployments."

---

### Q5. What is the biggest challenge in a DevOps transformation?

**Model Answer:**

"The technology is the easy part. The hardest part is cultural change.

The most common failure mode I've seen: a company buys Kubernetes, Jenkins, and Terraform, and declares itself 'DevOps.' But developers still throw code over a wall to the ops team. Post-mortems still result in blame. Deployments are still monthly because 'that's how we've always done it.'

Specific challenges:
1. **Resistance from Ops engineers** who fear automation will replace their jobs — addressing this means showing that they level up to more valuable platform/SRE work
2. **Developers who don't want to be on-call** — this requires a gradual culture shift and making on-call rotation manageable, not a death march
3. **Management that measures the wrong things** — optimizing for 'tickets closed' rather than 'deployment frequency' or 'MTTR'
4. **Breaking legacy processes** — Change Advisory Boards, release windows, and compliance gates that were built for a waterfall world
5. **Psychological safety deficits** — without it, people won't be honest in post-mortems, won't raise concerns early, and DevOps fails silently

The teams that succeed start with a willing pilot team, show measurable results, and build momentum from proof of value — not by mandating tool adoption."

---

## Technical Definition Questions

---

### Q6. What is Continuous Integration (CI)?

**Model Answer:**

"Continuous Integration is the practice of developers merging their code changes into a shared branch frequently — ideally multiple times per day — where each merge automatically triggers a build and test suite.

The key word is 'continuously' — you don't accumulate work for weeks and then integrate. You integrate constantly. The automated pipeline verifies every change immediately.

Benefits:
- Catch integration conflicts early, when they're cheap to fix
- No 'big bang' merge at the end of a sprint
- Fast feedback — developer knows within minutes if they broke something
- The codebase is always in a known, tested state

The pipeline typically runs: lint → unit tests → integration tests → build artifact → security scan → post results back to the PR."

---

### Q7. What's the difference between Continuous Delivery and Continuous Deployment?

**Model Answer:**

"Both involve automatically deploying code after CI passes — the difference is the final step.

**Continuous Delivery:** The pipeline automatically deploys to staging. Every change is verified to be deployable. But a human makes the final decision to deploy to production. The deployment button is always ready to push — pushing it is a business decision, not a technical event.

**Continuous Deployment:** Every change that passes the full automated pipeline is automatically deployed to production with zero human gate. This requires extremely high confidence in the test suite, monitoring, and rollback capabilities.

Most mature organizations practice Continuous Delivery with automated production deployments. Pure Continuous Deployment is used by companies like Netflix, Facebook, and Etsy where the test coverage and monitoring confidence justifies it.

A key phrase: 'Continuous Delivery means you CAN deploy at any time. Continuous Deployment means you DO deploy every time.'"

---

### Q8. What is Infrastructure as Code (IaC)?

**Model Answer:**

"Infrastructure as Code means managing your servers, networks, databases, and cloud resources using version-controlled code files — treated with the same discipline as application code.

Instead of clicking through the AWS console to create an EC2 instance (which is manual, undocumented, and impossible to reproduce exactly), you write a Terraform file that declares 'I want an EC2 instance of type t3.micro in us-east-1 with these security groups.' You commit that to Git, review it as a pull request, run it through a CI pipeline that validates it, and apply it automatically.

Benefits:
- **Reproducible:** Run the same code and get the exact same infrastructure
- **Version controlled:** Git history shows every infrastructure change, who made it, and why
- **Reviewable:** Infrastructure changes go through code review like application changes
- **Testable:** You can validate infrastructure configs before applying them
- **Disaster recovery:** Entire infrastructure can be recreated from code after a catastrophic failure

Primary tools: Terraform (multi-cloud, most popular), AWS CloudFormation (AWS-native), Pulumi (real programming languages), Ansible (configuration, not just provisioning)."

---

### Q9. What is a Docker container and how is it different from a VM?

**Model Answer:**

"A Docker container is an isolated, lightweight process that packages an application with all its dependencies — code, runtime, libraries, and config — into a single runnable unit.

The key difference from a Virtual Machine:

```
VIRTUAL MACHINE:                  CONTAINER:
  ┌─────────────────────┐          ┌─────────────────────┐
  │  App A + all deps   │          │  App A + libs       │
  │  Guest OS (2GB)     │          └─────────────────────┘
  │  Hypervisor         │          ┌─────────────────────┐
  │  Host OS            │          │  App B + libs       │
  │  Hardware           │          └─────────────────────┘
  └─────────────────────┘          Container Engine
                                   Host OS (shared)
                                   Hardware
```

VMs virtualize the entire machine — each one includes a complete guest OS. Containers share the host OS kernel — they're isolated processes, not separate machines.

This makes containers:
- **Lighter:** Megabytes, not gigabytes
- **Faster:** Seconds to start, not minutes
- **More portable:** Same container runs on your laptop, CI runner, and production K8s cluster
- **More efficient:** Run 10× more workloads on the same hardware

VMs are still valuable for full isolation, different OS requirements, and bare-metal-like workloads."

---

### Q10. What is Kubernetes and what problems does it solve?

**Model Answer:**

"Kubernetes (K8s) is an open-source container orchestration system that automates deployment, scaling, and management of containerized applications across a cluster of machines.

The problems it solves:

**Problem 1 — Scale:** 'Our app needs to run on 100 machines.' Without K8s, you'd manually SSH to 100 machines. K8s schedules containers automatically across the entire fleet.

**Problem 2 — Self-healing:** 'Container crashed.' K8s detects the crash and restarts it automatically. If a node dies, K8s reschedules the pods elsewhere.

**Problem 3 — Rolling deployments:** 'Deploy new version without downtime.' K8s rolling update replaces pods one at a time, keeping the service running.

**Problem 4 — Load balancing:** 'Route traffic across 20 replicas.' K8s Service provides a stable IP that load-balances across all healthy pods.

**Problem 5 — Scaling:** 'Traffic spiked 10×.' HPA (HorizontalPodAutoscaler) detects CPU crossing a threshold and adds more replicas automatically.

**Problem 6 — Config & secrets management:** Kubernetes ConfigMaps and Secrets decouple configuration from the container image.

It's the operating system for the cloud — it abstracts the underlying machines and gives you a single API to manage distributed workloads."

---

### Q11. What is GitOps?

**Model Answer:**

"GitOps is an operational model where Git is the single source of truth for the entire system state — including infrastructure and application configuration. Every change to the system goes through a Git pull request. An automated controller continuously reconciles the live cluster state to match what's declared in Git.

Key principles:
1. **Declarative** — Describe desired state (not steps to get there)
2. **Everything in Git** — Code, Kubernetes manifests, Helm values, Terraform — all versioned
3. **Automated sync** — ArgoCD or Flux watches Git and applies changes automatically
4. **Observable** — Drift (manual changes to the cluster not in Git) is detected immediately

Benefits:
- Full audit trail: Git history is a complete record of every infrastructure change
- Self-documenting: The state of your system is always readable as code
- Easy rollback: `git revert` undoes any change including infrastructure
- Consistency: No manual kubectl apply in production — ever

ArgoCD and Flux are the dominant GitOps operators for Kubernetes."

---

### Q12. What is the difference between DevOps and SRE?

(See [01-what-is-devops.md](01-what-is-devops.md) section 8 for the detailed reference)

**30-second answer:**

"DevOps is the philosophy — a culture of collaboration between development and operations, backed by automation and continuous improvement. SRE (Site Reliability Engineering) is Google's specific, concrete implementation of that philosophy. Where DevOps says 'devs and ops should work closely together,' SRE says 'here is exactly how you do that: error budgets, SLOs, toil thresholds, and embedding software engineers in operations roles.'

You can think of it as: DevOps is the destination, SRE is one proven GPS route to get there."

---

### Q13. What is an SLO, SLA, and SLI? How do they relate?

**Model Answer:**

"These three define the reliability contract for a service:

**SLI (Service Level Indicator):** A quantitative measurement of service behavior. It's a metric.
- Example: 'The percentage of HTTP requests that returned a 2xx response in the last 30 days'
- Example: 'The 99th percentile latency of search requests'

**SLO (Service Level Objective):** A target for that measurement. The goal your team commits to internally.
- Example: 'SLI >= 99.9%' (99.9% of requests must succeed)
- Example: 'p99 latency < 500ms'
- The SLO defines your error budget: if SLO is 99.9%, you have 0.1% to 'spend' on incidents (43.8 min/month of downtime)

**SLA (Service Level Agreement):** A legal contract between you and your customers with financial consequences if the SLO is breached.
- Example: 'We guarantee 99.5% uptime. If we miss this, customers receive credit on their bill.'
- The SLO should be tighter than the SLA — so you catch violations internally before they breach the customer contract.

**Relationship:**
```
SLI measures → SLO is the target for SLI → SLA is the public commitment backed by SLO
```

The error budget (100% - SLO target) is used to make deployment decisions: if you've used your entire error budget for the month, you stop feature deployments and focus on reliability."

---

## Comparison Questions

---

### Q14. What's the difference between Agile and DevOps?

**Model Answer:**

"Agile and DevOps are complementary, not competing.

Agile solved the problem: 'How do we build software in a more collaborative, adaptive way?' It introduced sprints, user stories, and iterative development. Agile stops at 'code is ready to ship.'

DevOps solved the next problem: 'The code is ready, but it takes 6 weeks to deploy it.' DevOps extends the Agile lifecycle through deployment, operations, and monitoring. It also applies the same iterative, collaborative principles to infrastructure.

Key difference: Agile is about how the development team works. DevOps is about how the development team AND the operations team work together throughout the full software lifecycle — including after it ships.

Without DevOps, an Agile team can write sprints of code that pile up waiting for a quarterly deployment window. DevOps closes that gap."

---

### Q15. What's the difference between a container and an image?

**Model Answer:**

"A Docker image is a static, immutable snapshot — it's the blueprint. Think of it like a class definition.

A Docker container is a running instance of that image — it has a process, filesystem, network interface, and state. Think of it like an object instantiated from a class.

```
Docker image  →  like a class definition (blueprint)
Docker container  →  like an object instance (running process)
```

You can run 10 containers from the same image simultaneously. Stopping and deleting a container doesn't affect the image. Changing the image requires rebuilding it — the container running from the old image continues with the old version until replaced."

---

### Q16. What's the difference between Terraform and Ansible?

**Model Answer:**

"They solve different problems, and are often used together:

**Terraform** is an Infrastructure Provisioning tool. It creates cloud resources — EC2 instances, VPCs, databases, DNS records, Kubernetes clusters. It's declarative: you describe what you want in HCL and Terraform figures out how to create it. It maintains state (knows what it already created) and can calculate diffs.

**Ansible** is a Configuration Management tool. It configures servers after they exist — installs software, copies config files, starts services, manages users. It connects via SSH to existing machines and runs tasks in order. It's procedural/idempotent: run the same playbook 10 times and the result is the same.

**Together:** Terraform creates the EC2 instance → Ansible configures the software on it.

**The shift in cloud-native:** In Kubernetes environments, Ansible is less needed because containers bake the configuration into the image (immutable infrastructure). Ansible is still essential for VM-based workloads and bare-metal servers."

---

### Q17. What is the difference between monolithic and microservices architecture from a DevOps perspective?

**Model Answer:**

"A monolith is one large application — all functionality in one deployable unit. Microservices split functionality across many small, independently deployable services.

**From a DevOps perspective:**

| Concern | Monolith | Microservices |
|---------|---------|--------------|
| CI/CD complexity | One pipeline | Many pipelines (one per service) |
| Deployment risk | One big deploy | Small, targeted deploys |
| Testing | Comprehensive suite in one place | Integration testing between services is harder |
| Rollback | Roll back entire app | Roll back one service independently |
| Scaling | Scale whole app | Scale only the bottleneck service |
| Observability | Single log stream | Need distributed tracing across services |
| Team ownership | Hard to assign clear ownership | Each team owns their service(s) |

**The DevOps recommendation:** Don't start with microservices. Start monolithic. Extract services when team ownership, scaling, or deployment frequency demands it. Microservices have 10× the operational complexity of a monolith — only worthwhile when the monolith creates more pain."

---

### Q18. What is the difference between push-based and pull-based deployment?

**Model Answer:**

"In a **push-based** deployment, the CI/CD system actively sends (pushes) the new version to the target environment. Example: GitHub Actions runs `kubectl apply` or `helm upgrade` to deploy to Kubernetes. The pipeline has write access to the cluster.

In a **pull-based** deployment (GitOps), an agent running inside the target environment continuously watches a Git repository and pulls changes down when it detects a difference. Example: ArgoCD runs inside the cluster, watches a Git repo, and applies new manifests when they're merged to main. The pipeline never touches the cluster directly.

**Pull-based (GitOps) advantages:**
- CI/CD system doesn't need cluster credentials (security improvement)
- Cluster is self-healing — if someone manually changes something, the agent reconciles back to Git
- Audit trail is Git history — complete record of every deployment
- Works in air-gapped environments where CI can't reach the cluster

**When to use push:** Simple setups, non-Kubernetes environments, when you need more control over the exact deployment sequence."

---

## Scenario Questions

---

### Q19. Production is down. Walk me through how you respond.

**Model Answer:**

"**First 5 minutes:**
1. Acknowledge the alert/page immediately — let the team know someone is on it
2. Pull up monitoring dashboards — error rate, latency, traffic (the four golden signals)
3. Check what changed recently: recent deployments, config changes, infrastructure changes
4. Assess impact: What % of users are affected? What functionality is broken?

**Next 10 minutes:**
5. If a recent deployment is the likely culprit → rollback immediately. Mitigating user impact is more important than finding root cause
6. If no obvious cause → dig into logs and traces. Look at the service that's returning errors, trace back to upstream dependencies
7. Escalate to appropriate people if the issue crosses multiple services or requires subject matter expertise

**During the incident:**
8. Keep a running log of actions taken and findings (in Slack, Google Doc, or incident tool)
9. Communicate status updates every 15–30 minutes to stakeholders
10. Focus on mitigation first, then root cause — users don't care why it's broken, they care when it's fixed

**After resolution:**
11. Write a blameless post-mortem within 24–48 hours
12. Track action items to completion — don't let post-mortems be write-only exercises"

---

### Q20. How would you set up a CI/CD pipeline from scratch for a new microservice?

**Model Answer:**

"I'd approach this in layers:

**Layer 1 — Source setup:**
- Create the Git repo with branch protection on main (require PRs + CI passing)
- Add CODEOWNERS to auto-assign reviewers
- Add `.gitignore` and pre-commit hooks (linting, formatting)
- Add a `Makefile` or `justfile` for local dev commands

**Layer 2 — CI Pipeline (GitHub Actions):**
- Trigger: on push to any branch + on PR
- Steps: checkout → install deps → lint → unit tests → integration tests → SAST scan → dependency scan → build Docker image → image scan (Trivy) → if on main: push to staging registry

**Layer 3 — CD to Staging:**
- On merge to main: promote image, tag with git SHA
- Deploy to staging namespace in Kubernetes using Helm or Kustomize
- Run smoke tests against staging
- Post result to Slack

**Layer 4 — Production promotion:**
- Either manual approval gate (CD) or automatic after smoke tests pass (Continuous Deployment)
- Canary deployment: 5% → 25% → 100% with monitoring gates between steps
- Auto-rollback if error rate or latency spikes above threshold

**Layer 5 — Observability:**
- All metrics emitted and scraped by Prometheus
- Dashboard created in Grafana for the new service (stored as code)
- Alerts configured for error rate, latency SLO
- Runbook written and linked in the alert

The whole thing lives as code in the repo — nobody runs manual deployments."

---

### Q21. How do you handle secrets in a CI/CD pipeline?

**Model Answer:**

"Never store secrets in code or CI/CD YAML. Here's the layered approach I use:

**Development:** Secrets in `.env` files that are `.gitignored`. Local HashiCorp Vault Dev server for dynamic secrets.

**CI Pipeline:**
- Short-lived secrets: stored in the CI/CD platform's secrets store (GitHub Actions Encrypted Secrets, GitLab CI Variables). These are masked from logs automatically.
- For accessing cloud resources: use short-lived credentials via OIDC federation. For AWS: GitHub Actions OIDC → assume IAM role → short-lived STS credentials. No static access keys stored anywhere.

**Production:**
- Secrets in HashiCorp Vault or cloud-native SM (AWS Secrets Manager, GCP Secret Manager)
- Kubernetes External Secrets Operator syncs from Vault into K8s Secrets
- Vault provides dynamic secrets with short TTLs (database credentials expire in 1 hour)
- All secret access is audited (Vault audit log)

**What I never do:**
- Secrets in Docker images
- Secrets in environment variable definitions in Docker Compose committed to Git
- Sharing credentials over Slack
- Using the same credentials for all environments"

---

### Q22. A developer says 'it works on my machine but not in CI.' How do you fix this?

**Model Answer:**

"This is the classic Docker/parity problem. My approach:

**Immediate fix:** Reproduce the failure locally in the same environment as CI.
```bash
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:20-alpine \
  npm ci && npm test
```
If CI runs Node 20-alpine, I run Node 20-alpine. The dev likely has a different Node version, different OS, or different global dependencies.

**Common root causes:**
1. **Different language runtime versions** — Solution: Pin the exact version in the Dockerfile and use `.nvmrc`/`.tool-versions`/`.python-version`, and make CI use the same file
2. **Missing environment variables** — CI doesn't have a `.env` file. Solution: Check what envs CI provides vs what the test expects
3. **Different OS behavior** — File path separators, case sensitivity (macOS is case-insensitive, Linux is not), line endings (CRLF vs LF). Solution: Always develop using Docker to match the Linux CI environment
4. **External service dependencies** — CI doesn't have access to a DB or third-party service. Solution: Use `docker-compose` for CI, mock external services
5. **Non-deterministic test order** — Tests pass in isolation but fail in a specific order. Solution: Fix test isolation, use `--randomize-tests` locally too

**Long-term fix:** Add a `make test` target that runs tests inside Docker with the exact same image as CI. If that passes locally, CI will pass."

---

## Culture & Process Questions

---

### Q23. What is a blameless post-mortem and why does it matter?

(Answer based on [04-culture-and-mindset.md](04-culture-and-mindset.md) section 4)

**Key points to hit:**
- Assumes systems cause failures, not individuals
- Documents: timeline, root cause (5 Whys), what went well, what could improve, action items with owners
- Creates organizational learning — same incident shouldn't happen twice
- Without blamelessness: engineers hide mistakes → problems recur
- Action items fix systems, not scapegoat individuals

---

### Q24. How do you handle pushback from Operations teams resistant to DevOps changes?

**Model Answer:**

"Resistance from Ops is usually rational — they have real concerns.

**First, understand the concern:**
- 'Will automation replace my job?' → Address directly: 'Automation handles the toil. You level up to platform engineering and architecture — more valuable, more interesting work.'
- 'We'll lose control of what gets deployed.' → Show them: GitOps means everything goes through Git and is fully auditable. More control, not less.
- 'What if the automation breaks?' → Start with non-production environments. Build trust incrementally.

**Then, involve them:**
- The worst thing you can do is build the CI/CD platform without them and hand it to them. Build it WITH them. Ops engineers have invaluable production knowledge.
- Let them set the initial guardrails — deployment windows, approval requirements, test criteria.
- Make them the heroes of the transformation — they own the platform.

**Show results early:**
- Start with a low-risk pilot (internal tool, non-revenue service)
- Measure and communicate: 'Our team went from 2 deployments per month to 20. Zero production incidents during the transition.'
- Once results are visible, Ops engineers often become the strongest advocates.

The goal is to make their jobs better, not to eliminate them."

---

### Q25. What's the difference between reactive and proactive operations?

**Model Answer:**

"**Reactive operations:** You find out something is wrong from a user report, a customer complaint, or an alert that fires after users are already impacted.

**Proactive operations:** You discover and resolve problems before users are impacted.

DevOps practices that enable proactive operations:
- **Synthetic monitoring:** Automated tests that constantly probe your service from the user's perspective — you see the failure before the user does
- **SLO burn rate alerts:** Instead of alerting when the service is down, alert when you're burning through your error budget faster than sustainable
- **Chaos engineering:** Deliberately inject failures to find weaknesses in a controlled environment
- **Capacity planning:** Predict traffic growth and provision headroom before hitting limits
- **Dependency vulnerability scanning:** Detect CVEs in your dependencies before they're exploited
- **Performance regression testing:** Catch slower p99 latency in staging before it reaches production

The goal is to move from a 'firefighting' posture (reactive) to an 'engineering' posture (proactive) — spending time building resilience rather than responding to crises."

---

## DORA Metrics & Measurement Questions

---

### Q26. What are the DORA metrics? Why are they important?

**Model Answer:**

"DORA stands for DevOps Research and Assessment. They identified 4 metrics that predict both software delivery performance and organizational performance:

**1. Deployment Frequency:** How often you deploy to production.
- Elite: multiple times per day
- Tells you: batch size, pipeline health, team agility

**2. Lead Time for Changes:** How long from code commit to running in production.
- Elite: less than 1 hour
- Tells you: pipeline efficiency, approval bottlenecks, batch size

**3. Change Failure Rate:** % of deployments that cause a production incident or require rollback.
- Elite: 0-15%
- Tells you: testing quality, deployment safety, risk management

**4. Mean Time to Restore (MTTR):** How long to recover from a production incident.
- Elite: less than 1 hour
- Tells you: monitoring quality, runbook completeness, team incident readiness

Why they matter: These are the metrics proven through academic research correlating with business outcomes. Elite performers on DORA 2× more likely to exceed company performance and profitability goals. They're not vanity metrics — they're leading indicators of engineering and business health."

---

### Q27. How do you improve deployment frequency?

**Model Answer:**

"Deployment frequency is blocked by either technical or cultural barriers:

**Technical barriers and solutions:**
- Manual deployments → Build fully automated CI/CD pipeline
- Long test suites → Parallelize tests, optimize slow tests, defer non-blocking tests
- Complex release approval process → Automate approval criteria (passing test suite = approved)
- Long deployment time → Optimize pipeline, use caching, parallelize steps
- Fear of breaking production → Add more automated testing, canary deployments, feature flags, monitoring

**Cultural barriers and solutions:**
- 'We only deploy on Fridays/Tuesdays' → Any day is safe if you have proper automation and monitoring
- 'Big bang releases' mentality → Shift to small, incremental changes with feature flags
- Risk aversion from past incidents → Build confidence gradually, start with non-critical services
- Manual change approval boards → Replace with automated risk-scoring and automated gates

**The compound effect:** Small deploys (5-10 changed lines) have orders of magnitude lower risk than large deploys (5000 changed lines). When deployments become boring routine rather than high-risk events, frequency increases naturally."

---

## Toolchain Questions

---

### Q28. What CI/CD tool do you prefer and why?

**Framework for answering (be honest about your experience, but show you understand tradeoffs):**

**GitHub Actions answer:**
"If the codebase is on GitHub, GitHub Actions is my first choice. The integration is seamless — workflows live in the same repo as the code, the Marketplace has pre-built actions for almost everything, and the OIDC integration with AWS/GCP/Azure eliminates static credentials. The YAML syntax is intuitive and the matrix build feature is excellent for testing across multiple versions. It's my default choice for cloud-native stacks.

The limitation is vendor lock-in to GitHub and cost at scale for private repos. For teams needing more control, self-hosted runners help, but Jenkins or GitLab CI might be better for full on-premises needs."

---

### Q29. Terraform `plan` shows 150 resources will be destroyed. What do you do?

**Model Answer:**

"Stop and investigate before applying. 150 resource destructions is almost certainly not intended.

**Immediate steps:**
1. Don't run `terraform apply` — read the `plan` output carefully
2. Look for patterns in what's being destroyed
3. Check recent changes to the Terraform code or variables
4. Check if the state file was manipulated (was it accidentally deleted or overwritten?)

**Common causes:**
- A refactor changed resource names (Terraform treats as destroy+create, not rename)
- A module was removed from the config
- A variable that controls count/for_each was accidentally set to zero
- Wrong workspace or wrong backend — you're planning against a different environment's state
- State file mismatch — remote state out of sync with actual resources

**Solutions:**
- `terraform state mv` to rename resources in state without destroying them
- `terraform import` to import existing resources into state
- Use `-target` to apply only specific resources if you need to make progress carefully
- Review the Git diff — what changed in the Terraform code that would cause this?

**Lesson:** Always run `terraform plan` in CI, treat unexpected destructions as blocking issues, and require peer review for any plan that shows > X destructive changes."

---

### Q30. What is Helm and why is it used?

**Model Answer:**

"Helm is the package manager for Kubernetes. Just as `apt` or `npm` manages software packages, Helm manages Kubernetes application packages called Charts.

**The problem without Helm:**
A real application might need a Deployment, Service, HorizontalPodAutoscaler, Ingress, ConfigMap, ServiceAccount — that's 6+ YAML files. Each environment (dev, staging, prod) needs slightly different values. You'd end up with dozens of YAML files with small differences that are hard to maintain.

**What Helm provides:**
- **Charts:** Bundled collection of Kubernetes manifests + default values
- **Templating:** Use values files (`values.yaml`) to customize the same chart for different environments
- **Versioning:** Charts are versioned and released to registries (ArtifactHub, Harbor)
- **Release management:** `helm install`, `helm upgrade`, `helm rollback` with history
- **Community charts:** Pre-built charts for Postgres, Redis, Nginx, Prometheus — deploy them in seconds

```bash
# Deploy official Nginx ingress controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx

# Deploy your app with environment-specific values
helm install myapp ./charts/myapp -f values/production.yaml
helm upgrade myapp ./charts/myapp -f values/production.yaml
helm rollback myapp 2   # Roll back to revision 2
```"

---

## Rapid Fire Questions

Quick 1-2 sentence answers:

---

**Q31. What is idempotency?**
"An operation is idempotent if running it multiple times produces the same result as running it once. In DevOps, this is critical — running a playbook or Terraform config 10 times should not create 10 copies of everything."

---

**Q32. What is a microservice?**
"A small, independently deployable service that owns a single bounded domain (e.g., 'authentication service', 'payment service'). Each has its own codebase, CI/CD pipeline, and often its own database."

---

**Q33. What is the 12-Factor App?**
"A methodology for building cloud-native applications across 12 principles: use environment variables for config, store nothing stateful on the server, run as stateless processes, export logs as event streams, treat backing services as attached resources, etc. The goal is maximum portability and scalability."

---

**Q34. What is immutable infrastructure?**
"Servers/containers are never modified after deployment. Instead of SSHing in to patch a server, you build a new image with the patch and replace the old server. This eliminates configuration drift and makes environments reproducible."

---

**Q35. What is infrastructure drift?**
"When the actual state of infrastructure diverges from the desired state declared in code. Example: someone manually added a security group rule in the AWS console that isn't in Terraform. Drift leads to inconsistency, security gaps, and eventually incidents."

---

**Q36. What is a feature flag?**
"A configuration mechanism that allows you to enable or disable features at runtime without deploying new code. Used to: deploy code to production without activating it, gradually roll out to a percentage of users, run A/B tests, and kill a failing feature instantly without a rollback."

---

**Q37. What is canary deployment?**
"A deployment strategy where the new version initially receives only a small percentage of traffic (e.g., 1%), while the rest stays on the old version. Traffic is shifted gradually after verifying the new version is healthy. Named after the 'canary in a coal mine' — a small test group that would detect problems before majority of users see them."

---

**Q38. What does 'shift left' mean?**
"Moving activities — especially testing and security — earlier (to the left) in the development pipeline. Instead of testing only before release, you test at every commit. Instead of security audits pre-release, you run SAST on every PR. Earlier detection means cheaper fixes and faster delivery."

---

**Q39. What is MTTR?**
"Mean Time to Restore — the average time it takes to restore service after a production incident. DORA Elite: less than 1 hour. It measures your organization's ability to detect, respond, and recover from failures."

---

**Q40. What is toil in SRE?**
"Manual, repetitive, tactical work that grows linearly with service scale and provides no enduring value. SRE principle: keep toil below 50% of working time. Above that threshold, the excess becomes a priority for automation."

---

**Q41. What is a runbook?**
"A documented set of procedures for handling a specific operational task or incident. A runbook for 'database failover' tells the on-call engineer step-by-step what to do — checks to make, commands to run, escalation paths. Good runbooks allow any engineer (not just 'Bob') to handle common incidents."

---

**Q42. What is the difference between observability and monitoring?**
"Monitoring tracks predefined known states (is the service up? is CPU below 80%?). You monitor things you know to look for. Observability is the ability to understand the internal state of a system from its external outputs (logs, metrics, traces). With observability, you can investigate unknown, novel failure modes you've never seen before — not just known conditions."

---

**Q43. What is blue-green deployment?**
"Maintaining two identical production environments (Blue = current, Green = new version). Traffic switches from Blue to Green atomically. Rollback is instant — flip traffic back to Blue. Zero downtime. Two times the infrastructure cost during the transition."

---

**Q44. What is a service mesh?**
"An infrastructure layer that manages all service-to-service communication in a microservices architecture. Deployed as sidecar proxies alongside each service. Provides mTLS, traffic routing, retries, circuit breaking, and automatic observability — without any changes to the application code."

---

**Q45. What is a Kubernetes Namespace?**
"A virtual cluster inside a Kubernetes cluster. Provides isolation between different teams, environments (dev/staging), or applications. Resources in different namespaces don't conflict with each other by name. RBAC can be scoped to namespaces — a developer can have full access to the `dev` namespace but read-only to `production`."

---

**Q46. What is RBAC in Kubernetes?**
"Role-Based Access Control. Defines what actions (get, list, create, delete) subjects (users, service accounts) can perform on what resources (pods, deployments, secrets) in which namespaces. Principle of least privilege: only grant the minimum permissions needed."

---

**Q47. What is a Kubernetes Operator?**
"An extension to Kubernetes that encodes operational knowledge into code. An Operator watches for custom resources (CRDs) and takes actions to reconcile the actual state to the desired state — just like the Kubernetes controller does for Deployments. Examples: the Prometheus Operator manages Prometheus instances, the cert-manager Operator manages TLS certificates."

---

**Q48. What is FinOps?**
"The practice of bringing financial accountability to cloud spending. In DevOps, this means engineers take ownership of cloud cost alongside performance and reliability. Key practices: right-sizing instances, using spot/preemptible instances, auto-scaling down in off-hours, tagging resources for cost allocation, setting budget alerts."

---

**Q49. What is a Webhook?**
"An HTTP callback — when an event happens in one system, it immediately sends an HTTP POST to a URL in another system. In DevOps: GitHub webhooks trigger CI pipelines on every push. PagerDuty webhooks can update a status page on incident creation. Webhooks are how most DevOps tools integrate in real time."

---

**Q50. What is the Principle of Least Privilege?**
"Every system, user, and process should have only the minimum permissions necessary to do its job — nothing more. In DevOps: a CI/CD pipeline should only have write access to the specific S3 bucket it deploys to, not all S3 buckets. An application's IAM role should only be able to read DynamoDB tables it uses, not write to EC2. Combined with short-lived credentials, this limits the blast radius of a compromise."

---

## Bonus: Common Interview Mistakes to Avoid

1. **Saying "DevOps is just CI/CD"** — It's much more. Culture, IaC, monitoring, security.

2. **Confusing Continuous Delivery and Continuous Deployment** — They're not the same. Know the difference.

3. **Saying "we need a DevOps team"** — Red flag. DevOps is a practice for all teams, not a separate silo.

4. **Not knowing DORA metrics** — Every DevOps job interview asks about these.

5. **Not having a strong answer on culture** — Technical skills matter, but hiring managers know culture makes DevOps work. Have examples.

6. **Saying Kubernetes is always the answer** — It's not. For a 2-person startup with a simple app, Docker Compose or a PaaS is the right answer. Show judgment.

7. **Confusing Docker images and containers** — Know the difference cold.

8. **Not being able to draw the CI/CD pipeline** — You will be asked to walk through a pipeline end-to-end. Practice this out loud.

---

**Back to module home:** [README.md](README.md)
