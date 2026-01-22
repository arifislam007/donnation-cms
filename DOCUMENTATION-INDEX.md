# Donation Management CMS - Complete Documentation Index

## 📋 Project Overview

**System:** Complete web-based donation management platform for NGOs  
**Stack:** PHP 8.2 + Laravel 10 + Vue.js 3 + PostgreSQL 15 + Redis 7 + Docker  
**Scope:** Monolithic architecture suitable for MVP (4-6 weeks development)  
**Location:** `f:/Personal-Project/donation-cms/`

---

## 📚 Core Documentation

### 1. [Architecture Design](docs/01-ARCHITECTURE.md)
**Purpose:** Understand system design and technology choices

**Key Sections:**
- System architecture overview (monolithic vs microservices)
- Technology stack recommendations
- Docker containerization strategy
- Data flow examples (donation, recurring billing, reporting)
- Security architecture
- Database scaling strategy
- Monitoring & observability
- Cost optimization
- Deployment pipeline

**When to Read:** First - get familiar with the system structure

---

### 2. [Database Schema](docs/02-DATABASE-SCHEMA.md)
**Purpose:** Complete database design with all tables, relationships, and constraints

**Key Tables:**
- `users` - Authentication
- `donor_profiles` - Extended donor information
- `donor_preferences` - Communication preferences
- `campaigns` - Fundraising campaigns
- `donations` - All donation records
- `recurring_plans` - Subscription plans
- `payment_transactions` - Payment history
- `documents` - Receipts, certificates, statements
- `email_logs` - Email sending history
- `templates` - Email and document templates
- `audit_logs` - Complete audit trail
- `donation_receipts` - Cached receipt data
- `fiscal_year_configs` - Bangladesh fiscal year setup
- `recurring_payment_logs` - Detailed charge logs
- `document_downloads` - Access tracking

**Indexes:** Performance-critical indexes included  
**Encryption:** Sensitive fields with pgcrypto  
**Partitioning:** Strategy for scaling  

**When to Read:** Second - understand data model and relationships

---

### 3. [REST API Design](docs/03-API-DESIGN.md)
**Purpose:** Complete API specification with request/response examples

**API Endpoints by Module:**

#### Authentication (Public)
- `POST /auth/register` - Donor registration
- `POST /auth/verify-otp` - Email verification
- `POST /auth/login` - Login
- `POST /auth/refresh-token` - Token refresh
- `POST /auth/logout` - Logout
- `POST /auth/forgot-password` - Password reset request
- `POST /auth/reset-password` - Reset password

#### Donor Portal
- `GET /donors/profile` - View profile
- `PUT /donors/profile` - Update profile
- `GET /donors/preferences` - View preferences
- `PUT /donors/preferences` - Update preferences
- `GET /campaigns` - List campaigns
- `GET /campaigns/{id}` - Campaign details
- `POST /donations/create-one-time` - Create one-time donation
- `POST /donations/create-recurring` - Create recurring plan
- `GET /donations/history` - Donation history
- `GET /donations/{id}` - Donation details
- `GET /recurring-plans` - List recurring plans
- `PUT /recurring-plans/{id}` - Update plan
- `POST /recurring-plans/{id}/pause` - Pause plan
- `POST /recurring-plans/{id}/resume` - Resume plan
- `POST /recurring-plans/{id}/cancel` - Cancel plan
- `GET /documents/receipt/{id}` - Download receipt
- `GET /documents/statement` - Download statement
- `GET /documents/certificate/{year}` - Download certificate
- `GET /documents/list` - List documents

#### Admin CMS
- `GET /admin/donors` - List all donors
- `GET /admin/donors/{id}` - Donor details
- `POST /admin/donors/{id}/suspend` - Suspend donor
- `POST /admin/donors/{id}/reactivate` - Reactivate donor
- `POST /admin/donors/merge` - Merge duplicates
- `GET /admin/donors/export-csv` - Export donors
- `GET /admin/donations` - List donations
- `POST /admin/donations/manual-entry` - Create manual donation
- `POST /admin/donations/{id}/verify` - Verify donation
- `POST /admin/donations/{id}/refund` - Refund donation
- `POST /admin/campaigns` - Create campaign
- `PUT /admin/campaigns/{id}` - Update campaign
- `POST /admin/campaigns/{id}/publish` - Publish campaign
- `GET /admin/recurring-plans` - List recurring plans
- `POST /admin/recurring-plans/{id}/retry-failed` - Retry charge
- `GET /admin/reports/daily-collections` - Daily report
- `GET /admin/reports/campaign-performance` - Campaign report
- `GET /admin/reports/donor-lifetime-value` - Donor LTV report
- `GET /admin/reports/tax-year-statement` - Tax report
- `GET /admin/templates` - List templates
- `POST /admin/templates` - Create template
- `GET /admin/email-logs` - Email logs
- `GET /admin/audit-logs` - Audit logs

