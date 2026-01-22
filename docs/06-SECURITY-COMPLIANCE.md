# Donation Management CMS - Security & Compliance

## Role-Based Access Control (RBAC)

### User Roles & Permissions

#### 1. DONOR
**Capabilities:**
- Register/login
- View own profile
- Edit own profile
- Make donations (one-time & recurring)
- View own donation history
- Download own receipts, statements, certificates
- Manage email preferences
- Cancel own recurring plans
- Access own documents

**Restrictions:**
- Cannot see other donors' data
- Cannot access admin panel
- Cannot manage campaigns
- Cannot view reports

**Database:**
```sql
INSERT INTO roles (name, display_name) VALUES ('DONOR', 'Donor');
INSERT INTO permissions (name, display_name) VALUES 
    ('donate.create', 'Create donation'),
    ('donate.view.own', 'View own donations'),
    ('profile.edit.own', 'Edit own profile'),
    ('document.download.own', 'Download own documents'),
    ('preferences.edit', 'Edit preferences');

-- Assign permissions to role
INSERT INTO role_permissions (role_id, permission_id) VALUES (1, 1), (1, 2), (1, 3), (1, 4), (1, 5);
```

#### 2. ADMIN
**Capabilities:**
- All DONOR capabilities
- View all donors (search, filter, export)
- View all donations
- Create manual donations (for offline/cheque)
- Manage campaigns (create, edit, publish, pause)
- Manage recurring plans (retry, force charge, cancel)
- Manage templates (create, edit, deactivate)
- View all email logs
- View audit logs
- View reports (all types)
- Generate reports & export
- Merge donor profiles
- Suspend/reactivate donor accounts

**Restrictions:**
- Cannot modify system settings
- Cannot manage other admins
- Cannot delete permanent records (soft delete only)

**Middleware Protection:**
```php
Route::middleware('auth:sanctum', 'role:ADMIN')->group(function () {
    Route::get('/admin/donors', [DonorController::class, 'index']);
    Route::post('/admin/donations/manual-entry', [DonationController::class, 'manualEntry']);
    Route::post('/admin/campaigns', [CampaignController::class, 'store']);
    // ... more admin routes
});
```

#### 3. FINANCE_OFFICER
**Capabilities:**
- View all donations
- View recurring plans
- Process refunds
- Verify payments (cheques, transfers)
- View payment reports
- Export financial reports
- View donor tax information
- Track payment reconciliation

**Restrictions:**
- Cannot create/edit campaigns
- Cannot manage templates
- Cannot suspend donors
- Cannot access user personal data beyond financial context

#### 4. CONTENT_MANAGER
**Capabilities:**
- Manage campaigns (create, edit, publish)
- Manage templates
- Create/edit impact updates
- View donation stats for campaigns
- Manage email templates (content only, not sending)

**Restrictions:**
- Cannot view donor personal data (except aggregate stats)
- Cannot refund donations
- Cannot access payment data

### Permission Matrix

| Action | DONOR | ADMIN | FINANCE | CONTENT |
|--------|-------|-------|---------|---------|
| Register/Login | ✓ | ✓ | ✓ | ✓ |
| View own profile | ✓ | ✓ | ✓ | ✓ |
| View all donors | ✗ | ✓ | ✗ | ✗ |
| Make donation | ✓ | ✓ | ✗ | ✗ |
| Refund donation | ✗ | ✓ | ✓ | ✗ |
| Create campaign | ✗ | ✓ | ✗ | ✓ |
| View reports | ✗ | ✓ | ✓ | ✗ |
| Manage templates | ✗ | ✓ | ✗ | ✓ |
| Manage recurring plans | ✗ | ✓ | ✓ | ✗ |

---

## Authentication

### JWT Token Strategy

```php
// Login response
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "token_type": "Bearer"
}

// Token Claims
{
    "sub": "uuid-user-id",
    "email": "user@example.com",
    "role": "DONOR",
    "iat": 1642852800,
    "exp": 1642856400  // expires in 1 hour
}
```

### Token Storage

