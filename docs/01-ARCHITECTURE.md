# Donation Management CMS - System Architecture

## Overview
A complete web-based donation management system for NGOs with separate donor portal and admin CMS. Built with PHP-Laravel, PostgreSQL, and containerized with Docker.

---

## System Architecture

### Option A: Monolithic Architecture (Initial MVP - Recommended for 4-6 weeks)

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend Layer                         │
│  ┌──────────────┐         ┌──────────────┐              │
│  │ Donor Portal │         │  Admin CMS   │              │
│  │  (Vue.js)    │         │  (Vue.js)    │              │
│  └──────┬───────┘         └──────┬───────┘              │
└─────────┼──────────────────────────┼────────────────────┘
          │                          │
          └──────────────┬───────────┘
                         │
┌─────────────────────────▼─────────────────────────────────┐
│                    API Layer (Laravel)                     │
│  REST API Routes & Controllers                             │
│  - /api/auth (login, register, OTP)                        │
│  - /api/donations (create, list, history)                  │
│  - /api/donors (profile, preferences)                      │
│  - /api/campaigns (list, details)                          │
│  - /api/admin/* (management endpoints)                     │
│  - /api/documents (receipts, certificates)                │
└─────────────────────────┬─────────────────────────────────┘
          │
┌─────────┴────────────────────────────────────────────────┐
│                    Business Logic Layer                    │
│  ┌─────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │  Services   │  │  Repositories  │  │   Factories  │  │
│  ├─────────────┤  ├────────────────┤  ├──────────────┤  │
│  │ DonationSvc │  │ DonationRepo   │  │ DocumentFact │  │
│  │ PaymentSvc  │  │ DonorRepo      │  │ EmailFact    │  │
│  │ RecurringSvc│  │ CampaignRepo   │  └──────────────┘  │
│  │ EmailSvc    │  │ PaymentRepo    │                      │
│  │ DocumentSvc │  └────────────────┘                      │
│  └─────────────┘                                           │
└─────────────────────────┬─────────────────────────────────┘
          │
┌─────────┴────────────────────────────────────────────────┐
│              Queue & Job Processing                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │Email Queue   │  │PDF Generation│  │Recurring Chg │   │
│  │(Redis)       │  │Jobs          │  │Jobs          │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────┬─────────────────────────────────┘
          │
┌─────────┴────────────────────────────────────────────────┐
│           Data Persistence & External Services            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ PostgreSQL   │  │    Redis     │  │ Payment GW   │   │
│  │   (Master)   │  │  (Cache)     │  │  (Stripe/etc)│   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐                      │
│  │ File Storage │  │  Email Svc   │                      │
│  │  (PDF/docs)  │  │  (SMTP/SES)  │                      │
│  └──────────────┘  └──────────────┘                      │
└─────────────────────────────────────────────────────────┘
```

### Option B: Microservices Architecture (Future Scalability)

```
┌──────────────────────────────────────────────────────────┐
│  API Gateway (Kong/Traefik)                               │
└───┬──────┬──────┬──────┬────────┬────────────────────────┘
    │      │      │      │        │
    ▼      ▼      ▼      ▼        ▼
  ┌──────────┐ ┌──────────┐ ┌──────────────┐
  │Auth Svc  │ │Donation  │ │Payment       │
  │          │ │Svc       │ │Service       │
  ├──────────┤ ├──────────┤ ├──────────────┤
  │ Login    │ │ Create   │ │ Webhook      │
  │ OTP      │ │ List     │ │ Process      │
  │ Register │ │ Recurring│ │ Refund       │
  └──────────┘ └──────────┘ └──────────────┘
  ┌──────────┐ ┌──────────┐ ┌──────────────┐
  │Document  │ │Email     │ │Report        │
  │Service   │ │Service   │ │Service       │
  ├──────────┤ ├──────────┤ ├──────────────┤
  │PDF Gen   │ │Send      │ │Analytics     │
  │Store     │ │Queue     │ │Export        │
  │Download  │ │Template  │ │Audit         │
  └──────────┘ └──────────┘ └──────────────┘

  ┌────────────────────────────────────────┐
  │  Shared Services                       │
  │  - Message Queue (RabbitMQ)            │
  │  - Cache (Redis)                       │
  │  - Database (PostgreSQL)               │
  │  - File Storage (S3)                   │
  └────────────────────────────────────────┘
```

**Recommendation:** Start with Option A for MVP (faster, simpler), then migrate to Option B for scaling.

---

## Technology Stack

### Option A: Monolithic (MVP)
- **Backend:** PHP 8.2 + Laravel 10
- **Frontend:** Vue.js 3 + Vite (SPA for both portal and CMS)
- **Database:** PostgreSQL 15
- **Cache/Queue:** Redis 7
- **Job Processing:** Laravel Queue (Redis driver)
- **File Storage:** Local FS + S3 (optional)
- **PDF Generation:** DomPDF or TCPDF
- **Email:** Laravel Mail (SMTP/SES)
- **Authentication:** JWT + Laravel Sanctum
- **Containerization:** Docker + Docker Compose
- **Web Server:** Nginx
- **Development:** PHPUnit, Pest, Laravel Dusk

### Option B: Microservices (Scalable)
- **Each Service:** PHP 8.2 + Slim/Lumen
- **API Gateway:** Kong or Traefik
- **Message Queue:** RabbitMQ
- **Cache:** Redis Cluster
- **Database:** PostgreSQL (one per service)
- **Orchestration:** Kubernetes (optional)
- **Service Discovery:** Consul or Kubernetes DNS
- **Monitoring:** Prometheus + Grafana

---

## Containerization Strategy

### Docker Compose Services
1. **PostgreSQL** - Primary datastore
2. **Redis** - Cache, sessions, queue
3. **Laravel App** - Main application (php-fpm)
4. **Nginx** - Web server
5. **Queue Worker** - Background job processor
6. **Scheduler** - Cron job runner
7. **MailHog** - Email testing (dev only)

### Development Workflow
```bash
cd donation-cms
cp .env.example .env
docker-compose up -d
docker-compose exec app php artisan migrate
docker-compose exec app php artisan seed:run
# Visit http://localhost:80
```

### Production Deployment
- Use managed database (AWS RDS PostgreSQL)
- Use managed Redis (AWS ElastiCache)
- Deploy Laravel app on AWS ECS/EKS
- Use S3 for file storage
- CloudFront for CDN
- SES or SendGrid for email
- Stripe/Razorpay for payments

---

## Key Design Patterns

### 1. Repository Pattern
- Abstraction between controllers and database
- Example: `DonationRepository`, `DonorRepository`

### 2. Service Layer
- Business logic encapsulation
- Example: `DonationService`, `RecurringBillingService`

### 3. Queue Pattern
- Async job processing
- Email notifications
- PDF generation
- Recurring charge processing

### 4. Event-Driven Architecture
- `DonationCreated`, `DonationConfirmed`, `PaymentFailed` events
- Listeners for side effects (send emails, generate PDFs, log audit)

### 5. Middleware for Cross-Cutting Concerns
- Authentication
- Role-based access control (RBAC)
- Rate limiting
- Request logging

### 6. Factory Pattern
- `DocumentFactory` for creating different document types
- `EmailFactory` for creating email instances

---

## Data Flow Examples

### Example 1: One-Time Donation Flow
```
1. Donor submits donation form
   ↓
2. API validates input & creates donation record
   ↓
3. Payment gateway processes payment (async webhook)
   ↓
4. Webhook received → Donation marked as CONFIRMED
   ↓
5. Events triggered: DonationConfirmed
   ↓
6. Listeners execute:
   a) GenerateReceiptPDF job queued
   b) SendReceiptEmail job queued
   c) UpdateDonorStats (lifetime value)
   ↓
7. Queue workers execute async jobs
   ↓
8. Donor receives receipt email with PDF
```

### Example 2: Recurring Donation Flow
```
1. Donor creates recurring plan (monthly)
   ↓
2. System records plan with next_charge_date
   ↓
3. Scheduler runs at midnight (Laravel schedule)
   ↓
4. RecurringBillingService finds due plans
   ↓
5. For each plan:
   a) Call payment gateway
   b) Create transaction record
   c) Handle success/failure
   ↓
6. If success:
   - Create donation record
   - Queue receipt & email
   - Update next_charge_date
   ↓
7. If failure:
   - Increment retry count
   - Queue retry for next day (max 3 retries)
   - Notify donor if final failure
   ↓
8. If cancelled by donor:
   - Mark plan as CANCELLED
   - Send cancellation confirmation
```

### Example 3: Admin Generates Annual Report
```
1. Admin selects date range & campaign filter
   ↓
2. API queries aggregated data
   ↓
3. ReportService compiles data:
   a) Total donations (one-time + recurring)
   b) Donor count
   c) Campaign-wise breakdown
   d) Donor-wise totals
   ↓
4. GenerateReportPDF job queued
   ↓
5. Queue worker:
   a) Uses ReportPDFTemplate
   b) Inserts data
   c) Stores in storage/app/reports/
   ↓
6. API returns download link
   ↓
7. Admin downloads report (or sends to stakeholders)
```

---

## Security Architecture

### Authentication Flow
```
Donor/Admin Login
    ↓
Validate email + password
    ↓
Generate JWT token + Refresh token
    ↓
Store refresh token in Redis (with TTL)
    ↓
Return tokens to frontend
    ↓
Frontend stores JWT in memory (not localStorage)
    ↓
Send JWT in Authorization header for API calls
    ↓
Middleware verifies JWT signature
    ↓
Extract user from token
```

### Authorization (RBAC)
```
Middleware: CheckRole, CheckPermission
    ↓
User roles: DONOR, ADMIN, FINANCE_OFFICER, CONTENT_MANAGER
    ↓
Each endpoint defines required roles
    ↓
If unauthorized → 403 Forbidden
    ↓
Audit log created
```

### Sensitive Data Protection
- Encrypt: tax_id, bank_account (at rest)
- Hash: passwords (bcrypt)
- Mask: payment card numbers (show last 4 digits only)
- HTTPS only (force redirect)
- CORS restrictions
- CSRF tokens for form submissions

---

## Database Scaling Strategy

### Initial (MVP)
- PostgreSQL single instance
- Local backups via Docker volume
- Read replicas (optional)

### Growth
- PostgreSQL with read replicas
- Connection pooling (PgBouncer)
- Partitioning large tables (donations, audit_logs)
- Caching layer (Redis)

### Enterprise
- Managed database (AWS RDS with Multi-AZ)
- Elasticsearch for reporting/analytics
- Data warehouse (Snowflake/BigQuery) for BI
- CDC (Change Data Capture) for real-time sync

---

## Monitoring & Observability

### Logging
- Application logs → ELK Stack or CloudWatch
- Database queries (slow query logs)
- Payment gateway interactions (sensitive data masked)

### Metrics
- Request latency (P50, P95, P99)
- Queue depth
- Payment success rate
- Email delivery rate
- Database connection pool utilization

### Tracing
- Correlation IDs for request tracking
- End-to-end tracing of donation flow

### Alerts
- Queue size > threshold
- Payment failures spike
- Database connection pool exhaustion
- Disk space low

---

## Disaster Recovery

### Backup Strategy
- Daily automated PostgreSQL backups
- 30-day retention
- Monthly snapshot to cold storage
- Test restore procedures monthly

### Business Continuity
- RTO (Recovery Time Objective): 4 hours
- RPO (Recovery Point Objective): 1 hour
- Failover to standby database
- Load balancer auto-failover

---

## Cost Optimization

### Development
- Docker Compose local (free)
- MailHog for email testing (free)

### Production (AWS Example)
- RDS PostgreSQL: ~$200-500/month
- ElastiCache Redis: ~$100-200/month
- ECS Fargate: ~$100-300/month (varies with traffic)
- S3 Storage: ~$0.023/GB/month
- Estimated monthly: $400-1000

---

## Deployment Pipeline

```
GitHub Push
    ↓
GitHub Actions Trigger
    ↓
Run Tests (PHPUnit + Pest)
    ↓
Run Linting (Laravel Pint)
    ↓
Build Docker Image
    ↓
Push to ECR (AWS)
    ↓
Deploy to ECS/EKS
    ↓
Run Migrations (blue-green)
    ↓
Health Checks
    ↓
Production Live
```

---

## Next Steps

See the following documentation files for details:
1. [Database Schema](02-DATABASE-SCHEMA.md)
2. [API Design](03-API-DESIGN.md)
3. [User Journeys](04-USER-JOURNEYS.md)
4. [Background Jobs](05-BACKGROUND-JOBS.md)
5. [Security & Compliance](06-SECURITY-COMPLIANCE.md)
6. [MVP Roadmap](07-MVP-ROADMAP.md)