#### Webhooks
- `POST /webhooks/payment-gateway/success` - Payment success
- `POST /webhooks/payment-gateway/failed` - Payment failure

**Response Format:** Standardized JSON with pagination, meta data, errors  
**Error Codes:** Complete list with HTTP status mappings  

**When to Read:** Third - understand available endpoints and their usage

---

### 4. [User Journeys & Screens](docs/04-USER-JOURNEYS.md)
**Purpose:** Visual flows and UI specifications for both donor portal and admin CMS

**Donor Journeys:**
1. First-Time Donor Registration & One-Time Donation (12 steps)
2. Recurring (Monthly) Donation Setup (11 steps)
3. Download Annual Statement & Certificate (10 steps)

**Admin Journeys:** (Implied in screen specs)

**Screen Specifications:**

**Donor Portal:**
- Screen 1: Homepage / Landing Page
- Screen 2: Donor Registration Form
- Screen 3: OTP Verification
- Screen 4: Donor Dashboard (Main Hub)
- Screen 5: Campaign List & Details
- Screen 6: Donation Form (One-Time)
- Screen 7: Recurring Donation Form
- Screen 8: Donation Success
- Screen 9: Donation History / List
- Screen 10: Recurring Plans Management
- Screen 11: Profile Management
- Screen 12: Communication Preferences

**Admin CMS:**
- Screen A1: Admin Dashboard
- Screen A2: Donor Management List
- Screen A3: Donation Management
- Screen A4: Campaign Management
- Screen A5: Reports
- Screen A6: Template Management

**Responsive Design:** Mobile (< 768px), Tablet (768-1024px), Desktop (1920px+)  
**Accessibility:** WCAG 2.1 AA compliance

**When to Read:** Fourth - understand user experience and screen designs

---

### 5. [Background Jobs & Queues](docs/05-BACKGROUND-JOBS.md)
**Purpose:** Asynchronous job processing, email queues, PDF generation, recurring billing

**Email Queue Jobs:**
1. `SendDonationReceiptEmail` - Email receipt after donation
2. `SendThankYouLetterEmail` - Thank you message (delayed 1 hour)
3. `SendMonthlyDonationSummaryEmail` - Monthly summary (1st of month)
4. `SendAnnualStatementEmail` - Annual statement (fiscal year end)
5. `SendPaymentFailureNotificationEmail` - Payment failed alert

**PDF Generation Jobs:**
6. `GenerateReceiptPDF` - Receipt document
7. `GenerateDonationStatementPDF` - Annual or custom range statement
8. `GenerateDonationCertificatePDF` - Tax certificate for fiscal year

**Recurring Billing Jobs:**
9. `ProcessRecurringDonations` - Daily charge (00:00 IST)
10. `RetryFailedRecurringCharges` - Retry failed charges (14:00 IST)
11. `CheckRecurringSubscriptionExpiry` - Check ended plans (06:00 IST)

**Maintenance Jobs:**
12. `CleanupExpiredOTPs` - Hourly cleanup
13. `ArchiveOldAuditLogs` - Monthly archive

**Queue Configuration:**
- Driver: Redis
- Worker: Laravel Queue worker (docker service)
- Scheduler: Laravel Scheduler (docker service)
- Retry: Exponential backoff
- Monitoring: Failed jobs table

**When to Read:** Fifth - understand async task processing and job queues

---

### 6. [Security & Compliance](docs/06-SECURITY-COMPLIANCE.md)
**Purpose:** Security architecture, RBAC, encryption, audit trails, compliance

**Role-Based Access Control (RBAC):**
- `DONOR` - Self-service donor portal only
- `ADMIN` - Full system access
- `FINANCE_OFFICER` - Financial operations
- `CONTENT_MANAGER` - Campaign and template management