- **Frontend:** Store JWT in memory (never localStorage)
- **Backend:** Store refresh token in Redis with TTL
- **Refresh:** Use refresh token to get new access token

### Token Revocation

```php
// On logout
public function logout(Request $request)
{
    // Revoke refresh token
    Cache::tags('refresh_tokens')
        ->forget(auth()->user()->id);
    
    // Invalidate current token (optional)
    return response()->json(['message' => 'Logged out']);
}

// Check if token is revoked
public function isTokenRevoked($tokenId)
{
    return !Cache::tags('refresh_tokens')
        ->has($tokenId);
}
```

---

## Data Encryption

### At-Rest Encryption

**PostgreSQL pgcrypto extension:**

```sql
-- Create encrypted column
ALTER TABLE donor_profiles 
ADD COLUMN tax_id_encrypted BYTEA;

-- Encrypt function
CREATE OR REPLACE FUNCTION encrypt_sensitive(data text, key text)
RETURNS BYTEA AS $$
SELECT pgp_pub_encrypt(data, dearmor(key))
$$ LANGUAGE SQL;

-- Decrypt function
CREATE OR REPLACE FUNCTION decrypt_sensitive(data BYTEA, key text)
RETURNS text AS $$
SELECT pgp_pub_decrypt(data, dearmor(key))
$$ LANGUAGE SQL;

-- Usage
INSERT INTO donor_profiles (tax_id_encrypted) 
VALUES (encrypt_sensitive('12345678', public_key));

SELECT decrypt_sensitive(tax_id_encrypted, private_key) 
FROM donor_profiles WHERE id = $1;
```

### Sensitive Fields to Encrypt

```php
// In model
protected $encrypted = [
    'tax_id',
    'pan_number',
    'bank_account_number',
    'bank_ifsc_code'
];

// Using Laravel Crypt
use Illuminate\Support\Facades\Crypt;

class DonorProfile extends Model
{
    protected $casts = [
        'tax_id' => 'encrypted',
        'pan_number' => 'encrypted',
        'bank_account_number' => 'encrypted',
    ];

    // Automatic encryption/decryption on save/retrieve
    public function getTaxIdAttribute($value)
    {
        return $value ? Crypt::decryptString($value) : null;
    }

    public function setTaxIdAttribute($value)
    {
        $this->attributes['tax_id'] = $value ? Crypt::encryptString($value) : null;
    }
}
```

### Sensitive Data Masking

```php
// In API response
public function toArray()
{
    return [
        'id' => $this->id,
        'full_name' => $this->full_name,
        'tax_id' => $this->maskTaxId($this->tax_id),  // Show: ****5678
        'bank_account' => $this->maskBankAccount(),    // Show: ****7890
        'email' => $this->email  // Already safe
    ];
}

private function maskTaxId($taxId)
{
    return strlen($taxId) > 4 
        ? '****' . substr($taxId, -4) 
        : '****';
}

private function maskBankAccount()
{
    return $this->bank_account_number 
        ? '****' . substr($this->bank_account_number, -4)
        : null;
}
```

---

## Transport Security

### HTTPS/TLS

**Requirements:**
- TLS 1.3 minimum
- Force redirect HTTP → HTTPS
- HSTS header

**Nginx Configuration:**
```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name _;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # HSTS header
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'" always;

    # ... rest of config
}
```

---

## Rate Limiting & Bot Protection

### Login Rate Limiting

```php
// In LoginController
public function login(Request $request)
{
    // Check rate limit: 5 attempts per 15 minutes
    $this->throttleLogin($request);

    // Attempt login...
}

private function throttleLogin(Request $request)
{
    $throttleKey = 'login-attempts:' . $request->ip();
    $maxAttempts = 5;
    $decayMinutes = 15;

    if (RateLimiter::tooManyAttempts($throttleKey, $maxAttempts)) {
        $seconds = RateLimiter::availableIn($throttleKey);
        throw ValidationException::withMessages([
            'email' => "Too many login attempts. Try again in {$seconds} seconds."
        ]);
    }

    RateLimiter::hit($throttleKey, $decayMinutes * 60);
}
```

