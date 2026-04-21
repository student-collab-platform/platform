# Student Collaboration Platform

> A free, open SaaS platform for students and supervisors to manage academic projects — without the limitations of paid tools like Slack or Trello.

![Django](https://img.shields.io/badge/Django-5.x-092E20?style=flat&logo=django&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-eu--west--3-FF9900?style=flat&logo=amazonaws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?style=flat&logo=terraform&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat&logo=python&logoColor=white)

---

## Overview

The **Student Collaboration Platform** is a Master's academic project built as a full-stack cloud-native SaaS application. It provides teams of students and their supervisors with a unified workspace to organise tasks, communicate in real time, and track project progress — all in one place, for free.

### Key features

- **Workspaces** — create shared spaces, invite members, assign roles (admin / member)
- **Kanban boards** — drag-and-drop task management with columns, assignees, and due dates
- **Real-time chat** — live messaging per workspace powered by WebSockets
- **Progress dashboard** — track task completion rates and project activity
- **Freemium model** — core features free, advanced features behind premium tier

---

## Tech stack

| Layer | Technology |
|---|---|
| Backend | Django 5 + Django REST Framework |
| Real-time | Django Channels + Redis (WebSockets) |
| Database | PostgreSQL 16 |
| Task queue | Celery + Celerybeat + Redis |
| Containerisation | Docker + Docker Compose |
| Cloud provider | AWS (region: eu-west-3 — Paris) |
| Infrastructure as Code | Terraform |
| CI/CD | GitHub Actions + AWS CodePipeline |
| Monitoring | AWS CloudWatch |
| Storage | AWS S3 |

---

## Architecture

```
Users
  │
  ▼
CloudFront (CDN)
  │
  ▼
ALB (Application Load Balancer)
  ├── /api/*   → ECS Fargate (Django app)
  └── /ws/*    → ECS Fargate (Django Channels / ASGI)
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
     RDS Postgres  ElastiCache  S3
                   (Redis)   (static/media)

GitHub → CodePipeline → CodeBuild → ECR → ECS
```

---

## Project structure

```
platform/
└── student_collab_platform/
    ├── config/                  # Django settings (base, local, production)
    ├── student_collab_platform/ # Main Django app
    │   ├── users/               # User model and auth
    │   ├── workspaces/          # Workspace and membership management
    │   ├── boards/              # Kanban boards, columns, tasks
    │   └── chat/                # Real-time chat (Django Channels)
    ├── compose/                 # Docker Compose configs
    ├── infra/                   # Terraform infrastructure code
    │   ├── terraform/
    │   │   ├── vpc.tf
    │   │   ├── rds.tf
    │   │   ├── ecs.tf
    │   │   ├── alb.tf
    │   │   └── cloudwatch.tf
    ├── docker-compose.local.yml
    ├── docker-compose.production.yml
    └── manage.py
```

---

## Getting started (local development)

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [Git](https://git-scm.com/)

### Setup

**1. Clone the repository**

```bash
git clone https://github.com/student-collab-platform/platform.git
cd platform
git checkout dev
```

**2. Navigate to the project folder**

```bash
cd student_collab_platform
```

**3. Build the containers**

```bash
docker compose -f docker-compose.local.yml build
```

**4. Run database migrations**

```bash
docker compose -f docker-compose.local.yml run --rm django python manage.py migrate
```

**5. Create a superuser (admin account)**

```bash
docker compose -f docker-compose.local.yml run --rm django python manage.py createsuperuser
```

**6. Start the application**

```bash
docker compose -f docker-compose.local.yml up
```

The app is now running at:

| Service | URL |
|---|---|
| Application | http://localhost:8000 |
| Admin panel | http://localhost:8000/admin |
| Mailpit (email testing) | http://localhost:8025 |
| Flower (Celery monitoring) | http://localhost:5555 |

---

## Running services (Docker Compose)

When you run `docker compose up`, 7 containers start:

| Container | Role |
|---|---|
| `django` | Django web server (ASGI) |
| `postgres` | PostgreSQL database |
| `redis` | Cache + Celery broker + Channel layer |
| `celeryworker` | Background task processor |
| `celerybeat` | Periodic task scheduler |
| `flower` | Celery monitoring dashboard |
| `mailpit` | Local email testing server |

---

## Team

| Role | Responsibility |
|---|---|
| Cloud engineer | AWS infrastructure, Terraform, VPC, RDS, ECS, CodePipeline, CloudWatch |
| Developer | Django backend, DRF APIs, Django Channels, React frontend, Celery tasks |

---

## Branch strategy

```
main   ← production only, protected (requires PR)
dev    ← active development branch
feature/*  ← individual features, merged into dev via PR
```

---

## Cloud infrastructure (AWS)

Infrastructure is fully defined as code using Terraform in the `infra/terraform/` directory.

Resources provisioned:

- VPC with public and private subnets (eu-west-3)
- RDS PostgreSQL (private subnet)
- ElastiCache Redis (private subnet)
- ECS Fargate cluster + task definitions
- Application Load Balancer with WebSocket routing
- S3 bucket for static files and media
- ECR for Docker image registry
- CodePipeline + CodeBuild + CodeDeploy for CI/CD
- CloudWatch dashboards and alarms

To deploy infrastructure:

```bash
cd infra/terraform
terraform init
terraform plan
terraform apply
```

---

## Academic context

This project is developed as a Master's degree capstone project in Network / Cloud / Software Engineering. It demonstrates:

- Cloud-native SaaS architecture
- Infrastructure as Code (Terraform)
- CI/CD pipeline automation
- Real-time communication with WebSockets
- Containerised microservices with Docker and ECS
- Freemium SaaS business model implementation

---

## License

This project is for academic purposes only. Not open source.
