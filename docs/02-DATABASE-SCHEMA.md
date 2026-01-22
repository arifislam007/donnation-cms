# Donation Management CMS - Database Schema

## Schema Design Principles
- Normalized 3NF with strategic denormalization for performance
- PostgreSQL with UUID primary keys for distributed system readiness
- Row-level security policies (optional, for multi-tenant expansion)
- Audit trails for all sensitive operations
- Soft deletes for data recovery
- Fiscal year support (Bangladesh context)

---

## Entity Relationship Diagram (Text Representation)

```
users (1) ───────────────────(M) donors
  │
  ├─(M) audit_logs
  └─(M) email_logs

donors (1) ───────────────────(M) donations
     │
     ├─(M) recurring_plans
     ├─(M) donor_preferences
     ├─(1) donor_profiles
     └─(M) payment_transactions

campaigns (1) ───────────────────(M) donations
        │
        └─(M) recurring_plans

donations (1) ───────────────────(M) documents
      │
      ├─(M) payment_transactions
      ├─(M) email_logs
      └─(M) donation_receipts

recurring_plans (1) ───────────────────(M) payment_transactions
             │
             ├─(M) recurring_payment_logs
             └─(M) email_logs

templates (1) ───────────────────(M) email_logs
         │
         └─(M) documents

documents (1) ───────────────────(M) document_downloads

fiscal_year_configs (1) ───────────────────(M) donations
```

---

## Table Definitions

### 1. users
User accounts for both donors and admin staff.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    status ENUM('ACTIVE', 'INACTIVE', 'SUSPENDED') DEFAULT 'ACTIVE',
    role ENUM('DONOR', 'ADMIN', 'FINANCE_OFFICER', 'CONTENT_MANAGER') DEFAULT 'DONOR',
    last_login_at TIMESTAMP,
    email_verified_at TIMESTAMP,
    phone_verified_at TIMESTAMP,
    otp_code VARCHAR(6),
    otp_expires_at TIMESTAMP,
    two_factor_enabled BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_status ON users(status);
```

### 2. donor_profiles
Detailed donor information (extended from users).

```sql
CREATE TABLE donor_profiles (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    full_name VARCHAR(255) NOT NULL,
    gender ENUM('MALE', 'FEMALE', 'OTHER', 'PREFER_NOT_TO_SAY'),
    date_of_birth DATE,
    organization_name VARCHAR(255),
    organization_type ENUM('INDIVIDUAL', 'CORPORATE', 'NGO', 'TRUST', 'GOVERNMENT'),
    phone_primary VARCHAR(20) NOT NULL,
    phone_secondary VARCHAR(20),
    address_line_1 VARCHAR(500) NOT NULL,
    address_line_2 VARCHAR(500),
    city VARCHAR(100) NOT NULL,
    state_province VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100) DEFAULT 'Bangladesh',
    tax_id VARCHAR(50) ENCRYPTED,
    tax_id_verified BOOLEAN DEFAULT false,
    pan_number VARCHAR(12) ENCRYPTED,
    bank_account_number VARCHAR(50) ENCRYPTED,
    bank_name VARCHAR(255),
    bank_ifsc_code VARCHAR(15),
    preferred_communication ENUM('EMAIL', 'PHONE', 'SMS') DEFAULT 'EMAIL',
    lifetime_donation_amount DECIMAL(18,2) DEFAULT 0,
    donation_count INT DEFAULT 0,
    total_donation_years INT DEFAULT 0,
    referred_by UUID REFERENCES users(id),
    is_corporate BOOLEAN DEFAULT false,
    kyc_status ENUM('PENDING', 'VERIFIED', 'REJECTED', 'EXPIRED') DEFAULT 'PENDING',
    kyc_verified_at TIMESTAMP,
    kyc_document_url VARCHAR(500),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_donor_profiles_user_id ON donor_profiles(user_id);