### API Rate Limiting

```php
// In routes/api.php
Route::middleware('throttle:api')->group(function () {
    Route::post('/donations', [DonationController::class, 'store']);
    Route::get('/campaigns', [CampaignController::class, 'index']);
});

// In config/app.php or middleware
// Donation endpoints: 10 per minute
// Payment endpoints: 5 per minute (strict)
// Admin endpoints: 100 per minute
// Public endpoints: 60 per minute

public function throttle()
{
    return 'api:' . auth()->user()?->id ?? $request->ip();
}
```

### CAPTCHA for Signup

```php
// In RegistrationRequest
public function rules()
{
    return [
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8|confirmed',
        'g_recaptcha_response' => 'required|recaptcha'
    ];
}

// In config/services.php
'recaptcha' => [
    'secret' => env('RECAPTCHA_SECRET_KEY'),
    'key' => env('RECAPTCHA_PUBLIC_KEY'),
],
```

---

## CSRF & CORS Protection

### CSRF Tokens

```php
// In forms
<input type="hidden" name="_token" value="{{ csrf_token() }}">

// Middleware verifies all POST/PUT/DELETE requests
protected $middleware = [
    VerifyCsrfToken::class
];

// Exclude payment webhooks from CSRF
protected $except = [
    'webhooks/payment-gateway/*'
];
```

### CORS Configuration

```php
// config/cors.php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_methods' => ['*'],
'allowed_origins' => explode(',', env('CORS_ALLOWED_ORIGINS', 'http://localhost:3000')),
'allowed_origins_patterns' => [],
'allowed_headers' => ['*'],
'exposed_headers' => [],
'max_age' => 0,
'supports_credentials' => true,
```

---

## Webhook Security

### Signature Verification

```php
// Webhook handler
public function handlePaymentWebhook(Request $request)
{
    $payload = $request->getContent();
    $signature = $request->header('X-Webhook-Signature');

    // Verify signature
    $expectedSignature = hash_hmac(
        'sha256',
        $payload,
        config('payment.webhook_secret')
    );

    if (!hash_equals($signature, $expectedSignature)) {
        Log::warning('Invalid webhook signature', ['ip' => $request->ip()]);
        abort(401, 'Unauthorized');
    }

    // Process webhook...
    // Create donation record
    // Trigger events
    // Send confirmations
}
```

### Webhook IP Whitelisting

```php
// Middleware
public function handle($request, Closure $next)
{
    $allowedIps = config('payment.webhook_ips');

    if (!in_array($request->ip(), $allowedIps)) {
        Log::warning("Webhook from unauthorized IP: {$request->ip()}");
        abort(403, 'Forbidden');
    }

    return $next($request);
}
```

---

## Audit Logging

### Audit Trail

```php
// Create audit log on sensitive operations
public function storeDonation(StoreDonationRequest $request)
{
    $donation = Donation::create($request->validated());

    AuditLog::create([
        'user_id' => auth()->id(),
        'action' => 'CREATE',
        'entity_type' => 'DONATION',
        'entity_id' => $donation->id,
        'description' => "Created donation {$donation->donation_number}",
        'new_values' => $donation->toArray(),
        'ip_address' => request()->ip(),
        'user_agent' => request()->userAgent()
    ]);

    return response()->json($donation);
}
```

### Audit Events to Log

- User registration
- User login/logout
- Password changes
- Donation creation/refund
- Recurring plan creation/cancellation
- Admin profile modifications
- Campaign creation/publication
- Template changes
- Document generation
- Email sends
- Payment failures
- Suspensions/reactivations

---

## Data Retention & Privacy

### Data Retention Policy

```
- Active donors: Permanent (with right to delete)
- Deleted donors: 90 days soft delete, then hard delete
- Audit logs: 2 years (move to archive after 1 year)
- Email logs: 1 year
- Payment transactions: 7 years (legal requirement)
- Failed jobs: 30 days
- OTP codes: 10 minutes
- Session tokens: 30 days (refresh tokens), 1 hour (access tokens)
```

