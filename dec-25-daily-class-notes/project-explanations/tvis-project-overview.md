# Project Overview

## Domain & Product

**TruValidate Integrated Solutions (TVIS)** is a **fraud prevention / identity verification** product in the **FinTech / Security** domain.

The product focuses on **Device Risk** — assessing the risk level of devices such as phones, laptops, and browsers interacting with client platforms. It uses data science models to fingerprint devices and flag suspicious activity like fraud, account takeover, or bot attacks.

## Team Structure

Your organization follows a **shared resource model**, meaning the DevOps/Cloud team is not dedicated to a single product. Instead, it supports **multiple application teams** horizontally.

| Team                       | Function                                        | Strength                                                                        |
| -------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| **Cloud/DevOps Team**      | Infrastructure, CI/CD, monitoring for all teams | 8 members (1 Manager, 2 Seniors, 4 Engineers, 1 Architect)                      |
| **App Team (Device Risk)** | Application development, QA, data science       | 12 members (1 Manager, 2 Seniors, 4 Devs, 2 QA, 1 Architect, 2 Data Scientists) |

This is a common enterprise pattern. The Cloud team acts as an **internal platform engineering team**, providing infrastructure as a service to product teams.

## Tech Stack Breakdown

| Tool                                 | Purpose                                                               |
| ------------------------------------ | --------------------------------------------------------------------- |
| **AWS**                              | Cloud provider for compute, networking, storage, and managed services |
| **EKS (Elastic Kubernetes Service)** | Managed Kubernetes for container orchestration                        |
| **Terraform**                        | Infrastructure as Code for provisioning AWS resources declaratively   |
| **Jenkins**                          | CI/CD automation for build, test, and deploy pipelines                |
| **Docker**                           | Containerization of microservices                                     |
| **Kubernetes**                       | Orchestrating containers at scale through EKS                         |
| **Git / Bitbucket**                  | Version control and source code management                            |
| **Prometheus**                       | Metrics collection and time-series monitoring                         |
| **Grafana**                          | Visualization dashboards for Prometheus metrics                       |
| **Scripting (Bash/Python)**          | Automation of operational tasks                                       |

## Day-to-Day Responsibilities & Agile Workflow

You follow **Agile/Scrum methodology** with:

* **Daily Standups (15 min)** — Quick sync on progress, blockers, and plans
* **Sprint Planning** — Scrum Master and Product Owner prioritize and assign tickets from the backlog
* **Sprint Review** — Demo completed work to stakeholders
* **Sprint Retrospective** — Reflect on and improve processes

All work is tracked in **Jira**. Tickets come from multiple teams and fall into three categories:

1. **Infrastructure Support** — Server provisioning, access management (IAM), environment refreshes such as resetting QA/Dev data
2. **Automation & CI/CD** — Building and improving Jenkins pipelines, integrating new tools, creating Bitbucket repos for new services
3. **Monitoring & Alerting** — Setting up Prometheus alerts and tuning thresholds to reduce alert fatigue

You prioritize work based on **ticket severity** — a production-down incident takes precedence over a new pipeline request.

## Environment Architecture

You maintain **4 isolated environments**, each on its **own dedicated EKS cluster**:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   DEV    │    │    QA    │    │   PROD   │    │    DR    │
│ 2 Nodes  │    │ 4 Nodes  │    │ 10 Nodes │    │ 4 Nodes  │
│          │    │          │    │          │    │          │
│ EKS #1   │    │ EKS #2   │    │ EKS #3   │    │ EKS #4   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### Why separate EKS clusters per environment?

* **Security isolation** — A compromised Dev cluster cannot reach Prod data
* **Blast radius containment** — Failures in one environment do not cascade
* **Independent scaling** — Prod needs 10 nodes; Dev only needs 2
* **Compliance** — Many regulated industries, such as FinTech, require strict environment separation

**DR (Disaster Recovery)** mirrors QA in size (4 nodes) and is a **standby replica of Prod**, ready to take over if the primary Prod region fails.

## Production Instance Types & Why They Differ

| Instance        | Family                                     | Why Used                                                               |
| --------------- | ------------------------------------------ | ---------------------------------------------------------------------- |
| **m4.4xlarge**  | General purpose (older generation)         | Balanced CPU/memory for legacy services                                |
| **c7g.2xlarge** | Compute-optimized (Graviton/ARM)           | CPU-intensive workloads like risk scoring engines; also cost-efficient |
| **m6i.2xlarge** | General purpose (current generation Intel) | Modern balanced workloads, API servers, data processing                |

Using **mixed instance types** in Production is a best practice because:

* Different microservices have **different resource profiles** such as CPU-heavy or memory-heavy
* **Cost optimization** — Graviton instances (c7g) are about 20% cheaper than Intel equivalents
* **Availability** — Diversifying instance families reduces the risk of capacity shortages in a single family

## Release Cycle

**Release frequency: Every 30 days (Sprint-based)**

This means:

* Each sprint, typically 2 weeks of development work, feeds into a monthly release train
* Code flows through **Dev → QA → Prod** with gated approvals
* Jenkins pipelines automate the build → test → deploy flow at each stage
* Rollbacks are possible via Kubernetes deployment strategies such as rolling updates and blue-green deployments

## Terraform Structure

The file mentions Terraform as the next topic. Theoretically, a typical Terraform setup for this project would look like this:

```
terraform/
├── modules/           # Reusable modules (EKS, VPC, IAM, S3, etc.)
│   ├── eks/
│   ├── vpc/
│   └── rds/
├── environments/      # Per-environment configurations
│   ├── dev/
│   ├── qa/
│   ├── prod/
│   └── dr/
├── backend.tf         # Remote state (S3 bucket + DynamoDB lock table)
└── variables.tf       # Shared variables
```

* **Remote state** is stored in **S3** with **DynamoDB locking** to prevent concurrent modifications
* **Workspaces** or separate directories per environment can be used for isolation

## Summary

Your project is a **real-world enterprise DevOps setup** supporting a fraud-detection product. You operate as a shared platform team, managing infrastructure across environments using modern IaC and CI/CD practices, with a strong emphasis on **security isolation, scalability, and observability**.

This is a solid, production-grade architecture commonly found in FinTech companies.