CREATE INDEX idx_donor_profiles_kyc_status ON donor_profiles(kyc_status);
CREATE INDEX idx_donor_profiles_tax_id ON donor_profiles(tax_id);
```

### 3. donor_preferences
Email and communication preferences per donor.

```sql
CREATE TABLE donor_preferences (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
    receive_donation_receipt_email BOOLEAN DEFAULT true,
    receive_monthly_summary_email BOOLEAN DEFAULT false,
    receive_annual_statement_email BOOLEAN DEFAULT true,
    receive_campaign_updates BOOLEAN DEFAULT false,
    receive_thank_you_letter BOOLEAN DEFAULT true,
    receive_sms_notifications BOOLEAN DEFAULT false,
    receive_donation_reminders BOOLEAN DEFAULT false,
    marketing_communication_opt_in BOOLEAN DEFAULT false,
    preferred_language ENUM('EN', 'BN') DEFAULT 'EN',
    newsletter_frequency ENUM('WEEKLY', 'MONTHLY', 'QUARTERLY', 'ANNUALLY') DEFAULT 'MONTHLY',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_donor_preferences_user_id ON donor_preferences(user_id);
```

### 4. campaigns
Fundraising campaigns or funds that donors can give to.

```sql
CREATE TABLE campaigns (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL UNIQUE,
    slug VARCHAR(255) NOT NULL UNIQUE,
    description TEXT NOT NULL,
    impact_statement TEXT,
    category ENUM('EDUCATION', 'HEALTHCARE', 'EMERGENCY', 'LIVELIHOOD', 'ENVIRONMENT', 'RESEARCH', 'OTHER') DEFAULT 'OTHER',
    target_amount DECIMAL(18,2) NOT NULL,
    collected_amount DECIMAL(18,2) DEFAULT 0,
    collected_donations_count INT DEFAULT 0,
    image_url VARCHAR(500),
    banner_url VARCHAR(500),
    status ENUM('DRAFT', 'ACTIVE', 'PAUSED', 'COMPLETED', 'CANCELLED') DEFAULT 'DRAFT',
    start_date DATE NOT NULL,
    end_date DATE,
    is_recurring_allowed BOOLEAN DEFAULT true,
    min_donation_amount DECIMAL(10,2) DEFAULT 100,
    max_donation_amount DECIMAL(18,2),
    currency VARCHAR(3) DEFAULT 'BDT',
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_campaigns_status ON campaigns(status);
CREATE INDEX idx_campaigns_category ON campaigns(category);
CREATE INDEX idx_campaigns_created_by ON campaigns(created_by);
CREATE INDEX idx_campaigns_start_date ON campaigns(start_date);
```

### 5. donations
All donation records (one-time and recurring instances).

```sql
CREATE TABLE donations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    donation_number VARCHAR(50) NOT NULL UNIQUE, -- Format: DND-YYYYMMDD-00001
    user_id UUID NOT NULL REFERENCES users(id),
    campaign_id UUID NOT NULL REFERENCES campaigns(id),
    recurring_plan_id UUID REFERENCES recurring_plans(id),
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'BDT',
    donation_type ENUM('ONE_TIME', 'RECURRING_INSTANCE', 'ANONYMOUS') DEFAULT 'ONE_TIME',
    payment_method ENUM('CREDIT_CARD', 'DEBIT_CARD', 'BANK_TRANSFER', 'MOBILE_WALLET', 'UPI', 'CRYPTO', 'CHEQUE', 'CASH') DEFAULT 'CREDIT_CARD',
    status ENUM('PENDING', 'CONFIRMED', 'FAILED', 'REFUNDED', 'CANCELLED') DEFAULT 'PENDING',
    donation_date DATE NOT NULL,
    fiscal_year INT NOT NULL, -- e.g., 2024 for FY 2024-2025
    is_anonymous BOOLEAN DEFAULT false,
    is_tax_deductible BOOLEAN DEFAULT true,
    notes TEXT,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_donations_user_id ON donations(user_id);
CREATE INDEX idx_donations_campaign_id ON donations(campaign_id);
CREATE INDEX idx_donations_donation_date ON donations(donation_date);
CREATE INDEX idx_donations_status ON donations(status);
CREATE INDEX idx_donations_fiscal_year ON donations(fiscal_year);
CREATE INDEX idx_donations_recurring_plan_id ON donations(recurring_plan_id);
```

### 6. recurring_plans
Subscription plans for recurring donations.

```sql
CREATE TABLE recurring_plans (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    plan_number VARCHAR(50) NOT NULL UNIQUE, -- Format: REC-YYYYMMDD-00001
    user_id UUID NOT NULL REFERENCES users(id),
    campaign_id UUID NOT NULL REFERENCES campaigns(id),
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'BDT',
    frequency ENUM('WEEKLY', 'MONTHLY', 'QUARTERLY', 'SEMI_ANNUAL', 'ANNUAL') NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE, -- NULL for indefinite
    next_charge_date DATE NOT NULL,
    payment_method ENUM('CREDIT_CARD', 'DEBIT_CARD', 'BANK_TRANSFER', 'MOBILE_WALLET', 'UPI', 'MANDATE') DEFAULT 'CREDIT_CARD',
    status ENUM('ACTIVE', 'PAUSED', 'CANCELLED', 'FAILED', 'EXPIRED') DEFAULT 'ACTIVE',
    total_charges_planned INT, -- NULL for indefinite
    total_charges_completed INT DEFAULT 0,
    last_charge_date DATE,
    next_retry_date DATE, -- For retry after failure
    failure_count INT DEFAULT 0,
    max_failures INT DEFAULT 3,
    failure_reason VARCHAR(500),
    payment_gateway_subscription_id VARCHAR(255),
    auto_retry_enabled BOOLEAN DEFAULT true,
    notification_days_before INT DEFAULT 3, -- Notify donor N days before charge
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    cancelled_at TIMESTAMP,
    cancelled_by UUID REFERENCES users(id),
    cancellation_reason TEXT
);

CREATE INDEX idx_recurring_plans_user_id ON recurring_plans(user_id);
CREATE INDEX idx_recurring_plans_campaign_id ON recurring_plans(campaign_id);
CREATE INDEX idx_recurring_plans_status ON recurring_plans(status);
CREATE INDEX idx_recurring_plans_next_charge_date ON recurring_plans(next_charge_date);
```

### 7. payment_transactions
Transaction records for each donation payment attempt.

```sql
CREATE TABLE payment_transactions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    transaction_number VARCHAR(50) NOT NULL UNIQUE, -- Format: TXN-YYYYMMDD-00001
    donation_id UUID REFERENCES donations(id),
    recurring_plan_id UUID REFERENCES recurring_plans(id),
    user_id UUID NOT NULL REFERENCES users(id),
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'BDT',
    payment_method ENUM('CREDIT_CARD', 'DEBIT_CARD', 'BANK_TRANSFER', 'MOBILE_WALLET', 'UPI', 'CRYPTO', 'CHEQUE', 'CASH', 'MANDATE') DEFAULT 'CREDIT_CARD',
    payment_gateway VARCHAR(50), -- 'STRIPE', 'RAZORPAY', 'BKASH', 'NAGAD', 'ROCKET'
    payment_gateway_transaction_id VARCHAR(255),
    payment_gateway_reference_id VARCHAR(255),
    status ENUM('PENDING', 'AUTHORIZED', 'CAPTURED', 'FAILED', 'CANCELLED', 'REFUNDED') DEFAULT 'PENDING',
    response_code VARCHAR(10),
    response_message TEXT,
    card_last_four VARCHAR(4),
    card_brand VARCHAR(50),
    bank_name VARCHAR(255),
    bank_reference_number VARCHAR(100),
    idempotency_key UUID NOT NULL UNIQUE, -- Prevent double-charging
    retry_count INT DEFAULT 0,
    max_retries INT DEFAULT 3,
    next_retry_at TIMESTAMP,
    refund_id VARCHAR(255),
    refund_amount DECIMAL(18,2),
    refund_status ENUM('NONE', 'PENDING', 'COMPLETED', 'FAILED') DEFAULT 'NONE',
    refund_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payment_transactions_user_id ON payment_transactions(user_id);
CREATE INDEX idx_payment_transactions_donation_id ON payment_transactions(donation_id);
CREATE INDEX idx_payment_transactions_status ON payment_transactions(status);
CREATE INDEX idx_payment_transactions_payment_gateway_transaction_id ON payment_transactions(payment_gateway_transaction_id);
CREATE INDEX idx_payment_transactions_idempotency_key ON payment_transactions(idempotency_key);
CREATE UNIQUE INDEX idx_payment_transactions_recurring_plan_charge ON payment_transactions(recurring_plan_id, created_at) WHERE donation_id IS NULL;
```

### 8. documents
Generated documents (receipts, certificates, statements, letters).

```sql
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    document_number VARCHAR(50) NOT NULL UNIQUE, -- Format: RCP-YYYYMMDD-00001 / CERT-YYYYMMDD-00001
    document_type ENUM('RECEIPT', 'CERTIFICATE', 'STATEMENT', 'THANK_YOU_LETTER', 'TAX_CERTIFICATE') DEFAULT 'RECEIPT',
    user_id UUID NOT NULL REFERENCES users(id),
    donation_id UUID REFERENCES donations(id),
    recurring_plan_id UUID REFERENCES recurring_plans(id),
    campaign_id UUID REFERENCES campaigns(id),
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size INT, -- in bytes
    mime_type VARCHAR(100) DEFAULT 'application/pdf',
    generated_at TIMESTAMP NOT NULL,
    file_url VARCHAR(500),
    download_count INT DEFAULT 0,
    last_downloaded_at TIMESTAMP,
    expires_at TIMESTAMP, -- For sensitive documents
    is_archived BOOLEAN DEFAULT false,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_documents_user_id ON documents(user_id);
CREATE INDEX idx_documents_document_type ON documents(document_type);
CREATE INDEX idx_documents_donation_id ON documents(donation_id);
CREATE INDEX idx_documents_document_number ON documents(document_number);
```

### 9. email_logs
Email sent/failed tracking.

```sql
CREATE TABLE email_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    recipient_email VARCHAR(255) NOT NULL,
    recipient_name VARCHAR(255),
    subject VARCHAR(500) NOT NULL,
    email_type ENUM('RECEIPT', 'THANK_YOU', 'CONFIRMATION', 'MONTHLY_SUMMARY', 'ANNUAL_STATEMENT', 'CERTIFICATE', 'FAILURE_NOTIFICATION', 'REMINDER', 'WELCOME', 'CANCELLATION') DEFAULT 'RECEIPT',
    user_id UUID REFERENCES users(id),
    donation_id UUID REFERENCES donations(id),
    recurring_plan_id UUID REFERENCES recurring_plans(id),
    template_id UUID REFERENCES templates(id),
    status ENUM('PENDING', 'SENT', 'FAILED', 'BOUNCED', 'COMPLAINED', 'OPENED', 'CLICKED') DEFAULT 'PENDING',
    body TEXT,
    attachment_url VARCHAR(500),
    attachment_file_name VARCHAR(255),
    email_provider VARCHAR(50), -- 'SMTP', 'SES', 'SENDGRID'
    provider_message_id VARCHAR(500),
    error_message TEXT,
    retry_count INT DEFAULT 0,
    max_retries INT DEFAULT 3,
    next_retry_at TIMESTAMP,
    opened_at TIMESTAMP,
    clicked_at TIMESTAMP,
    scheduled_for TIMESTAMP, -- For scheduled emails
    sent_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_email_logs_recipient_email ON email_logs(recipient_email);
CREATE INDEX idx_email_logs_user_id ON email_logs(user_id);
CREATE INDEX idx_email_logs_status ON email_logs(status);
CREATE INDEX idx_email_logs_email_type ON email_logs(email_type);
CREATE INDEX idx_email_logs_created_at ON email_logs(created_at);
CREATE INDEX idx_email_logs_scheduled_for ON email_logs(scheduled_for);
```

### 10. templates
Email and document templates with dynamic placeholders.

```sql
CREATE TABLE templates (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL UNIQUE,
    template_type ENUM('EMAIL', 'DOCUMENT') NOT NULL,
    document_type ENUM('RECEIPT', 'CERTIFICATE', 'STATEMENT', 'THANK_YOU_LETTER', 'TAX_CERTIFICATE') REFERENCES documents(document_type),
    email_type ENUM('RECEIPT', 'THANK_YOU', 'CONFIRMATION', 'MONTHLY_SUMMARY', 'ANNUAL_STATEMENT', 'CERTIFICATE', 'FAILURE_NOTIFICATION', 'REMINDER', 'WELCOME', 'CANCELLATION'),
    subject VARCHAR(500), -- For email templates
    body TEXT NOT NULL,
    html_content TEXT, -- For email templates
    available_placeholders TEXT[], -- ARRAY of placeholders: {{donor_name}}, {{amount}}, etc.
    created_by UUID NOT NULL REFERENCES users(id),
    updated_by UUID REFERENCES users(id),
    is_default BOOLEAN DEFAULT false,
    is_active BOOLEAN DEFAULT true,
    version INT DEFAULT 1,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_templates_template_type ON templates(template_type);
CREATE INDEX idx_templates_name ON templates(name);
```

### 11. audit_logs
Comprehensive audit trail for compliance and security.

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    audit_number VARCHAR(50) NOT NULL UNIQUE, -- Format: AUD-YYYYMMDD-00001
    user_id UUID REFERENCES users(id),
    action VARCHAR(255) NOT NULL, -- 'CREATE', 'UPDATE', 'DELETE', 'APPROVE', 'REJECT'
    entity_type VARCHAR(100) NOT NULL, -- 'DONATION', 'RECURRING_PLAN', 'DONOR_PROFILE', 'CAMPAIGN', 'USER'
    entity_id UUID NOT NULL,
    description TEXT,
    old_values JSONB, -- Previous values for UPDATE
    new_values JSONB, -- New values for UPDATE
    ip_address VARCHAR(45),
    user_agent TEXT,
    status ENUM('SUCCESS', 'FAILURE') DEFAULT 'SUCCESS',
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_entity_type ON audit_logs(entity_type);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
```

### 12. donation_receipts
Cached receipt data for quick access (denormalization).

```sql
CREATE TABLE donation_receipts (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    donation_id UUID NOT NULL UNIQUE REFERENCES donations(id),
    receipt_number VARCHAR(50) NOT NULL UNIQUE,
    donor_name VARCHAR(255) NOT NULL,
    donor_email VARCHAR(255) NOT NULL,
    donor_phone VARCHAR(20),
    donor_address TEXT,
    campaign_name VARCHAR(255) NOT NULL,
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'BDT',
    payment_method VARCHAR(50),
    donation_date DATE NOT NULL,
    receipt_generated_date TIMESTAMP NOT NULL,
    organization_name VARCHAR(255),
    organization_address TEXT,
    organization_tax_id VARCHAR(50),
    qr_code_url VARCHAR(500),
    verification_url VARCHAR(500),
    is_tax_deductible BOOLEAN,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_donation_receipts_donation_id ON donation_receipts(donation_id);
CREATE INDEX idx_donation_receipts_receipt_number ON donation_receipts(receipt_number);
```

### 13. fiscal_year_configs
Configuration for fiscal year (Bangladesh context).

```sql
CREATE TABLE fiscal_year_configs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    fiscal_year INT NOT NULL UNIQUE, -- e.g., 2024 for FY 2024-2025
    start_date DATE NOT NULL, -- e.g., 2024-07-01
    end_date DATE NOT NULL, -- e.g., 2025-06-30
    status ENUM('ACTIVE', 'CLOSED', 'ARCHIVED') DEFAULT 'ACTIVE',
    description TEXT,
    created_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_fiscal_year_configs_status ON fiscal_year_configs(status);
```

### 14. recurring_payment_logs
Detailed logs of each recurring charge attempt.

```sql
CREATE TABLE recurring_payment_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    recurring_plan_id UUID NOT NULL REFERENCES recurring_plans(id),
    charge_date DATE NOT NULL,
    status ENUM('SCHEDULED', 'ATTEMPTED', 'SUCCESS', 'FAILED', 'SKIPPED') DEFAULT 'SCHEDULED',
    payment_transaction_id UUID REFERENCES payment_transactions(id),
    error_message TEXT,
    retry_attempt INT DEFAULT 1,
    next_retry_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_recurring_payment_logs_recurring_plan_id ON recurring_payment_logs(recurring_plan_id);
CREATE INDEX idx_recurring_payment_logs_charge_date ON recurring_payment_logs(charge_date);
```

### 15. document_downloads
Track document access for analytics.

```sql
CREATE TABLE document_downloads (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    document_id UUID NOT NULL REFERENCES documents(id),
    user_id UUID NOT NULL REFERENCES users(id),
    download_ip VARCHAR(45),
    download_user_agent TEXT,
    download_method ENUM('DIRECT', 'EMAIL', 'API') DEFAULT 'DIRECT',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_document_downloads_document_id ON document_downloads(document_id);
CREATE INDEX idx_document_downloads_user_id ON document_downloads(user_id);
```

---

## Indexes Strategy

### High-Frequency Queries
- User lookups by email, role
- Donations by user, campaign, date range, status
- Recurring plans by user, next charge date
- Email logs by status, scheduled for

### Composite Indexes
```sql
CREATE INDEX idx_donations_user_campaign_date ON donations(user_id, campaign_id, donation_date);
CREATE INDEX idx_recurring_plans_user_status ON recurring_plans(user_id, status);
```

---

## Encryption Strategy

### At Rest (PostgreSQL pgcrypto)
- tax_id
- pan_number
- bank_account_number
- address (sensitive data)

Example:
```sql
-- Encrypt
INSERT INTO donor_profiles (tax_id) VALUES (pgp_pub_encrypt('12345678', dearmor(public_key)));

-- Decrypt
SELECT pgp_pub_decrypt(tax_id, dearmor(private_key)) FROM donor_profiles WHERE id = $1;
```

### In Transit
- HTTPS/TLS 1.3 only
- JWT signed tokens

### Sensitive Fields (Masked)
- Card numbers (show: XXXX-XXXX-XXXX-1234)
- Bank account (show: ****7890)

---

## Partitioning Strategy (For Scale)

```sql
-- Partition donations by year
CREATE TABLE donations_2024 PARTITION OF donations
  FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE donations_2025 PARTITION OF donations
  FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

-- Partition email_logs by month
CREATE TABLE email_logs_202501 PARTITION OF email_logs
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

---

## Backup & Recovery

### Daily Backups
```bash
pg_dump donation_cms > backup_$(date +%Y%m%d).sql
```

### Point-in-Time Recovery
```bash
pg_basebackup -D /var/lib/postgresql/backup -Pv -X stream
```

---

## Performance Optimization

### Connection Pooling (PgBouncer)
```
[databases]
donation_cms = host=localhost port=5432 dbname=donation_cms user=postgres password=secret

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
```

### Query Caching (Redis)
- Cache donation summaries (5-min TTL)
- Cache campaign statistics (10-min TTL)
- Cache donor preferences (1-day TTL)

---

## Next Steps

See [API Design](03-API-DESIGN.md) for integration with these tables.