**Security Features:**
- JWT token-based authentication
- Refresh token rotation
- HTTPS/TLS 1.3 enforcement
- HSTS headers
- CSRF protection
- CORS configuration
- SQL injection prevention (parameterized queries)
- XSS protection
- Rate limiting (API, login)
- CAPTCHA for signup
- Webhook signature verification
- IP whitelisting (webhooks)
- Sensitive data encryption (at-rest)
- Payment card masking
- Audit logging (all operations)
- Data retention policies
- Right to be forgotten (GDPR-like)

**Compliance:**
- Bangladesh NGO requirements
- FCRA compliance (if applicable)
- Tax compliance (7-year retention)
- Annual reporting
- Donation receipt issuance
- Data localization

**Testing & Monitoring:**
- Penetration testing checklist
- Incident response procedures
- Backups & disaster recovery

**When to Read:** Sixth - understand security requirements and compliance

---

### 7. [MVP Roadmap](docs/07-MVP-ROADMAP.md)
**Purpose:** 4-6 week development timeline and feature prioritization

**Weekly Breakdown:**

| Week | Focus | Deliverable |
|------|-------|-------------|
| 1 | Foundation & Auth | Docker stack, Auth working |
| 2 | Profiles & Campaigns | Donor profiles, Campaign management |
| 3 | One-Time Donations | Payment gateway, Donations, Receipts |
| 4 | PDF Generation | Receipts, Statements, Certificates |
| 5 | Recurring & Reports | Recurring donations, Admin reporting |
| 6 | Polish & Deploy | Testing, Documentation, Launch |

**Must-Have Features (MVP):**
- User registration with OTP
- Donor/Admin login
- Profile management
- Campaigns
- One-time donations
- Payment gateway
- Receipt PDFs
- Donation history
- Email notifications
- Admin dashboard
- Audit logging
- Email queue

**Should-Have Features (MVP+):**
- Recurring donations
- Statements & certificates
- Reporting
- Scheduler
- Thank you letters

**Won't-Have (Post-MVP):**
- Microservices
- Multiple payment gateways
- Mobile apps
- Advanced analytics

**Resource Requirement:** 3-4.5 FTE for 6 weeks  
**Risk Mitigation:** Comprehensive checklist  
**Success Metrics:** Technical and business KPIs

**When to Read:** Last - understand project timeline and phasing

---

## 📁 Key Files by Function

### Configuration Files
- `.env.example` - Environment variables template
- `docker-compose.yml` - Docker services configuration
- `docker/Dockerfile` - Laravel app container
- `docker/nginx/default.conf` - Nginx routing configuration
- `docker/nginx/nginx.conf` - Nginx server config
- `docker/php/php.ini` - PHP settings
- `docker/postgres/init.sql` - PostgreSQL initialization

### Laravel Scaffolding (To Be Created)
```
app/
├── Http/Controllers/
│   ├── AuthController.php
│   ├── DonationController.php
│   ├── CampaignController.php
│   ├── AdminController.php
│   └── ...
├── Models/
│   ├── User.php
│   ├── Donation.php
│   ├── RecurringPlan.php
│   └── ...
├── Services/
│   ├── PaymentService.php
│   ├── DonationService.php
│   ├── DocumentService.php
│   └── ...
├── Jobs/
│   ├── SendDonationReceiptEmail.php
│   ├── GenerateReceiptPDF.php
│   └── ...
└── ...

database/
├── migrations/
│   ├── create_users_table.php
│   ├── create_donations_table.php
│   └── ...
└── seeders/
    ├── UserSeeder.php
    └── ...

resources/
├── views/
│   ├── emails/
│   │   ├── receipt.blade.php
│   │   └── ...
│   └── pdfs/
│       ├── receipt.blade.php
│       └── ...
└── ...
```

---

## 🚀 Getting Started

### Quick Start
```bash
cd f:/Personal-Project/donation-cms
docker-compose up -d
docker-compose exec app php artisan migrate
docker-compose exec app php artisan seed:run
# Access http://localhost
```

### Reading Order
1. **README.md** - Overview and quick start
2. **docs/01-ARCHITECTURE.md** - System design
3. **docs/02-DATABASE-SCHEMA.md** - Data model
4. **docs/03-API-DESIGN.md** - API specification
5. **docs/04-USER-JOURNEYS.md** - UI/UX flows
6. **docs/05-BACKGROUND-JOBS.md** - Job processing
7. **docs/06-SECURITY-COMPLIANCE.md** - Security details
8. **docs/07-MVP-ROADMAP.md** - Implementation timeline