### GDPR/Privacy Compliance

```php
// Right to be forgotten
public function deleteAccount(User $user)
{
    // Create deletion record for audit
    AuditLog::create([
        'action' => 'DELETE',
        'entity_type' => 'USER',
        'entity_id' => $user->id,
        'description' => 'Account deleted per user request'
    ]);

    // Anonymize personal data
    $user->donor_profile->update([
        'full_name' => 'Deleted User',
        'phone_primary' => null,
        'address_line_1' => null,
        'tax_id' => null
    ]);

    $user->update([
        'email' => "deleted-{$user->id}@example.com",
        'password' => Hash::make(Str::random(32))
    ]);

    // Soft delete
    $user->delete();

    // Schedule hard delete after 90 days
    DeleteUserPermanently::dispatch($user)
        ->delay(now()->addDays(90));
}
```

---

## Backups & Disaster Recovery

### Backup Strategy

```bash
# Daily backup at 02:00 IST
0 2 * * * /usr/local/bin/backup-database.sh

# Weekly full backup (Sundays)
0 3 * * 0 /usr/local/bin/backup-full.sh

# Monthly backup to cold storage (1st of month)
0 4 1 * * /usr/local/bin/backup-archive.sh
```

**Backup Script:**
```bash
#!/bin/bash

DATE=$(date +%Y-%m-%d-%H%M%S)
BACKUP_DIR="/var/backups/donation-cms"
DB_NAME="donation_cms"

# PostgreSQL backup
pg_dump -h postgres -U postgres $DB_NAME | gzip > "$BACKUP_DIR/db-$DATE.sql.gz"

# Redis backup
redis-cli --rdb "$BACKUP_DIR/redis-$DATE.rdb"

# Verify backup
if [ -f "$BACKUP_DIR/db-$DATE.sql.gz" ]; then
    echo "Backup successful: $DATE"
else
    echo "Backup failed: $DATE" | mail -s "Backup Failure" admin@example.com
fi

# Keep only 30 days of daily backups
find $BACKUP_DIR -name "db-*.sql.gz" -mtime +30 -delete
```

### Disaster Recovery Plan

```
RTO (Recovery Time Objective): 4 hours
RPO (Recovery Point Objective): 1 hour

1. Identify issue
2. Provision new server
3. Restore latest backup
4. Verify data integrity
5. Update DNS/Load balancer
6. Monitor for issues
```

---

## Incident Response

### Security Incident Checklist

1. **Contain:** Isolate affected systems
2. **Identify:** Determine scope and impact
3. **Log:** Document everything
4. **Notify:** Alert stakeholders if necessary
5. **Remediate:** Fix vulnerability
6. **Verify:** Test fix
7. **Review:** Post-incident analysis

### Incident Response Team

- Security Lead
- CTO/Tech Lead
- Database Admin
- Legal/Compliance Officer
- Communications Lead

---

## Compliance Requirements

### For Bangladesh NGO

- **Tax Compliance:** Maintain records for 7 years
- **Donation Receipts:** Issue within 24 hours
- **Tax Certificate:** Yearly for donors claiming deductions
- **Annual Reports:** Available to government
- **Data Localization:** Keep donor data within Bangladesh (if required)
- **Reserve Bank Guidelines:** Follow RBI regulations on foreign donations

### International Donors (FCRA Compliance - if applicable)

- **Registration:** FCRA permit
- **Reporting:** Quarterly/annual reports to authorities
- **Donor Information:** Record foreign donor details
- **Documentation:** Maintain detailed records
- **Audit:** Regular internal/external audits

---

## Security Testing

### Penetration Testing Checklist

- SQL injection vulnerability scan
- Cross-site scripting (XSS) testing
- CSRF token validation
- Authentication bypass attempts
- Authorization bypass attempts
- Rate limiting effectiveness
- Sensitive data exposure
- Encryption validation
- API security testing

---

## Next Steps

See [MVP Roadmap](07-MVP-ROADMAP.md) for implementation timeline.
