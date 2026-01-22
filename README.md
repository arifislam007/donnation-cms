# Donation Management CMS - Complete Setup Guide

## Project Overview

A complete, production-ready web-based donation management system for NGOs built with PHP-Laravel, PostgreSQL, and containerized with Docker. Supports both donor self-service portal and comprehensive admin CMS.

**Key Features:**
- Donor registration and profile management
- One-time and recurring donations
- Automated receipt and certificate generation
- Email notifications
- Admin dashboard with reporting
- Payment gateway integration
- Recurring billing engine
- Full audit trail

**Technology Stack:**
- Backend: PHP 8.2 + Laravel 10
- Frontend: Vue.js 3 + Vite
- Database: PostgreSQL 15
- Cache/Queue: Redis 7
- Web Server: Nginx
- Containerization: Docker + Docker Compose
- Payment: Stripe/Razorpay (pluggable)

---

## Quick Start (Development)

### Prerequisites
- Docker & Docker Compose installed
- Git installed
- Node.js 16+ (for frontend)
- Text editor (VS Code recommended)

### Setup in 5 Steps

#### 1. Clone Repository
```bash
cd f:/Personal-Project
git clone https://github.com/your-org/donation-cms.git
cd donation-cms
```

#### 2. Create Environment File
```bash
cp .env.example .env

# Edit .env with your settings:
APP_NAME="Donation CMS"
APP_ENV=local
APP_KEY= # Will be generated
APP_DEBUG=true
APP_URL=http://localhost

DB_DATABASE=donation_cms
DB_USERNAME=postgres
DB_PASSWORD=secret

REDIS_HOST=redis
REDIS_PORT=6379

MAIL_DRIVER=log
# or use MailHog:
MAIL_HOST=mailhog
MAIL_PORT=1025

# Payment Gateway (get from Stripe/Razorpay)
PAYMENT_GATEWAY=stripe
STRIPE_PUBLIC_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

#### 3. Start Docker Containers
```bash
docker-compose up -d

# Watch logs
docker-compose logs -f app

# Check all services are running
docker-compose ps
```

Output should show:
```
donation-cms-db         running
donation-cms-redis      running
donation-cms-app        running
donation-cms-nginx      running
donation-cms-queue      running
donation-cms-scheduler  running
donation-cms-mailhog    running
```

#### 4. Run Database Migrations
```bash
docker-compose exec app php artisan migrate
docker-compose exec app php artisan seed:run  # (optional) seed test data
```

#### 5. Access Application
- **Frontend:** http://localhost
- **API:** http://localhost/api/v1
- **MailHog:** http://localhost:8025 (for email testing)
- **Admin:** Login with default admin account

---

## Frontend Setup

### Setup Vue.js Frontend (Optional - Already included in docs)

```bash
# In a separate directory
npm install -g create-vite
npm create vite@latest donation-cms-frontend -- --template vue
cd donation-cms-frontend
npm install

# Copy .env.example to .env
cp .env.example .env

# Update API URL in .env
VITE_API_URL=http://localhost/api/v1