### Development Flow
1. Setup environment: `docker-compose up -d`
2. Create migrations: Follow database schema
3. Build APIs: Follow API design
4. Implement services: Payment, email, documents
5. Create frontend: Dashboard, forms, pages
6. Add jobs: Email queue, PDF generation
7. Test thoroughly: Unit, integration, E2E
8. Deploy: Follow MVP roadmap

---

## 🔑 Key Features

### Donor Portal
✅ Register with email verification  
✅ Manage profile and preferences  
✅ Make one-time donations  
✅ Setup recurring donations  
✅ View donation history  
✅ Download receipts & statements  
✅ Receive email notifications  
✅ Verify impact with certificates  

### Admin CMS
✅ Manage donors (view, search, merge)  
✅ Manage campaigns  
✅ Manage donations (create, verify, refund)  
✅ Manage recurring plans  
✅ View reports and analytics  
✅ Manage email templates  
✅ View email logs  
✅ View audit trail  

### Technical Features
✅ Payment gateway integration  
✅ Recurring billing engine  
✅ PDF document generation  
✅ Email queue system  
✅ Role-based access control  
✅ Audit logging  
✅ Webhook processing  
✅ Error handling & retries  

---

## 📊 Technology Decisions

### Why Laravel + Vue.js?
- **Laravel:** Mature framework, rich ecosystem, excellent ORM, built-in queue/scheduler
- **Vue.js:** Reactive frontend, great DX, SPA support, good for admin panels

### Why PostgreSQL?
- ACID compliance for financial data
- JSON support for flexible data
- Native UUID support
- pgcrypto for encryption
- Full-text search capabilities

### Why Redis?
- Fast caching layer
- Queue backend
- Session storage
- Pub/sub for real-time features

### Why Docker?
- Consistent dev/prod environments
- Easy scaling
- CI/CD friendly
- Isolated services

---

## 💡 Best Practices

### Code Organization
- Repository pattern for database access
- Service layer for business logic
- Event-listener pattern for side effects
- Job classes for async tasks
- Middleware for cross-cutting concerns

### Database
- Migrations for version control
- Seeders for test data
- Soft deletes for data recovery
- Audit logging for compliance
- Indexes on foreign keys and frequently queried columns

### API Design
- RESTful principles
- Consistent response format
- Proper HTTP status codes
- Request validation
- Rate limiting

### Security
- Sanitize all user input
- Encrypt sensitive data
- Use HTTPS only
- Validate webhooks
- Implement CORS properly

---

## 🐛 Common Pitfalls

1. **Scope Creep** → Use strict feature freeze, change control process
2. **Payment Errors** → Implement idempotency keys, test thoroughly
3. **Email Delivery** → Use reputable providers, setup SPF/DKIM
4. **Performance** → Index columns early, use Redis caching
5. **Concurrency** → Use database locks for critical operations
6. **Data Loss** → Daily backups, test restore procedures

---

## 📞 Support & Resources

### External Documentation
- [Laravel 10 Docs](https://laravel.com/docs/10.x)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Stripe API](https://stripe.com/docs/api)
- [Razorpay API](https://razorpay.com/docs/)
- [Docker Docs](https://docs.docker.com/)
- [Vue.js 3 Docs](https://vuejs.org/)

### Community
- Laravel Discord
- Stack Overflow (tag: laravel)
- GitHub Discussions
- NGO tech forums

---

## 📝 License & Attribution

This comprehensive design is provided as-is for the NGO donation management system.

---

## ✅ Checklist for Implementation

- [ ] Read all documentation
- [ ] Setup Docker environment
- [ ] Create database migrations
- [ ] Implement authentication
- [ ] Build core APIs
- [ ] Setup payment gateway
- [ ] Implement email queue
- [ ] Add PDF generation
- [ ] Create frontend (Vue.js)
- [ ] Setup recurring billing
- [ ] Implement admin panel
- [ ] Add comprehensive testing
- [ ] Security audit
- [ ] Performance testing
- [ ] Deploy to staging
- [ ] User acceptance testing
- [ ] Deploy to production
- [ ] Monitor and maintain

---

**Last Updated:** January 22, 2026  
**Version:** 1.0 (MVP-Ready)  
**Status:** Ready for Implementation 🚀
