# DevOps Zero to Hero — Complete Repo

> A hands-on, end-to-end DevOps learning repository covering everything from Linux fundamentals to production-grade deployments. Built to accompany the **DevOps Zero to Hero** playlist.

---

## Table of Contents

- [About](#about)
- [Who Is This For?](#who-is-this-for)
- [Curriculum Overview](#curriculum-overview)
- [Tech Stack & Tools](#tech-stack--tools)
- [Getting Started](#getting-started)
- [Repo Structure](#repo-structure)
- [Contributing](#contributing)
- [License](#license)

---

## About

This repository is the official companion to the **DevOps Zero to Hero** video playlist. Each folder maps to a module in the series and contains:

- Step-by-step notes and cheatsheets
- Hands-on project files (Dockerfiles, Kubernetes manifests, Terraform configs, CI/CD pipelines, etc.)
- Real-world examples you can deploy immediately
- Interview prep questions per topic

Whether you are a complete beginner or a developer looking to break into DevOps, this repo gives you a structured, practical path.

---

## Who Is This For?

| Role | How to Use |
|---|---|
| Absolute Beginner | Follow modules 1–3 before touching any tooling |
| Developer moving into DevOps | Start from module 3 (Containers) and go forward |
| DevOps Engineer (upskilling) | Jump to any module; use as a reference/cheatsheet |
| Interview Prep | Each module folder has an `interview-questions.md` |

---

## Curriculum Overview

| # | Module | Key Topics |
|---|--------|------------|
| 01 | **DevOps Fundamentals** | What is DevOps, SDLC, DevOps lifecycle, culture & mindset, toolchain overview |
| 02 | **Linux & Shell Scripting** | Filesystem, permissions, bash scripting, cron, systemd |
| 03 | **Networking Fundamentals** | OSI model, DNS, TCP/IP, SSH, firewalls, load balancers |
| 04 | **Git & Version Control** | Branching strategies, rebasing, pull requests, GitOps |
| 05 | **Containers — Docker** | Images, containers, volumes, networks, Compose, best practices |
| 06 | **CI/CD Pipelines** | GitHub Actions, Jenkins, GitLab CI, ArgoCD |
| 07 | **Cloud Platforms (AWS / GCP / Azure)** | Core services, IAM, networking, compute, storage |
| 08 | **Infrastructure as Code — Terraform** | Providers, state, modules, workspaces, remote backend |
| 09 | **Configuration Management — Ansible** | Playbooks, roles, inventory, Ansible Vault |
| 10 | **Container Orchestration — Kubernetes** | Pods, Deployments, Services, Ingress, Helm, RBAC |
| 11 | **Python for DevOps** *(learn alongside)* | Python basics, scripting, file I/O, APIs, boto3/cloud SDKs, automation scripts |
| 12 | **Monitoring & Observability** | Prometheus, Grafana, Loki, alerting, dashboards |
| 13 | **Security & DevSecOps** | Secrets management, SAST/DAST, image scanning, Vault |
| 14 | **Site Reliability Engineering (SRE)** | SLIs/SLOs/SLAs, incident management, chaos engineering |

---

## Tech Stack & Tools

**Containers & Orchestration**
`Docker` · `Kubernetes` · `Helm` · `containerd`

**CI/CD**
`GitHub Actions` · `Jenkins` · `GitLab CI` · `ArgoCD` · `Tekton`

**Infrastructure as Code**
`Terraform` · `Ansible` · `Pulumi`

**Cloud**
`AWS` · `GCP` · `Azure`

**Monitoring & Logging**
`Prometheus` · `Grafana` · `Loki` · `OpenTelemetry` · `ELK Stack`

**Security**
`HashiCorp Vault` · `Trivy` · `SonarQube` · `OPA / Gatekeeper`

**Other**
`Linux` · `Bash` · `Python` · `Git` · `Nginx` · `Makefile`

---

## Getting Started

### Prerequisites

- Git installed
- Docker Desktop (or Docker Engine on Linux)
- A terminal (bash / zsh)

### Clone the repo

```bash
git clone https://github.com/ajazbeig/DevOps-Complete-Repo.git
cd DevOps-Complete-Repo
```

Navigate into any module folder and follow the `README.md` inside it.

---

## Repo Structure

```
DevOps-Complete-Repo/
├── 01-devops-fundamentals/
├── 02-linux-shell-scripting/
├── 03-networking/
├── 04-git/
├── 05-docker/
├── 06-cicd/
├── 07-cloud/
├── 08-terraform/
├── 09-ansible/
├── 10-kubernetes/
├── 11-python-for-devops/
├── 12-monitoring/
├── 13-devsecops/
├── 14-sre/
└── README.md
```

Each module folder follows this layout:

```
<module>/
├── README.md          # Notes, concepts, commands
├── interview-questions.md
├── projects/          # Hands-on project files
└── cheatsheet.md
```

---

## Contributing

Contributions are welcome! Please open an issue first to discuss changes.

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-topic`
3. Commit your changes: `git commit -m "feat: add <topic>"`
4. Push and open a Pull Request

---

## License

This repository is licensed under the [MIT License](LICENSE).

---

> "The best way to learn DevOps is by doing DevOps." — Start with module 01 and build your way up. Good luck! 🚀

