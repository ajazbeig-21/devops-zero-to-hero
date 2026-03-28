# 05 — DevOps Toolchain Overview

> "A DevOps engineer doesn't need to master every tool — they need to understand the purpose of every tool category, know the leading tools in each, and be able to learn new tools quickly."

---

## Table of Contents

1. [The DevOps Toolchain Map](#1-the-devops-toolchain-map)
2. [Category 1 — Version Control](#2-category-1--version-control)
3. [Category 2 — CI/CD Platforms](#3-category-2--cicd-platforms)
4. [Category 3 — Containerization](#4-category-3--containerization)
5. [Category 4 — Container Orchestration](#5-category-4--container-orchestration)
6. [Category 5 — Infrastructure as Code (IaC)](#6-category-5--infrastructure-as-code-iac)
7. [Category 6 — Configuration Management](#7-category-6--configuration-management)
8. [Category 7 — Cloud Platforms](#8-category-7--cloud-platforms)
9. [Category 8 — Monitoring & Observability](#9-category-8--monitoring--observability)
10. [Category 9 — Security & Compliance (DevSecOps)](#10-category-9--security--compliance-devsecops)
11. [Category 10 — Artifact & Registry Management](#11-category-10--artifact--registry-management)
12. [Category 11 — GitOps](#12-category-11--gitops)
13. [Category 12 — Service Mesh](#13-category-12--service-mesh)
14. [Category 13 — Secrets Management](#14-category-13--secrets-management)
15. [Category 14 — Collaboration & Planning](#15-category-14--collaboration--planning)
16. [Category 15 — Testing Tools](#16-category-15--testing-tools)
17. [Complete Toolchain at a Glance](#17-complete-toolchain-at-a-glance)
18. [Choosing the Right Tools](#18-choosing-the-right-tools)
19. [The Minimum Viable DevOps Stack](#19-the-minimum-viable-devops-stack)
20. [Summary](#20-summary)

---

## 1. The DevOps Toolchain Map

Each DevOps lifecycle stage maps to one or more tool categories:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DEVOPS TOOLCHAIN                           │
├─────────────┬───────────────────────────────────────────────────────┤
│  LIFECYCLE  │  TOOL CATEGORIES                                      │
│  STAGE      │                                                       │
├─────────────┼───────────────────────────────────────────────────────┤
│  Plan       │  Project Management, Collaboration                    │
├─────────────┼───────────────────────────────────────────────────────┤
│  Code       │  Version Control, IDEs, Static Analysis               │
├─────────────┼───────────────────────────────────────────────────────┤
│  Build      │  CI/CD Platforms, Build Tools, Containerization       │
├─────────────┼───────────────────────────────────────────────────────┤
│  Test       │  Testing Frameworks, Security Scanning                │
├─────────────┼───────────────────────────────────────────────────────┤
│  Release    │  Artifact Registries, Release Management              │
├─────────────┼───────────────────────────────────────────────────────┤
│  Deploy     │  CI/CD, GitOps, Orchestration, IaC                    │
├─────────────┼───────────────────────────────────────────────────────┤
│  Operate    │  Cloud Platforms, Config Management, Secrets Mgmt     │
├─────────────┼───────────────────────────────────────────────────────┤
│  Monitor    │  Monitoring & Observability Stack                     │
└─────────────┴───────────────────────────────────────────────────────┘
```

**Important principle:** The tools support the culture and practices. Never choose a tool first and build a culture around it.

---

## 2. Category 1 — Version Control

### Purpose

Track every change to code and infrastructure. Enable collaboration and provide the single source of truth.

### The Non-Negotiable: Git

Git is the industry-standard distributed version control system. Created by Linus Torvalds in 2005 for Linux kernel development.

**Core Git concepts:**

```bash
# Repository — the project container
git init                   # Create a new repo
git clone <url>            # Copy an existing repo

# Staging and Committing
git add .                  # Stage all changes
git commit -m "feat: add login page"  # Save staged changes

# Branching
git branch feature/login   # Create a branch
git checkout feature/login # Switch to branch
git switch feature/login   # Modern alternative

# Merging and Rebasing
git merge feature/login    # Merge branch into current
git rebase main            # Rebase current branch onto main

# Remote operations
git push origin main       # Push to remote
git pull origin main       # Fetch + merge from remote
git fetch                  # Fetch without merging

# Viewing history
git log --oneline --graph  # Visual branch history
git diff main              # Diff against main branch
```

**Important Git concepts for DevOps:**

| Concept | Why it matters |
|---------|---------------|
| `.gitignore` | Prevents secrets, build artifacts, and OS files from entering version control |
| Git hooks | Pre-commit, post-commit, pre-push scripts that enforce standards |
| Git tags | Mark release points — how versions like `v1.2.3` are created |
| Branch protection rules | Require PR reviews and CI checks before merging to main |
| CODEOWNERS | Auto-assign reviewers based on which files changed |
| Signed commits | Verify that commits came from a specific person (security) |

### Git Hosting Platforms

| Platform | Key Features | Best For |
|----------|-------------|---------|
| **GitHub** | GitHub Actions CI, GitHub Packages, Copilot, largest community | Open source, startups, most enterprises |
| **GitLab** | All-in-one (Git + CI/CD + registry + security scanning) | Self-hosted enterprises, full DevOps platform |
| **Bitbucket** | Jira integration, Atlassian ecosystem | Teams using Jira/Confluence/Trello |
| **Azure DevOps (ADO)** | Azure integration, Boards, Pipelines | Microsoft-stack enterprises |
| **AWS CodeCommit** | AWS-native, IAM integration | Strictly AWS environments (less popular) |

### GitOps — Version Control as the Source of Truth for Infrastructure

**GitOps principle:** Everything — application code AND infrastructure configuration — lives in Git. The cluster/environment is continuously reconciled to match what's in Git.

```
Developer pushes config change to Git repo
              ↓
  GitOps controller (ArgoCD/Flux) detects drift
              ↓
  Controller applies the change to Kubernetes cluster
              ↓
  Cluster state now matches Git state
```

---

## 3. Category 2 — CI/CD Platforms

### Purpose

Automate the journey from code commit to deployed artifact. The backbone of DevOps automation.

### GitHub Actions (Most popular as of 2024–2026)

Tight integration with GitHub. Workflow-as-code in YAML. Massive marketplace of pre-built actions.

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test -- --coverage

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          severity: 'HIGH,CRITICAL'
          exit-code: '1'

      - name: Push to registry
        run: |
          echo "${{ secrets.REGISTRY_PASSWORD }}" | \
            docker login myregistry.io -u ${{ secrets.REGISTRY_USER }} --password-stdin
          docker push myapp:${{ github.sha }}
```

**GitHub Actions key concepts:**

| Concept | Description |
|---------|-------------|
| Workflow | YAML file defining automation (lives in `.github/workflows/`) |
| Trigger (on:) | When the workflow runs — push, PR, schedule, manual |
| Job | A unit of work that runs on a runner |
| Step | A single command or Action within a job |
| Action | Reusable unit of automation (from Marketplace or custom) |
| Runner | The machine that executes jobs (GitHub-hosted or self-hosted) |
| Secrets | Encrypted values for credentials and tokens |
| Environments | Named deployment targets with protection rules and secrets |
| Matrix builds | Run a job across multiple versions/OS combinations |

---

### Jenkins

The original open-source CI/CD server. Highly flexible but requires significant maintenance.

```groovy
// Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'myregistry.io'
        IMAGE_TAG = "${GIT_COMMIT[0..7]}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm ci && npm test'
            }
            post {
                always {
                    junit 'test-results/*.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/myapp:${IMAGE_TAG} ."
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh "kubectl set image deployment/myapp myapp=${DOCKER_REGISTRY}/myapp:${IMAGE_TAG}"
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Deploy"
            }
            steps {
                sh "kubectl set image deployment/myapp myapp=${DOCKER_REGISTRY}/myapp:${IMAGE_TAG} -n production"
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#alerts', message: "Build failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
```

**Jenkins architecture:**

```
Jenkins Controller (Master)
        ↓ distributes jobs to
Jenkins Agents (Workers)
[Agent 1]  [Agent 2]  [Agent 3]  [Agent 4]
 Linux       macOS      Windows    ARM
```

**Jenkins ecosystem:**
- Jenkins X: Kubernetes-native Jenkins (modern replacement)
- Jenkins Configuration as Code (JCasC): YAML-defined Jenkins config
- 1800+ plugins for every integration imaginable

---

### GitLab CI/CD

All-in-one platform — version control, CI/CD, registry, security scanning, all built-in.

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - security
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

sast:
  stage: security
  # GitLab's built-in SAST — no config needed
  
container_scanning:
  stage: security
  # GitLab's built-in Trivy container scan

deploy-staging:
  stage: deploy
  environment: staging
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n staging
  only:
    - main

deploy-production:
  stage: deploy
  environment: production
  when: manual   # Human approval required
  script:
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n production
  only:
    - main
```

---

### CI/CD Platform Comparison

| Feature | GitHub Actions | GitLab CI/CD | Jenkins |
|---------|---------------|-------------|---------|
| Hosting | Cloud (+ self-hosted) | Cloud + self-hosted | Self-hosted |
| Configuration | YAML | YAML | Groovy (Jenkinsfile) |
| Built-in security scanning | Basic | Comprehensive | Via plugins |
| Built-in container registry | GitHub Packages | GitLab Registry | No |
| Learning curve | Low | Low-Medium | High |
| Cost | Free for public; paid for private | Free for public; paid tiers | Free (but infra costs) |
| Best for | GitHub repos, simplicity | Full DevOps platform | Legacy enterprise |
| Plugin ecosystem | Marketplace (10k+ actions) | Limited | 1800+ plugins |

**Other CI/CD tools worth knowing:**

| Tool | Description |
|------|-------------|
| **CircleCI** | Cloud-first, fast, strong parallelization |
| **Travis CI** | Historically popular for open source; declining |
| **Tekton** | Kubernetes-native CI/CD framework |
| **Drone CI** | Lightweight, Docker-based, self-hosted |
| **TeamCity** | JetBrains' CI server, popular in Java shops |
| **Buildkite** | Hybrid: SaaS orchestration + your own agents |
| **AWS CodePipeline** | AWS-native, integrates tightly with AWS services |

---

## 4. Category 3 — Containerization

### Purpose

Package applications with all dependencies into a portable, reproducible unit that runs consistently everywhere.

### Docker — The Industry Standard

**Core Docker concepts:**

```
DOCKERFILE → IMAGE → CONTAINER

Dockerfile: Blueprint (like a class definition)
Image:      Immutable snapshot built from Dockerfile (like a class)
Container:  Running instance of an image (like an object)
```

**Docker CLI essentials:**

```bash
# Images
docker build -t myapp:1.0 .          # Build image from Dockerfile
docker pull ubuntu:22.04             # Download image from registry
docker push myregistry/myapp:1.0     # Upload image to registry
docker images                        # List local images
docker rmi myapp:1.0                 # Remove an image

# Containers
docker run -d -p 8080:3000 \         # Run container in background
  --name myapp \                     # Name it
  -e DATABASE_URL=postgres://...     # Pass env var
  myapp:1.0

docker ps                            # List running containers
docker ps -a                         # List all containers (including stopped)
docker stop myapp                    # Stop a container
docker rm myapp                      # Remove a container
docker exec -it myapp bash           # Shell into a running container
docker logs -f myapp                 # Follow container logs

# Multi-service (Docker Compose)
docker compose up -d                 # Start all services
docker compose down                  # Stop all services
docker compose logs -f api           # Follow logs for specific service
```

**Docker Compose — Local development environment:**

```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - ./src:/app/src    # Hot reload in development
  
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5
  
  cache:
    image: redis:7-alpine
    
volumes:
  postgres_data:
```

**Container runtimes (beyond Docker):**

| Runtime | Description |
|---------|-------------|
| **containerd** | Industry-standard low-level runtime; used by Kubernetes |
| **CRI-O** | Kubernetes-focused runtime; lightweight |
| **Podman** | Daemonless, rootless Docker alternative; Red Hat |
| **Docker Desktop** | Development tool on Mac/Windows |

---

## 5. Category 4 — Container Orchestration

### Purpose

Manage large numbers of containers across multiple machines — scheduling, scaling, healing, networking, and storage.

### Kubernetes (K8s) — The Dominant Standard

```
THE KUBERNETES ARCHITECTURE:

┌──────────────────── CLUSTER ────────────────────────────┐
│                                                          │
│  ┌────────── CONTROL PLANE ──────────┐                  │
│  │  API Server    ←── clients        │                  │
│  │  etcd          ←── cluster state  │                  │
│  │  Scheduler     ←── place pods     │                  │
│  │  Controller    ←── desired state  │                  │
│  └────────────────────────────────────┘                 │
│                                                          │
│  ┌────── WORKER NODE 1 ──────┐  ┌── WORKER NODE 2 ──┐  │
│  │  kubelet (node agent)     │  │  kubelet           │  │
│  │  kube-proxy (networking)  │  │  kube-proxy        │  │
│  │  container runtime        │  │  container runtime │  │
│  │  ┌────────────────────┐   │  │  ┌──────────────┐  │  │
│  │  │  Pod [Container A] │   │  │  │Pod [Cont. B] │  │  │
│  │  │  Pod [Container A] │   │  │  │Pod [Cont. B] │  │  │
│  │  └────────────────────┘   │  │  └──────────────┘  │  │
│  └───────────────────────────┘  └────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

**Core Kubernetes objects:**

| Object | Purpose | Analogy |
|--------|---------|---------|
| Pod | Smallest deployable unit, wraps 1+ containers | A single process |
| Deployment | Manages ReplicaSets; rolling updates | Process manager |
| Service | Stable network endpoint for pods | Load balancer |
| Ingress | HTTP routing from outside cluster into services | Nginx reverse proxy |
| ConfigMap | Non-secret configuration data | Config file |
| Secret | Sensitive data (passwords, tokens) | Encrypted config file |
| PersistentVolume | Storage that survives pod restarts | Hard drive |
| Namespace | Virtual cluster isolation | Team workspace |
| HorizontalPodAutoscaler | Auto-scale pods based on metrics | Auto-scaling group |
| DaemonSet | Run one pod per node (e.g., log collector) | System-wide agent |
| StatefulSet | Pods with stable identity (e.g., databases) | Stateful VM |

**Managed Kubernetes by cloud provider:**

| Provider | Service |
|----------|---------|
| AWS | EKS (Elastic Kubernetes Service) |
| Google Cloud | GKE (Google Kubernetes Engine) — most mature |
| Azure | AKS (Azure Kubernetes Service) |
| DigitalOcean | DOKS |
| VMware | Tanzu |

**Other container orchestration options:**

| Tool | Description |
|------|-------------|
| **Docker Swarm** | Simple, built into Docker, declining in adoption |
| **Nomad** (HashiCorp) | Multi-workload orchestrator (not just containers) |
| **Amazon ECS** | AWS-native container service (no Kubernetes needed) |
| **Amazon ECS Fargate** | Serverless containers — no node management |

---

## 6. Category 5 — Infrastructure as Code (IaC)

### Purpose

Define, provision, and manage infrastructure using code instead of manual console clicks — enabling version control, repeatability, and automation.

### Terraform — The Industry Standard IaC Tool

```hcl
# main.tf — Provision an AWS EC2 instance + S3 bucket

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.region
}

variable "region" {
  description = "AWS region"
  default     = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t3.micro"
  
  tags = {
    Name        = "web-server"
    Environment = "production"
    ManagedBy   = "terraform"
  }
}

resource "aws_s3_bucket" "assets" {
  bucket = "myapp-assets-production"
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  versioning_configuration {
    status = "Enabled"
  }
}

output "web_ip" {
  value = aws_instance.web.public_ip
}
```

**Terraform workflow:**

```bash
terraform init      # Download providers and modules
terraform fmt       # Format code
terraform validate  # Check syntax and configuration validity
terraform plan      # Preview what will change (ALWAYS do this)
terraform apply     # Apply the changes (--auto-approve in CI)
terraform destroy   # Tear down all managed resources
terraform state     # Manage state file
```

**Terraform key concepts:**

| Concept | Description |
|---------|-------------|
| Provider | Plugin for a cloud/service API (AWS, GCP, Azure, GitHub, Datadog) |
| Resource | A piece of infrastructure to manage (EC2, S3 bucket, RDS) |
| Data source | Read existing infrastructure (find an AMI, get a secret) |
| Variable | Input values to make configs reusable |
| Output | Export values for use in other modules |
| Module | Reusable group of resources (e.g., "VPC module") |
| State file | Terraform's record of managed infrastructure (store remotely in S3) |
| Workspace | Multiple environments from one config (dev/staging/prod) |

**IaC tools comparison:**

| Tool | Approach | Language | Best for |
|------|---------|---------|---------|
| **Terraform** | Declarative | HCL | Multi-cloud, most popular |
| **Pulumi** | Declarative | Python, TypeScript, Go, Java | Developers who prefer real languages |
| **AWS CloudFormation** | Declarative | YAML/JSON | AWS-only, native |
| **AWS CDK** | Imperative | TypeScript, Python | Developers wanting real code for AWS |
| **Azure Bicep** | Declarative | Bicep | Azure-native |
| **Google Deployment Manager** | Declarative | YAML | GCP-native |
| **OpenTofu** | Declarative | HCL | Open-source Terraform fork |

---

## 7. Category 6 — Configuration Management

### Purpose

Automate the configuration of servers and software after they're provisioned — ensure consistency across all machines.

**Note:** In cloud-native/Kubernetes environments, Config Management is largely replaced by IaC + containers. But it remains critical for VM-based workloads and bare-metal servers.

### Ansible — The Most Popular Configuration Management Tool

```yaml
# playbook.yml — Configure a web server
---
- name: Configure web servers
  hosts: webservers
  become: yes    # Run as sudo
  vars:
    nginx_port: 80
    app_version: "1.2.3"
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Copy Nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx
    
    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Deploy application
      unarchive:
        src: "https://myregistry.io/myapp-{{ app_version }}.tar.gz"
        dest: /opt/myapp
        remote_src: yes
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Ansible core concepts:**

| Concept | Description |
|---------|-------------|
| Inventory | List of servers to manage (static file or dynamic from cloud) |
| Playbook | YAML file with tasks to run on hosts |
| Task | A single automation step (install package, copy file, run command) |
| Module | Ansible's built-in functions (apt, yum, copy, template, service...) |
| Role | Reusable bundle of tasks, variables, and templates |
| Handler | Task that runs only when notified by another task |
| Vault | Encrypted secrets within Ansible (ansible-vault) |
| Galaxy | Ansible's marketplace for pre-built roles |

**Ansible architecture:**

```
Control Node (your machine / CI runner)
  Ansible installed here
  Reads inventory file
  Connects via SSH
         ↓
  Managed Nodes (your servers)
  Only need Python + SSH
  No agent required!
```

**Ansible vs other config management tools:**

| Tool | Language | Agent | Learning curve | Best for |
|------|---------|-------|---------------|---------|
| **Ansible** | YAML (Jinja2 templates) | Agentless (SSH) | Low | Linux server configuration |
| **Chef** | Ruby (DSL) | Agent required | High | Large enterprises, complex workflows |
| **Puppet** | Puppet DSL | Agent required | High | Enterprise compliance environments |
| **Salt (SaltStack)** | YAML/Python | Agent (or agentless) | Medium | Large-scale, performance-sensitive |

---

## 8. Category 7 — Cloud Platforms

### The Big Three

Every serious DevOps engineer needs to understand the major cloud providers:

**AWS (Amazon Web Services) — Market leader (~32% share)**

| Service Category | Key Services |
|----------------|-------------|
| Compute | EC2, Lambda, ECS, EKS, Fargate |
| Storage | S3, EBS, EFS, Glacier |
| Database | RDS, DynamoDB, Aurora, ElastiCache, Redshift |
| Networking | VPC, Route53, CloudFront, ALB/NLB, Direct Connect |
| Security | IAM, KMS, WAF, GuardDuty, Security Hub |
| Developer Tools | CodePipeline, CodeBuild, CodeDeploy, CodeArtifact |
| Monitoring | CloudWatch, X-Ray |
| Containers | ECR, ECS, EKS |
| IaC | CloudFormation, CDK |

**GCP (Google Cloud Platform) — Third (~11% share)**

| Service Category | Key Services |
|----------------|-------------|
| Compute | Compute Engine, Cloud Run, GKE, Cloud Functions |
| Storage | Cloud Storage, Persistent Disk, Filestore |
| Database | Cloud SQL, Spanner, Bigtable, Firestore, BigQuery |
| Networking | VPC, Cloud DNS, Cloud CDN, Cloud Load Balancing |
| Security | IAM, Cloud KMS, Security Command Center |
| Containers | Artifact Registry, GKE (best managed K8s) |
| Monitoring | Cloud Monitoring, Cloud Trace, Cloud Logging |

**Azure (Microsoft Azure) — Second (~23% share)**

| Service Category | Key Services |
|----------------|-------------|
| Compute | Virtual Machines, App Service, AKS, Functions, Container Instances |
| Storage | Blob Storage, Azure Disk, Azure Files |
| Database | Azure SQL, Cosmos DB, Azure Database for PostgreSQL |
| Networking | Virtual Network, Azure DNS, Application Gateway, Front Door |
| Security | Azure AD (Entra ID), Key Vault, Defender for Cloud |
| Developer Tools | Azure DevOps, ACR, Azure Pipelines |
| Monitoring | Azure Monitor, Application Insights, Log Analytics |

**Cloud computing models:**

```
On-Premises:   You own and manage everything
IaaS:          Cloud provides hardware (VMs, networking, storage)
               You manage OS, runtime, app
               Examples: EC2, GCE, Azure VMs

PaaS:          Cloud provides platform (OS, runtime, middleware)
               You manage just your app and data
               Examples: Heroku, App Engine, Azure App Service, AWS Elastic Beanstalk

CaaS:          Cloud provides container platform
               You manage containers and application
               Examples: ECS/EKS, GKE, AKS

FaaS/Serverless: Cloud provides execution environment
               You write only functions — no infrastructure
               Examples: Lambda, Cloud Functions, Azure Functions

SaaS:          Cloud provides the full application
               You use it
               Examples: Gmail, Salesforce, GitHub
```

---

## 9. Category 8 — Monitoring & Observability

### Purpose

Understand the health, performance, and behavior of systems in production.

### The Prometheus + Grafana Stack (Most Popular Open-Source Stack)

**Prometheus — Metrics Collection and Storage**

```yaml
# prometheus.yml — Prometheus configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true

  - job_name: 'myapp'
    static_configs:
      - targets: ['myapp:8080']
    metrics_path: /metrics
```

**PromQL (Prometheus Query Language) essentials:**

```
# Error rate (%)
100 * (
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  / sum(rate(http_requests_total[5m]))
)

# 99th percentile latency
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# Requests per second
sum(rate(http_requests_total[2m]))

# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage per pod
container_memory_working_set_bytes / 1024 / 1024
```

**Grafana — Visualization**

```
Data Sources: Prometheus, Loki, Tempo, Elasticsearch, CloudWatch, Datadog, etc.
Dashboards: JSON-as-code stored in Git
Alerting: Route alerts to PagerDuty, Slack, OpsGenie
```

**ELK Stack (Logging):**

```
Application → Filebeat/Fluentd → Logstash (parse/transform) → Elasticsearch (store/index) → Kibana (visualize)
```

**Loki + Grafana (modern logging — lightweight):**

```
Application → Promtail/Fluentd → Loki (label-indexed storage) → Grafana (visualize)
```

**Distributed Tracing:**

```
Application (instrumented with OpenTelemetry)
         ↓
  Tracing Backend (Jaeger / Tempo / AWS X-Ray)
         ↓
  Visualization (Grafana / Jaeger UI)
```

**The Observability Stack by category:**

| Category | Open-Source | Commercial |
|----------|------------|-----------|
| Metrics | Prometheus, OpenMetrics | Datadog, New Relic |
| Logs | Elasticsearch, Loki | Splunk, Datadog, Papertrail |
| Traces | Jaeger, Tempo, Zipkin | Datadog APM, Dynatrace, AWS X-Ray |
| Alerting | Prometheus Alertmanager, Grafana | PagerDuty, OpsGenie, VictorOps |
| Dashboards | Grafana | Datadog, Kibana, New Relic |
| All-in-one APM | N/A | Datadog, Dynatrace, New Relic |

---

## 10. Category 9 — Security & Compliance (DevSecOps)

### Purpose

Shift security left — integrate security checks at every stage of the pipeline rather than a separate late-stage gate.

### Security in the Pipeline

```
Code → SAST (Static analysis) → Dependency scan → Docker build → Image scan → 
Deploy to staging → DAST (Dynamic analysis) → Deploy to production → 
Runtime security → Compliance audit
```

**Security scanning tools by type:**

| Type | What it scans | Tools |
|------|-------------|-------|
| SAST (Static) | Source code for vulnerabilities | SonarQube, Semgrep, CodeQL (GitHub), Checkmarx |
| SCA (Dependencies) | Third-party library CVEs | Dependabot, Snyk, OWASP Dependency-Check |
| Secrets scanning | API keys, passwords in code | GitLeaks, GitHub Secret Scanning, TruffleHog |
| Container image scanning | CVEs in Docker image layers | Trivy, Snyk, Clair, Amazon ECR scanning |
| IaC scanning | Misconfigurations in Terraform/CloudFormation | Checkov, tfsec, Terrascan |
| DAST (Dynamic) | Running app for vulnerabilities | OWASP ZAP, Burp Suite |
| Runtime security | Container behavior in production | Falco, Sysdig, Aqua |

**Trivy Example (most popular container scanner):**

```bash
# Scan a Docker image
trivy image myapp:1.2.3

# Scan with severity threshold (exit 1 if HIGH or CRITICAL found)
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:1.2.3

# Scan infrastructure as code
trivy config ./terraform/

# Scan filesystem
trivy fs .

# Scan a Git repository
trivy repo https://github.com/myorg/myapp
```

**OWASP Top 10 and DevOps responsibilities:**

| OWASP Risk | DevOps Tool/Practice |
|-----------|---------------------|
| Broken Access Control | IaC with least-privilege IAM; RBAC in K8s |
| Cryptographic Failures | Secrets in Vault (not env vars); TLS everywhere |
| Injection | SAST scanning; parameterized queries |
| Insecure Design | Threat modeling in planning; security design review |
| Security Misconfiguration | IaC scanning with Checkov; CIS benchmarks |
| Vulnerable Components | Dependabot; Snyk; automated dependency updates |
| Auth Failures | SSO/OIDC; MFA; short-lived credentials (STS) |
| Software Integrity Failures | Signed commits; signed container images; SLSA |
| Logging Failures | Structured logging; centralized logging; audit trails |
| SSRF | Network policies; egress filtering in K8s |

---

## 11. Category 10 — Artifact & Registry Management

### Purpose

Store, version, and distribute build artifacts (Docker images, packages, binaries) securely.

| Type | Tools |
|------|-------|
| **Docker image registry** | Docker Hub, AWS ECR, GCP Artifact Registry, Azure ACR, Harbor, GitHub Container Registry |
| **Universal artifact manager** | JFrog Artifactory, Sonatype Nexus |
| **Language-specific** | npm Registry (JS), PyPI (Python), Maven Central (Java), NuGet (.NET) |
| **Helm charts** | Helm chart repositories: ArtifactHub, GitHub Releases, Harbor |

**Harbor** — Self-hosted enterprise registry with:
- Role-based access control
- Vulnerability scanning (Trivy built-in)
- Image signing (Cosign/Notary)
- Garbage collection
- Replication across regions

---

## 12. Category 11 — GitOps

### Purpose

Use Git as the single source of truth for both application code and infrastructure/deployment configuration. Changes to production are made only through Git.

### GitOps Principles

1. **Declarative:** Everything described in Git as desired state (not scripts)
2. **Versioned:** Git history = complete audit trail of every change
3. **Automated:** A controller continuously reconciles the live state to match Git
4. **Observable:** Any drift from Git is detected and can be auto-corrected

### ArgoCD — Most Popular GitOps Tool for Kubernetes

```
Developer merges PR with new K8s manifest
         ↓
ArgoCD watches the Git repo
         ↓
ArgoCD detects the manifest changed
         ↓
ArgoCD applies the change to the K8s cluster
         ↓
ArgoCD marks the app as "Synced"
         ↓
          (No kubectl apply by humans ever)
```

**ArgoCD Application manifest:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-config
    targetRevision: main
    path: kubernetes/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true      # Delete resources removed from Git
      selfHeal: true   # Re-apply if someone manually changed the cluster
```

**Flux** is the other major GitOps operator (originally from Weaveworks, now CNCF graduated).

| Feature | ArgoCD | Flux |
|---------|--------|------|
| UI | Rich web UI | CLI-focused (UI available) |
| Multi-cluster | Yes | Yes |
| Helm support | Yes | Yes (better) |
| Multi-tenancy | Yes | Yes (better) |
| Push vs Pull | Pull (GitOps) | Pull (GitOps) |

---

## 13. Category 12 — Service Mesh

### Purpose

Manage service-to-service communication in a microservices architecture with features like traffic routing, mutual TLS, observability, and circuit breaking — without changing application code.

### How a Service Mesh Works

```
WITHOUT SERVICE MESH:
  Service A ──→ Service B  (no visibility, no control)

WITH SERVICE MESH (Sidecar pattern):
  Service A + [Proxy Sidecar] ──→ [Proxy Sidecar] + Service B
               ↑                          ↑
          (Envoy proxy)             (Envoy proxy)
          Reports to control plane, enforces policies
```

**What service meshes provide:**

| Capability | Description |
|-----------|-------------|
| mTLS | Every service-to-service call encrypted and authenticated |
| Traffic control | Canary deployments, A/B testing, retries, timeouts |
| Observability | Automatic golden signal metrics for every service |
| Circuit breaking | Stop sending requests to unhealthy services |
| Rate limiting | Control request rates between services |

**Service mesh tools:**

| Tool | Description |
|------|-------------|
| **Istio** | Most feature-rich; uses Envoy proxy; complex to set up |
| **Linkerd** | Lightweight, simpler than Istio, CNCF graduated |
| **Consul Connect** | HashiCorp's service mesh; works with VMs too |
| **AWS App Mesh** | AWS-native Istio-compatible service mesh |
| **Cilium** | eBPF-based, at the kernel level — no sidecar needed |

---

## 14. Category 13 — Secrets Management

### Purpose

Securely store, distribute, rotate, and audit access to sensitive values (passwords, API keys, certificates, database credentials).

### The Problem with Environment Variables for Secrets

```bash
# INSECURE — Don't do this in production
export DATABASE_PASSWORD=supersecret123
docker run -e DATABASE_PASSWORD=supersecret123 myapp

# Problems:
# - Secrets visible in process list: ps aux
# - Secrets in CI/CD logs if pipeline prints env vars
# - No rotation — change requires redeploy
# - No audit trail
# - No access control per secret
```

### HashiCorp Vault — The Gold Standard

```
Application → Vault Agent (sidecar) → Vault Server
                    ↓
           Dynamic secrets generated
           on-demand → database
           credentials expire after 1hr
           → auto-rotated
```

```bash
# Vault CLI examples
vault secrets enable database          # Enable DB secrets engine
vault write database/config/mydb \     # Configure DB connection
  plugin_name=postgresql-database-plugin \
  allowed_roles="myapp-role" \
  connection_url="postgresql://{{username}}:{{password}}@db:5432/myapp"

vault write database/roles/myapp-role \  # Define role with short TTL
  db_name=mydb \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';" \
  default_ttl="1h" \
  max_ttl="24h"

# Application requests credentials
vault read database/creds/myapp-role
# Returns: username=v_myapp_AbCd1234, password=Xyz...  (expires in 1hr)
```

**Secrets management tools:**

| Tool | Description | Best for |
|------|-------------|---------|
| **HashiCorp Vault** | Self-hosted, full-featured, dynamic secrets | Multi-cloud, on-prem, complex requirements |
| **AWS Secrets Manager** | AWS-native, auto-rotation, pay-per-secret | AWS environments |
| **GCP Secret Manager** | GCP-native, simple API | GCP environments |
| **Azure Key Vault** | Azure-native, certificates + keys + secrets | Azure environments |
| **Doppler** | Developer-friendly SaaS, multi-environment | Startups, simple workflows |
| **External Secrets Operator** | K8s operator to sync from Vault/AWS SM/etc | Kubernetes with external secret stores |

**Kubernetes secrets best practices:**

```yaml
# Never store secrets as plain base64 (that's not encryption!)

# Better: Use External Secrets Operator to pull from Vault
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: myapp-db-secret
  data:
  - secretKey: password
    remoteRef:
      key: secret/myapp/database
      property: password
```

---

## 15. Category 14 — Collaboration & Planning

| Category | Tools |
|---------|-------|
| **Project Management** | Jira, Linear, GitHub Issues, Asana, Monday.com |
| **Documentation** | Confluence, Notion, GitBook, Backstage (developer portal) |
| **Diagramming** | draw.io, Miro, Lucidchart, Excalidraw |
| **Communication** | Slack, Microsoft Teams, Discord |
| **Incident Communication** | Statuspage, Atlassian Statuspage |
| **On-call** | PagerDuty, OpsGenie, VictorOps, xMatters |
| **Developer Portal** | Backstage (Spotify open-source), Port, Cortex |

**Backstage (Internal Developer Portal):**

```
Backstage gives every developer a single place to:
  - Discover services owned by other teams
  - Create new services from pre-approved templates (scaffolding)
  - See documentation for any service
  - Manage CI/CD pipelines
  - See infrastructure and Kubernetes status
  - Find runbooks and ADRs
  → Reduces cognitive load, enables self-service
```

---

## 16. Category 15 — Testing Tools

| Category | Tools |
|---------|-------|
| **Unit testing — JavaScript** | Jest, Vitest, Mocha + Chai |
| **Unit testing — Python** | pytest, unittest |
| **Unit testing — Java** | JUnit 5, TestNG, Mockito |
| **Unit testing — Go** | go test (native), Testify |
| **E2E browser testing** | Playwright, Cypress, Selenium |
| **API testing** | Postman, Newman, REST-Assured, Pact (contracts) |
| **Load/performance testing** | k6, Gatling, JMeter, Locust, Artillery |
| **Infrastructure testing** | Terratest, Kitchen-Terraform |
| **K8s testing** | Kyverno (policies), Conftest (OPA) |
| **Mutation testing** | Stryker (JS), PiTest (Java), mutmut (Python) |

---

## 17. Complete Toolchain at a Glance

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPLETE DEVOPS TOOLCHAIN                       │
├──────────────────────┬──────────────────────────────────────────────┤
│  CATEGORY            │  TOOLS                                        │
├──────────────────────┼──────────────────────────────────────────────┤
│  Version Control     │  Git, GitHub, GitLab, Bitbucket               │
│  Project Mgmt        │  Jira, Linear, GitHub Issues                  │
│  IDE                 │  VS Code, IntelliJ IDEA, Neovim               │
│  CI/CD               │  GitHub Actions, GitLab CI, Jenkins, CircleCI │
│  Containerization    │  Docker, Podman, containerd                   │
│  Orchestration       │  Kubernetes (EKS, GKE, AKS), Nomad            │
│  GitOps              │  ArgoCD, Flux                                 │
│  IaC                 │  Terraform, Pulumi, CloudFormation, CDK       │
│  Config Mgmt         │  Ansible, Chef, Puppet                        │
│  Cloud               │  AWS, GCP, Azure                              │
│  Artifact Registry   │  ECR, Harbor, Artifactory, Nexus              │
│  Metrics             │  Prometheus, Datadog, CloudWatch              │
│  Logging             │  ELK Stack, Loki + Grafana, Splunk            │
│  Tracing             │  Jaeger, Tempo, AWS X-Ray                     │
│  Alerting            │  Alertmanager, PagerDuty, OpsGenie            │
│  Dashboards          │  Grafana, Kibana, Datadog                     │
│  Secrets             │  HashiCorp Vault, AWS SM, GCP SM              │
│  Security (SAST)     │  SonarQube, Semgrep, CodeQL                   │
│  Security (Deps)     │  Dependabot, Snyk                             │
│  Security (Images)   │  Trivy, Clair                                 │
│  Security (IaC)      │  Checkov, tfsec                               │
│  Service Mesh        │  Istio, Linkerd, Cilium                       │
│  Service Mesh (ingress) │ Nginx Ingress, Traefik, Envoy, Kong       │
│  Developer Portal    │  Backstage, Port                              │
│  Performance Testing │  k6, Gatling, Locust, JMeter                  │
│  Communication       │  Slack, Teams                                 │
│  Incident Mgmt       │  PagerDuty, OpsGenie                         │
└──────────────────────┴──────────────────────────────────────────────┘
```

---

## 18. Choosing the Right Tools

**Principles for tool selection:**

1. **Choose boring technology** — Use well-established tools with large communities over cutting-edge with small support bases (unless the new tool solves a specific pain the boring tool can't)

2. **Avoid NIH (Not Invented Here)** — Don't build tools that already exist well in open source

3. **Consider operational burden** — Self-hosted tools require maintenance. Managed services cost money but reduce toil.

4. **Align with team skills** — A great tool nobody on the team knows is worse than a good tool everyone does

5. **Interoperability** — Tools that work well with each other are worth more than individually superior tools that don't integrate

6. **CNCF Graduated Projects** — The Cloud Native Computing Foundation has a maturity model; Graduated = production-ready

**CNCF Graduated Projects (the most battle-tested):**

```
Kubernetes, Prometheus, Envoy, CoreDNS, containerd, 
Fluentd, Jaeger, Vitess, Argo, Flux, Linkerd, Helm,
Harbor, CRI-O, etcd, Rook, TUF, Falco, Open Policy Agent
```

---

## 19. The Minimum Viable DevOps Stack

If you're starting from scratch and need to pick the most important tools:

```
TIER 1 — Non-negotiable:
  ✓ Git + GitHub/GitLab   (version control)
  ✓ GitHub Actions or GitLab CI  (CI/CD)
  ✓ Docker               (containerization)
  ✓ Terraform            (infrastructure as code)

TIER 2 — Add as you grow:
  ✓ Kubernetes (EKS/GKE/AKS)  (orchestration)
  ✓ Prometheus + Grafana       (monitoring)
  ✓ ArgoCD                     (GitOps)
  ✓ HashiCorp Vault or cloud SM (secrets)
  ✓ Trivy + Dependabot          (security scanning)

TIER 3 — Scale and mature:
  ✓ Service mesh (Istio/Linkerd)
  ✓ Distributed tracing (Jaeger/Tempo)
  ✓ Internal developer platform (Backstage)
  ✓ Chaos engineering (LitmusChaos)
  ✓ Advanced security (Falco, OPA/Gatekeeper)
```

**What to learn first as a DevOps beginner:**

```
1. Git (essential for everything else)
2. Linux + Bash (essential for everything else)
3. Docker (containerization is everywhere)
4. CI/CD — GitHub Actions (practical automation)
5. Cloud — AWS fundamentals (EC2, S3, IAM, VPC)
6. Terraform (IaC — provision the cloud)
7. Kubernetes (the most in-demand skill)
8. Monitoring (Prometheus + Grafana)
9. Ansible (configuration management)
10. Security (DevSecOps scanning)
```

---

## 20. Summary

| Category | Leader Tool | Runner-up | Key Concept |
|---------|------------|----------|-------------|
| Version Control | Git + GitHub | GitLab | Everything in Git |
| CI/CD | GitHub Actions | GitLab CI, Jenkins | Automate from commit to deploy |
| Containers | Docker | Podman | Immutable, portable artifacts |
| Orchestration | Kubernetes | Nomad | Self-healing, scaling, zero-downtime |
| IaC | Terraform | Pulumi, CDK | Infrastructure is code |
| Config Management | Ansible | Chef, Puppet | Consistent, reproducible servers |
| Cloud | AWS | GCP, Azure | Run on managed infrastructure |
| Monitoring | Prometheus + Grafana | Datadog | Metrics, logs, traces |
| GitOps | ArgoCD | Flux | Git = single source of truth |
| Secrets | HashiCorp Vault | AWS Secrets Manager | Never hardcode credentials |
| Security | Trivy + SonarQube | Snyk, Checkov | Shift left |

---

**Previous:** [04 — Culture & Mindset](04-culture-and-mindset.md)
**Next:** [Interview Questions](interview-questions.md)