# Start development server
npm run dev
```

Runs on http://localhost:5173

---

## Project Structure

```
donation-cms/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── AuthController.php
│   │   │   ├── DonorController.php
│   │   │   ├── DonationController.php
│   │   │   ├── CampaignController.php
│   │   │   ├── RecurringPlanController.php
│   │   │   └── AdminController.php
│   │   ├── Middleware/
│   │   │   ├── CheckRole.php
│   │   │   ├── CheckPermission.php
│   │   │   └── RateLimiter.php
│   │   └── Requests/
│   ├── Models/
│   │   ├── User.php
│   │   ├── DonorProfile.php
│   │   ├── Donation.php
│   │   ├── RecurringPlan.php
│   │   ├── Campaign.php
│   │   ├── PaymentTransaction.php
│   │   ├── Document.php
│   │   ├── EmailLog.php
│   │   ├── AuditLog.php
│   │   └── Template.php
│   ├── Services/
│   │   ├── PaymentService.php
│   │   ├── DonationService.php
│   │   ├── RecurringBillingService.php
│   │   ├── DocumentService.php
│   │   ├── EmailService.php
│   │   └── AuditService.php
│   ├── Jobs/
│   │   ├── SendDonationReceiptEmail.php
│   │   ├── SendThankYouLetterEmail.php
│   │   ├── GenerateReceiptPDF.php
│   │   ├── GenerateDonationStatementPDF.php
│   │   ├── ProcessRecurringDonations.php
│   │   └── ... (other jobs)
│   ├── Events/
│   │   ├── DonationCreated.php
│   │   ├── DonationConfirmed.php
│   │   ├── PaymentFailed.php
│   │   └── RecurringPlanCreated.php
│   ├── Listeners/
│   │   ├── SendReceiptEmail.php
│   │   ├── LogDonation.php
│   │   └── UpdateDonorStats.php
│   ├── Repositories/
│   │   ├── DonationRepository.php
│   │   ├── DonorRepository.php
│   │   ├── CampaignRepository.php
│   │   └── ...
│   └── Rules/
│       └── (custom validation rules)
├── database/
│   ├── migrations/
│   │   ├── 2024_01_15_000001_create_users_table.php
│   │   ├── 2024_01_15_000002_create_donor_profiles_table.php
│   │   ├── 2024_01_15_000003_create_campaigns_table.php
│   │   ├── 2024_01_15_000004_create_donations_table.php
│   │   ├── 2024_01_15_000005_create_recurring_plans_table.php
│   │   ├── 2024_01_15_000006_create_payment_transactions_table.php
│   │   ├── 2024_01_15_000007_create_documents_table.php
│   │   ├── 2024_01_15_000008_create_email_logs_table.php
│   │   ├── 2024_01_15_000009_create_audit_logs_table.php
│   │   ├── 2024_01_15_000010_create_templates_table.php
│   │   └── ...
│   └── seeders/
│       ├── UserSeeder.php
│       ├── CampaignSeeder.php
│       ├── DonorProfileSeeder.php
│       └── TemplateSeeder.php
├── resources/
│   ├── views/
│   │   ├── emails/
│   │   │   ├── receipt.blade.php
│   │   │   ├── thank-you.blade.php
│   │   │   └── ...
│   │   ├── pdfs/
│   │   │   ├── receipt.blade.php
│   │   │   ├── statement.blade.php
│   │   │   └── certificate.blade.php
│   │   └── ...
│   └── js/
│       ├── app.js
│       ├── components/
│       ├── pages/
│       └── stores/
├── routes/
│   ├── api.php (v1)
│   ├── web.php
│   └── channels.php (for real-time - optional)
├── config/
│   ├── donation.php (custom config)
│   ├── payment.php (payment gateway config)
│   ├── email.php
│   └── queue.php
├── storage/
│   ├── app/
│   │   ├── pdf/
│   │   │   ├── receipts/
│   │   │   ├── statements/
│   │   │   └── certificates/
│   │   └── uploads/
│   └── logs/
├── tests/
│   ├── Feature/
│   │   ├── AuthTest.php
│   │   ├── DonationTest.php
│   │   ├── PaymentTest.php
│   │   └── ...
│   ├── Unit/
│   │   ├── Services/
│   │   ├── Models/
│   │   └── ...
│   └── Pest.php
├── docker/
│   ├── Dockerfile
│   ├── nginx/
│   │   ├── nginx.conf
│   │   └── default.conf
│   ├── php/
│   │   └── php.ini
│   └── postgres/
│       └── init.sql
├── docs/
│   ├── 01-ARCHITECTURE.md
│   ├── 02-DATABASE-SCHEMA.md
│   ├── 03-API-DESIGN.md
│   ├── 04-USER-JOURNEYS.md
│   ├── 05-BACKGROUND-JOBS.md
│   ├── 06-SECURITY-COMPLIANCE.md
│   └── 07-MVP-ROADMAP.md
├── docker-compose.yml
├── .env.example
├── .gitignore
├── composer.json
├── package.json
└── README.md
```

---

## Common Commands

### Docker
```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f app

# Restart a service
docker-compose restart app

# Execute command in container
docker-compose exec app bash
```

### Laravel Artisan
```bash
# Run migrations
docker-compose exec app php artisan migrate

# Seed database
docker-compose exec app php artisan seed:run

