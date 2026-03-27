# Amazon Lightsail — Cloud Migration Proposal

> A turnkey cloud migration plan for SMB clients including architecture diagram, 6-month utilization plan, and detailed use case.

---

## Scenario

**Client:** Cross-border e-commerce / SaaS startup (revenue < USD 10M)  
**Requirement:** Migrate existing web application to cloud, reduce operational overhead, improve availability

---

## 1. Architecture Diagram

```
Internet
    │
    ▼
┌─────────────────────────────────────────────────────┐
│                  Amazon Lightsail                    │
│                                                      │
│  ┌──────────────┐     ┌──────────────────────────┐  │
│  │  Static IP   │────▶│  Lightsail Load Balancer  │  │
│  └──────────────┘     │  (HTTPS / SSL termination)│  │
│                       └────────────┬──────────────┘  │
│                                    │                  │
│              ┌─────────────────────┼────────────┐    │
│              ▼                     ▼            │    │
│  ┌────────────────┐   ┌────────────────┐        │    │
│  │  Web Instance  │   │  Web Instance  │        │    │
│  │  2 vCPU / 4GB  │   │  2 vCPU / 4GB  │        │    │
│  │  $20/mo each   │   │  $20/mo each   │        │    │
│  └───────┬────────┘   └───────┬────────┘        │    │
│          └─────────┬──────────┘                 │    │
│                    ▼                             │    │
│        ┌───────────────────────┐                │    │
│        │  Lightsail Database   │                │    │
│        │  MySQL 8.0 Standard   │                │    │
│        │  1GB RAM / 40GB SSD   │                │    │
│        │  $15/mo               │                │    │
│        └───────────────────────┘                │    │
│                                                  │    │
│  ┌───────────────────────────────────────────┐  │    │
│  │  Lightsail Object Storage (S3-compatible) │  │    │
│  │  Media files / Static assets  5GB  $1/mo │  │    │
│  └───────────────────────────────────────────┘  │    │
│                                                      │
│  VPC Peering (optional) → Amazon S3 / SES / CF      │
└─────────────────────────────────────────────────────┘

Automated Snapshots → Daily, 7-day retention
```

### Key Design Principles

- **High Availability** — Dual web instances behind a Load Balancer
- **Separation of Concerns** — Dedicated managed database, decoupled from compute
- **Performance** — Static assets offloaded to Object Storage to reduce instance pressure
- **Security** — Static IP + free SSL/TLS certificate (included with Lightsail Load Balancer)
- **Scale-Out IP Strategy** — 500 Static IPs requested in Month 1 to support multi-tenant isolation or proxy use cases

### Static IP Quota Increase — Month 1 Action Plan

> **Default Lightsail quota:** 5 Static IPs per account per region  
> **Target:** 500 Static IPs

