# PetZonic — Infrastructure & Deployment

> **Version**: 1.0.0  
> **Date**: May 28, 2026

---

## 1. AWS Architecture

### Region: ap-south-1 (Mumbai)
Chosen for lowest latency to Indian users and data residency compliance.

---

## 2. Infrastructure Components

### 2.1 Compute

| Service | Resource | Specs (Launch) | Purpose |
|---------|----------|---------------|---------|
| ECS Fargate | API containers | 2 tasks × (1 vCPU, 2GB RAM) | Backend API |
| ECS Fargate | Worker containers | 1 task × (0.5 vCPU, 1GB RAM) | Background job processing |
| EC2 | t3.small | 1 instance | Meilisearch server |

### 2.2 Database

| Service | Resource | Specs (Launch) | Purpose |
|---------|----------|---------------|---------|
| RDS PostgreSQL | db.t3.medium | 2 vCPU, 4GB RAM, 50GB SSD | Primary database |
| ElastiCache Redis | cache.t3.micro | 1 node, 0.5GB | Cache, sessions, queues |

### 2.3 Storage & CDN

| Service | Resource | Purpose |
|---------|----------|---------|
| S3 | Media bucket | Pet photos, product images, documents |
| S3 | Web static bucket | Next.js static assets |
| CloudFront | CDN distribution | Global asset delivery, SSL termination |

### 2.4 Networking

| Service | Resource | Purpose |
|---------|----------|---------|
| VPC | Custom VPC | Network isolation |
| ALB | Application Load Balancer | Request routing, health checks |
| Route 53 | DNS | Domain management |
| ACM | SSL Certificate | HTTPS for all domains |
| NAT Gateway | Single AZ | Outbound internet for private subnets |

### 2.5 Security

| Service | Resource | Purpose |
|---------|----------|---------|
| IAM | Roles + Policies | Service permissions |
| Security Groups | Per-service SGs | Network access control |
| Secrets Manager | API keys, DB passwords | Secret management |
| WAF | Web Application Firewall | DDoS protection, SQL injection blocking |

---

## 3. Network Architecture

```
VPC: 10.0.0.0/16

├── Public Subnet (10.0.1.0/24) - AZ-a
│   ├── ALB
│   └── NAT Gateway
│
├── Public Subnet (10.0.2.0/24) - AZ-b
│   └── ALB (multi-AZ)
│
├── Private Subnet - App (10.0.10.0/24) - AZ-a
│   ├── ECS Tasks (API)
│   └── ECS Tasks (Worker)
│
├── Private Subnet - App (10.0.11.0/24) - AZ-b
│   └── ECS Tasks (API - failover)
│
├── Private Subnet - Data (10.0.20.0/24) - AZ-a
│   ├── RDS Primary
│   ├── ElastiCache
│   └── Meilisearch EC2
│
└── Private Subnet - Data (10.0.21.0/24) - AZ-b
    └── RDS Standby (Multi-AZ)
```

---

## 4. CI/CD Pipeline

### 4.1 Pipeline Architecture

```mermaid
graph LR
    DEV[Developer Push] --> GH[GitHub]
    GH --> GA[GitHub Actions]
    GA --> LINT[Lint + Type Check]
    LINT --> TEST[Unit Tests]
    TEST --> BUILD[Docker Build]
    BUILD --> ECR[Push to ECR]
    ECR --> DEP{Branch?}
    DEP -->|develop| STAGING[Deploy to Staging]
    DEP -->|main| PROD[Deploy to Production]
    STAGING --> SMOKE[Smoke Tests]
    PROD --> HEALTH[Health Check]
```

### 4.2 Environments

| Environment | Purpose | Trigger | URL |
|-------------|---------|---------|-----|
| **Local** | Development | Docker Compose | localhost:3000 |
| **Staging** | Testing & QA | Push to `develop` | staging-api.petzonic.com |
| **Production** | Live users | Push to `main` (manual approve) | api.petzonic.com |

### 4.3 Deployment Strategy
- **Blue-Green deployment** via ECS (zero-downtime)
- Health checks: `/health` endpoint
- Rollback: automatic if health check fails within 5 minutes
- Database migrations: run before deployment (Prisma migrate)

---

## 5. Docker Configuration

### API Dockerfile (Production)
```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

FROM node:22-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/package.json ./
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### Docker Compose (Local Development)
```yaml
services:
  api:
    build: ./backend
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/petzonic
      REDIS_URL: redis://redis:6379
      MEILISEARCH_URL: http://meilisearch:7700
    depends_on: [db, redis, meilisearch]

  db:
    image: postgres:16-alpine
    ports: ["5432:5432"]
    environment:
      POSTGRES_DB: petzonic
      POSTGRES_PASSWORD: postgres
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  meilisearch:
    image: getmeili/meilisearch:v1
    ports: ["7700:7700"]
    environment:
      MEILI_MASTER_KEY: devMasterKey123
    volumes: [msdata:/meili_data]