# Clear cache
docker-compose exec app php artisan cache:clear

# Clear queued jobs
docker-compose exec app php artisan queue:flush

# View failed jobs
docker-compose exec app php artisan queue:failed

# Retry failed job
docker-compose exec app php artisan queue:retry {id}

# Create model with migration
docker-compose exec app php artisan make:model ModelName -m

# Create controller
docker-compose exec app php artisan make:controller ControllerName

# Create job
docker-compose exec app php artisan make:job JobName

# Create seeder
docker-compose exec app php artisan make:seeder SeederName

# Run tests
docker-compose exec app php artisan test

# Run specific test file
docker-compose exec app php artisan test tests/Feature/DonationTest.php
```

### Database
```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U postgres -d donation_cms

# Backup database
docker-compose exec postgres pg_dump -U postgres donation_cms > backup.sql

# Restore database
docker-compose exec postgres psql -U postgres donation_cms < backup.sql
```

### Queue
```bash
# View queue status
docker-compose exec app php artisan queue:monitor

# Restart queue workers
docker-compose exec app php artisan queue:restart

# Process one job then exit
docker-compose exec queue php artisan queue:work --once
```

---

## Configuration Guide

### Payment Gateway Setup

#### Stripe Integration
```env
PAYMENT_GATEWAY=stripe
STRIPE_PUBLIC_KEY=pk_test_xxxxx
STRIPE_SECRET_KEY=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
STRIPE_WEBHOOK_URL=https://yourdomain.com/webhooks/stripe
```

#### Razorpay Integration
```env
PAYMENT_GATEWAY=razorpay
RAZORPAY_KEY_ID=rzp_test_xxxxx
RAZORPAY_KEY_SECRET=xxxxx
RAZORPAY_WEBHOOK_SECRET=xxxxx
RAZORPAY_WEBHOOK_URL=https://yourdomain.com/webhooks/razorpay
```

### Email Configuration

#### Using SMTP
```env
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=465
MAIL_USERNAME=your_username
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@donationcms.local
```

#### Using SendGrid
```env
MAIL_DRIVER=sendgrid
SENDGRID_API_KEY=SG.xxxxx
MAIL_FROM_ADDRESS=noreply@donationcms.local
```

#### Using AWS SES
```env
MAIL_DRIVER=ses
AWS_ACCESS_KEY_ID=xxxxx
AWS_SECRET_ACCESS_KEY=xxxxx
AWS_DEFAULT_REGION=ap-south-1
MAIL_FROM_ADDRESS=noreply@donationcms.local
```

---

## Production Deployment

### AWS EC2 Deployment (Example)

#### 1. Provision Server
```bash
# Create EC2 instance
- Ubuntu 22.04 LTS
- t3.medium (2GB RAM, 2 vCPU)
- 50GB EBS volume
- Security group: Allow 80, 443, 22

# Create RDS PostgreSQL instance
- PostgreSQL 15
- db.t3.small
- 50GB storage
- Multi-AZ backup

# Create ElastiCache Redis
- Redis 7
- cache.t3.micro
```

#### 2. Setup Server
```bash
# SSH into server
ssh -i key.pem ubuntu@instance-ip

# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker & Docker Compose
sudo apt install -y docker.io docker-compose
sudo usermod -aG docker ubuntu

# Clone repository
git clone https://github.com/your-org/donation-cms.git
cd donation-cms

# Copy production env
cp .env.example .env
# Edit .env with production values

# Generate app key
docker-compose exec app php artisan key:generate

# Run migrations
docker-compose exec app php artisan migrate --force
```

#### 3. SSL Certificate
```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Generate certificate
sudo certbot certonly --standalone -d yourdomain.com

# Auto-renew
sudo systemctl enable certbot.timer
```

#### 4. Monitoring & Logging
```bash
# Install CloudWatch agent (if using AWS)
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb

# Setup log rotation
sudo vi /etc/logrotate.d/donation-cms
```

#### 5. Backups
```bash
# Database backup cron
0 2 * * * docker-compose exec postgres pg_dump -U postgres donation_cms | gzip > /backups/db-$(date +\%Y\%m\%d).sql.gz