| Step | Action | Owner | Timeline |
|------|--------|-------|----------|
| 1 | Submit quota increase via [AWS Service Quotas console](https://console.aws.amazon.com/servicequotas/) → Lightsail → Static IP addresses | Cloud Admin | Day 1 |
| 2 | If Service Quotas not available for Lightsail, open AWS Support case (Limit Increase) with justification | Cloud Admin | Day 1–2 |
| 3 | AWS reviews and approves (typically 1–5 business days) | AWS Support | Day 2–7 |
| 4 | Provision IPs in batches; assign to instances as they come online | Cloud Admin | Week 2–4 |

> ⚠️ **Note:** For 500+ IPs at scale, consider supplementing with **Amazon EC2 Elastic IPs** (also quota-adjustable) via VPC Peering from Lightsail. EC2 EIPs offer finer-grained routing control and are better suited for large IP pools. Unattached Static IPs in Lightsail are billed at **$0.005/hr** (~$3.60/mo each).

---

## 2. 6-Month Utilization Plan

| Month | Phase | Key Activities | Est. Monthly Cost |
|-------|-------|---------------|-------------------|
| **Month 1** | Migration Prep + IP Provisioning | Provision environment, single-instance testing, data migration validation; **submit Service Quota increase request for 500 Static IPs** (via AWS Support or Service Quotas console); begin staged IP assignment to instances | ~$36 + $0.005/hr per unattached IP |
| **Month 2** | Go-Live | DNS cutover, Load Balancer onboarding, dual-instance scale-out, load testing | ~$72 |
| **Month 3** | Steady State | Configure alarms & alerts, enable auto-snapshots, establish performance baseline | ~$90 |
| **Month 4** | Optimization | Migrate static assets to Object Storage, configure CloudFront CDN | ~$95 |
| **Month 5** | Growth | Evaluate instance upgrade (4GB → 8GB) based on traffic; DB HA if needed | ~$95–$140 |
| **Month 6** | Review & Roadmap | ROI review; assess migration to EC2/ECS if business outgrows Lightsail | ~$140 |

### Resource Breakdown (Steady State)

| Resource | Spec | Monthly Cost |
|----------|------|-------------|
| Web Instance ×2 | 2 vCPU / 4GB RAM / 80GB SSD | $40 |
| Load Balancer | Up to 5 targets, 1 TLS cert | $18 |
| Managed Database | MySQL 8.0 Standard / 1GB RAM | $15 |
| Object Storage | 5GB | $1 |
| Automated Snapshots | Daily, 7-day retention | ~$2 |
| **Total** | | **≈ $76/month** |

> **Data Transfer:** Each instance includes 2TB outbound/month. Overage billed at $0.09/GB.

---

## 3. Detailed Use Case

### Cross-Border E-Commerce Website + Admin Portal Migration

#### Business Background

A Greater Bay Area cross-border e-commerce company currently runs its corporate website, WordPress blog, and admin panel on a domestic VPS. Key pain points:

- High latency for overseas visitors
- No automated backup or disaster recovery strategy
- Growing operational burden on in-house IT team

#### Objectives

| Goal | Target |
|------|--------|
| Global website latency | < 200ms |
| System availability | > 99.5% |
| Monthly cloud spend | < $100 |

#### Phase-by-Phase Execution

| Plan Phase | Actions |
|-----------|---------|
| **Month 1** – Migration Prep | Spin up Lightsail WordPress instance; import existing database; validate full functionality in staging |
| **Month 2** – Go-Live | Cut over DNS to Lightsail Static IP; activate Load Balancer; run both environments in parallel for 2 weeks as fallback |
| **Month 3** – Steady State | Configure Lightsail alarms (CPU > 80%, connection thresholds) with email/SNS notifications; enable daily automated snapshots |
| **Month 4** – Optimization | Migrate media files to Object Storage; configure Amazon CloudFront for global edge caching (~40% latency reduction for overseas users) |
| **Month 5** – Growth | Pre-scale for peak season: upgrade Web instances to 8GB plan; upgrade DB to High Availability ($30/mo) if uptime SLA requires; roll back after traffic normalizes |
| **Month 6** – Review & Roadmap | If DAU > 5,000 or peak concurrency > 200, plan migration to EC2 + RDS; Lightsail VPC Peering enables seamless transition with no re-architecture required |

#### Key Value Propositions

| | |
|--|--|
| ✅ **Ready out-of-the-box** | Pre-configured blueprints (WordPress, LAMP, Node.js); production-ready in under an hour |
| ✅ **Predictable billing** | Fixed monthly pricing, no bill shock; easy to present to finance and procurement teams |
| ✅ **Clear upgrade path** | VPC Peering to EC2/RDS when the business scales — no vendor lock-in, no re-architecture |
| ✅ **Built-in resilience** | Automated daily snapshots satisfy basic DR requirements from day one |

---

## References

- [Amazon Lightsail Documentation](https://docs.aws.amazon.com/lightsail/)
- [Lightsail Pricing](https://aws.amazon.com/lightsail/pricing/)
- [Lightsail to EC2 Migration Guide](https://docs.aws.amazon.com/lightsail/latest/userguide/migrate-your-lightsail-instance-to-amazon-ec2.html)
- [CloudFront + Lightsail Integration](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-cloudfront-support-for-amazon-lightsail/)