volumes:
  pgdata:
  msdata:
```

---

## 6. Domain & SSL Setup

| Domain | Service | Purpose |
|--------|---------|---------|
| petzonic.com | CloudFront → S3/Next.js | Customer website |
| admin.petzonic.com | CloudFront → S3/Next.js | Admin panel |
| api.petzonic.com | ALB → ECS | Backend API |
| media.petzonic.com | CloudFront → S3 | Image/video CDN |
| seller.petzonic.com | CloudFront → S3/Next.js | Seller web portal (future) |

All domains: ACM-managed SSL certificates (auto-renewal).

---

## 7. Backup & Disaster Recovery

| Component | Backup Strategy | Recovery |
|-----------|----------------|----------|
| PostgreSQL (RDS) | Automated daily snapshots + continuous backup (PITR) | Restore to any point within 7 days |
| Redis | No backup (cache-only, rebuilds from DB) | Restart and rebuild |
| S3 (Media) | Versioning enabled + cross-region replication (future) | Restore previous version |
| Meilisearch | Daily data dump to S3 | Rebuild index from PostgreSQL |
| Application Code | GitHub (source of truth) | Redeploy from Git |
| Secrets | AWS Secrets Manager (auto-rotation) | Managed by AWS |

**RTO (Recovery Time Objective)**: < 1 hour  
**RPO (Recovery Point Objective)**: < 5 minutes (database PITR)

---

## 8. Cost Estimation (Monthly)

### Launch Phase (0-5K users)

| Service | Configuration | Est. Cost/month |
|---------|--------------|-----------------|
| ECS Fargate (API) | 2 tasks × 1vCPU/2GB | ₹4,000 |
| ECS Fargate (Worker) | 1 task × 0.5vCPU/1GB | ₹1,500 |
| RDS PostgreSQL | db.t3.medium, 50GB | ₹5,000 |
| ElastiCache Redis | cache.t3.micro | ₹1,500 |
| EC2 (Meilisearch) | t3.small | ₹1,500 |
| S3 + CloudFront | 50GB storage, 100GB transfer | ₹1,000 |
| ALB | Per hour + LCU | ₹2,000 |
| Route 53 | Hosted zone + queries | ₹200 |
| NAT Gateway | Per hour + data | ₹3,000 |
| Secrets Manager | 10 secrets | ₹500 |
| **Total** | | **~₹20,000/month** |

### Growth Phase (5K-50K users) — add:
- ECS auto-scaling (2-6 tasks): +₹8,000
- RDS read replica: +₹5,000
- ElastiCache upgrade: +₹2,000
- S3/CDN increased usage: +₹3,000
- **Total**: ~₹38,000/month

---

## 9. Monitoring & Alerting

### CloudWatch Alarms

| Metric | Threshold | Action |
|--------|-----------|--------|
| API 5xx errors | > 10/min | Alert to Slack + PagerDuty |
| API latency p95 | > 2000ms | Alert to Slack |
| CPU utilization (ECS) | > 80% for 5min | Auto-scale up |
| CPU utilization (ECS) | < 20% for 15min | Auto-scale down |
| RDS CPU | > 80% | Alert + investigate |
| RDS storage | < 5GB free | Alert to expand |
| Redis memory | > 80% | Alert |
| Health check | Unhealthy | Auto-replace task |

### Logging Structure (JSON)
```json
{
  "timestamp": "2026-05-28T10:30:00Z",
  "level": "info",
  "service": "petzonic-api",
  "traceId": "abc-123",
  "userId": "user_456",
  "method": "POST",
  "path": "/api/v1/pets",
  "statusCode": 201,
  "responseTime": 145,
  "message": "Pet listing created"
}
```

---

## 10. Scaling Triggers

| Metric | Scale Up | Scale Down |
|--------|----------|-----------|
| CPU Avg | > 70% for 3min | < 30% for 10min |
| Request count | > 1000 req/min | < 200 req/min |
| Min tasks | 2 | 2 |
| Max tasks | 8 | 2 |
| Cooldown | 60s | 300s |

---

## 11. Infrastructure as Code (Terraform)

All infrastructure managed via Terraform:
```
infrastructure/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── modules/
│   │   ├── vpc/
│   │   ├── ecs/
│   │   ├── rds/
│   │   ├── redis/
│   │   ├── s3/
│   │   ├── cloudfront/
│   │   ├── alb/
│   │   └── iam/
│   └── environments/
│       ├── staging.tfvars
│       └── production.tfvars
```

**Workflow**: Code change → PR review → `terraform plan` (in CI) → Approve → `terraform apply`