# S3 sync
0 3 * * * aws s3 sync /backups s3://my-backup-bucket/donation-cms/
```

---

## Testing

### Setup Testing Environment
```bash
# Create test database
docker-compose exec postgres createdb -U postgres donation_cms_test

# Run tests
docker-compose exec app php artisan test

# Run with coverage
docker-compose exec app php artisan test --coverage

# Run specific test
docker-compose exec app php artisan test tests/Feature/DonationTest.php
```

### Test Files Structure
```
tests/
├── Feature/
│   ├── AuthTest.php
│   ├── DonationTest.php
│   ├── RecurringPlanTest.php
│   ├── PaymentTest.php
│   ├── AdminTest.php
│   └── ReportTest.php
├── Unit/
│   ├── Services/
│   │   ├── PaymentServiceTest.php
│   │   ├── DonationServiceTest.php
│   │   └── RecurringBillingServiceTest.php
│   └── Models/
│       ├── DonationTest.php
│       ├── RecurringPlanTest.php
│       └── UserTest.php
└── Pest.php
```

---

## Troubleshooting

### Common Issues

#### 1. Container won't start
```bash
# Check logs
docker-compose logs app

# Rebuild container
docker-compose up --build

# Check if port is already in use
sudo netstat -tlnp | grep :8000
```

#### 2. Database connection failed
```bash
# Check PostgreSQL status
docker-compose logs postgres

# Test connection
docker-compose exec app php artisan tinker
>>> DB::connection()->getDatabaseName()

# Verify credentials in .env
```

#### 3. Queue not processing jobs
```bash
# Check queue worker logs
docker-compose logs queue

# Check failed jobs
docker-compose exec app php artisan queue:failed

# Restart queue
docker-compose restart queue
```

#### 4. Payment webhook not received
```bash
# Check webhook logs
docker-compose exec app tail -f storage/logs/webhook.log

# Verify webhook secret in .env
# Test webhook manually in Stripe/Razorpay dashboard
```

#### 5. Email not sending
```bash
# Check mail logs
docker-compose logs mailhog

# Test SMTP connection
docker-compose exec app php artisan tinker
>>> Mail::raw('test', function ($message) { $message->to('test@example.com'); });

# Check MailHog UI: http://localhost:8025
```

---

## Security Checklist

- [ ] Change default database password in .env
- [ ] Generate new APP_KEY: `php artisan key:generate`
- [ ] Set APP_DEBUG=false in production
- [ ] Setup HTTPS with SSL certificate
- [ ] Configure CORS properly for frontend URL
- [ ] Setup rate limiting
- [ ] Enable CSRF protection
- [ ] Setup firewall rules (security groups)
- [ ] Enable database backups
- [ ] Setup monitoring and alerting
- [ ] Rotate API keys periodically
- [ ] Review audit logs regularly
- [ ] Enable audit logging for admin actions
- [ ] Setup VPN for admin access (optional)
- [ ] Enable two-factor authentication (future)

---

## Support & Documentation

### Key Documentation Files
- [Architecture Design](docs/01-ARCHITECTURE.md)
- [Database Schema](docs/02-DATABASE-SCHEMA.md)
- [API Design & Endpoints](docs/03-API-DESIGN.md)
- [User Journeys & Screens](docs/04-USER-JOURNEYS.md)
- [Background Jobs & Queues](docs/05-BACKGROUND-JOBS.md)
- [Security & Compliance](docs/06-SECURITY-COMPLIANCE.md)
- [MVP Roadmap](docs/07-MVP-ROADMAP.md)

### Getting Help
- Check documentation first
- Review relevant code files
- Search GitHub issues
- Contact development team

---

## License
[Your License Here]

## Contributors
[List contributors]

---

## Next Steps

1. Review architecture: [01-ARCHITECTURE.md](docs/01-ARCHITECTURE.md)
2. Setup development environment: `docker-compose up -d`
3. Review database schema: [02-DATABASE-SCHEMA.md](docs/02-DATABASE-SCHEMA.md)
4. Explore API endpoints: [03-API-DESIGN.md](docs/03-API-DESIGN.md)
5. Start building: Follow MVP roadmap in [07-MVP-ROADMAP.md](docs/07-MVP-ROADMAP.md)

Happy coding! 🚀
