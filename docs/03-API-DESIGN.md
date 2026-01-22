# Donation Management CMS - REST API Design

## API Overview

### Base URL
```
http://localhost:8000/api/v1
```

### Authentication
```
Authorization: Bearer {JWT_TOKEN}
Content-Type: application/json
```

### Response Format
All responses follow this structure:

```json
{
  "success": true,
  "status": 200,
  "message": "Operation successful",
  "data": { },
  "meta": {
    "timestamp": "2024-01-22T10:30:00Z",
    "request_id": "req_12345abcde",
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total": 100,
      "last_page": 5
    }
  },
  "errors": []
}
```

### Error Response
```json
{
  "success": false,
  "status": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email already exists"
    }
  ]
}
```

---

## Authentication Endpoints

### POST /auth/register
Register a new donor.

**Request:**
```json
{
  "email": "donor@example.com",
  "password": "SecurePass@123",
  "password_confirmation": "SecurePass@123",
  "phone": "+8801712345678",
  "full_name": "John Doe",
  "country": "Bangladesh"
}
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Registration successful. OTP sent to email.",
  "data": {
    "user_id": "uuid-1234",
    "email": "donor@example.com",
    "otp_expires_in": 600,
    "requires_otp_verification": true
  }
}
```

### POST /auth/verify-otp
Verify OTP for email confirmation.

**Request:**
```json
{
  "email": "donor@example.com",
  "otp_code": "123456"
}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Email verified successfully",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600,
    "user": {
      "id": "uuid-1234",
      "email": "donor@example.com",
      "role": "DONOR"
    }
  }
}
```

### POST /auth/login
Donor/Admin login with email and password.

**Request:**
```json
{
  "email": "donor@example.com",
  "password": "SecurePass@123",
  "device_name": "Chrome Browser"
}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Login successful",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600,
    "user": {
      "id": "uuid-1234",
      "email": "donor@example.com",
      "role": "DONOR",
      "full_name": "John Doe"
    }
  }
}
```

### POST /auth/refresh-token
Refresh access token using refresh token.

**Request:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Token refreshed",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600
  }
}
```

### POST /auth/logout
Logout and revoke tokens.

**Request:**
```json
{}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Logout successful"
}
```

### POST /auth/forgot-password
Request password reset token.

**Request:**
```json
{
  "email": "donor@example.com"
}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Password reset link sent to email"
}
```

### POST /auth/reset-password
Reset password with token.

**Request:**
```json
{
  "email": "donor@example.com",
  "token": "reset_token_abc123",
  "password": "NewSecurePass@123",
  "password_confirmation": "NewSecurePass@123"
}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Password reset successful"
}
```

---

## Donor Portal Endpoints

### GET /donors/profile
Get current donor profile.

**Request:**
```
GET /donors/profile
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid-1234",
    "user_id": "uuid-user",
    "full_name": "John Doe",
    "email": "donor@example.com",
    "phone_primary": "+8801712345678",
    "address_line_1": "123 Main Street",
    "city": "Dhaka",
    "postal_code": "1205",
    "country": "Bangladesh",
    "tax_id": "****5678",
    "organization_name": "John's Company",
    "organization_type": "CORPORATE",
    "gender": "MALE",
    "date_of_birth": "1990-05-15",
    "kyc_status": "VERIFIED",
    "lifetime_donation_amount": 50000.00,
    "donation_count": 12,
    "total_donation_years": 3,
    "preferred_communication": "EMAIL"
  }
}
```

### PUT /donors/profile
Update donor profile.

**Request:**
```json
{
  "full_name": "John Doe Updated",
  "phone_primary": "+8801712345678",
  "address_line_1": "456 New Avenue",
  "city": "Dhaka",
  "postal_code": "1210",
  "organization_name": "Updated Company",
  "preferred_communication": "EMAIL"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "data": {
    "id": "uuid-1234",
    "full_name": "John Doe Updated",
    "updated_at": "2024-01-22T10:30:00Z"
  }
}
```

### GET /donors/preferences
Get email and communication preferences.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid-pref",
    "receive_donation_receipt_email": true,
    "receive_monthly_summary_email": true,
    "receive_annual_statement_email": true,
    "receive_campaign_updates": false,
    "receive_thank_you_letter": true,
    "receive_sms_notifications": false,
    "receive_donation_reminders": false,
    "marketing_communication_opt_in": false,
    "preferred_language": "EN",
    "newsletter_frequency": "MONTHLY"
  }
}
```

### PUT /donors/preferences
Update communication preferences.

**Request:**
```json
{
  "receive_donation_receipt_email": true,
  "receive_monthly_summary_email": false,
  "preferred_language": "BN"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Preferences updated"
}
```

---

## Donation Endpoints

### GET /campaigns
List all active campaigns.

**Request:**
```
GET /campaigns?page=1&per_page=20&category=EDUCATION&status=ACTIVE
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-camp-1",
      "name": "Education for Rural Children",
      "slug": "education-rural-children",
      "description": "Provide quality education to underprivileged children",
      "category": "EDUCATION",
      "target_amount": 1000000.00,
      "collected_amount": 450000.00,
      "collected_donations_count": 287,
      "image_url": "https://cdn.example.com/campaign-1.jpg",
      "status": "ACTIVE",
      "start_date": "2024-01-01",
      "end_date": "2024-12-31",
      "is_recurring_allowed": true,
      "min_donation_amount": 100,
      "max_donation_amount": 500000,
      "currency": "BDT",
      "progress_percentage": 45
    }
  ],
  "meta": {
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total": 15,
      "last_page": 1
    }
  }
}
```

### GET /campaigns/{id}
Get campaign details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid-camp-1",
    "name": "Education for Rural Children",
    "description": "Provide quality education to underprivileged children",
    "impact_statement": "We've impacted 5000+ children in 50 villages",
    "category": "EDUCATION",
    "target_amount": 1000000.00,
    "collected_amount": 450000.00,
    "collected_donations_count": 287,
    "status": "ACTIVE",
    "impact_updates": [
      {
        "date": "2024-01-15",
        "title": "January Milestone Reached",
        "description": "We've trained 50 teachers in 10 villages",
        "image_url": "https://cdn.example.com/impact-update-1.jpg"
      }
    ]
  }
}
```

### POST /donations/create-one-time
Create a one-time donation.

**Request:**
```json
{
  "campaign_id": "uuid-camp-1",
  "amount": 5000,
  "currency": "BDT",
  "payment_method": "CREDIT_CARD",
  "is_anonymous": false,
  "is_tax_deductible": true,
  "custom_message": "In memory of loved ones"
}
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Donation created. Proceed to payment.",
  "data": {
    "donation_id": "uuid-don-1",
    "donation_number": "DND-20240122-00001",
    "amount": 5000,
    "currency": "BDT",
    "status": "PENDING",
    "payment_url": "https://payment-gateway.example.com/checkout/abc123",
    "expires_in": 1800
  }
}
```

### POST /donations/create-recurring
Create a recurring (subscription) donation.

**Request:**
```json
{
  "campaign_id": "uuid-camp-1",
  "amount": 1000,
  "currency": "BDT",
  "frequency": "MONTHLY",
  "payment_method": "CREDIT_CARD",
  "start_date": "2024-02-01",
  "end_date": null,
  "total_charges_planned": null,
  "notification_days_before": 3,
  "is_tax_deductible": true
}
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Recurring plan created",
  "data": {
    "recurring_plan_id": "uuid-rec-1",
    "plan_number": "REC-20240122-00001",
    "amount": 1000,
    "frequency": "MONTHLY",
    "next_charge_date": "2024-02-01",
    "status": "ACTIVE",
    "payment_url": "https://payment-gateway.example.com/setup/xyz789"
  }
}
```

### GET /donations/history
Get donation history for current donor.

**Request:**
```
GET /donations/history?page=1&per_page=20&status=CONFIRMED&start_date=2024-01-01&end_date=2024-12-31&campaign_id=uuid-camp-1
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-don-1",
      "donation_number": "DND-20240122-00001",
      "campaign_name": "Education for Rural Children",
      "amount": 5000,
      "currency": "BDT",
      "donation_type": "ONE_TIME",
      "payment_method": "CREDIT_CARD",
      "status": "CONFIRMED",
      "donation_date": "2024-01-22",
      "fiscal_year": 2024,
      "is_tax_deductible": true,
      "receipt_url": "https://example.com/download/receipt-123",
      "receipt_generated": true
    },
    {
      "id": "uuid-don-2",
      "donation_number": "DND-20240121-00001",
      "campaign_name": "Healthcare for Poor",
      "amount": 2000,
      "currency": "BDT",
      "donation_type": "RECURRING_INSTANCE",
      "payment_method": "CREDIT_CARD",
      "status": "CONFIRMED",
      "donation_date": "2024-01-01",
      "fiscal_year": 2024,
      "is_tax_deductible": true,
      "receipt_url": "https://example.com/download/receipt-122"
    }
  ],
  "meta": {
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total": 12,
      "last_page": 1
    }
  }
}
```

### GET /donations/{id}
Get donation details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid-don-1",
    "donation_number": "DND-20240122-00001",
    "campaign": {
      "id": "uuid-camp-1",
      "name": "Education for Rural Children"
    },
    "amount": 5000,
    "currency": "BDT",
    "status": "CONFIRMED",
    "donation_date": "2024-01-22",
    "fiscal_year": 2024,
    "payment_method": "CREDIT_CARD",
    "payment_transaction": {
      "id": "uuid-txn-1",
      "transaction_number": "TXN-20240122-00001",
      "card_last_four": "1234",
      "card_brand": "VISA"
    },
    "documents": [
      {
        "id": "uuid-doc-1",
        "document_type": "RECEIPT",
        "download_url": "/api/documents/uuid-doc-1/download"
      }
    ]
  }
}
```

### GET /recurring-plans
Get list of recurring plans for current donor.

**Request:**
```
GET /recurring-plans?status=ACTIVE
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-rec-1",
      "plan_number": "REC-20240122-00001",
      "campaign_name": "Education for Rural Children",
      "amount": 1000,
      "frequency": "MONTHLY",
      "status": "ACTIVE",
      "start_date": "2024-02-01",
      "next_charge_date": "2024-02-01",
      "total_charges_completed": 0,
      "total_donations_so_far": 0,
      "auto_retry_enabled": true
    }
  ]
}
```

### PUT /recurring-plans/{id}
Update recurring plan (change amount, frequency, pause).

**Request:**
```json
{
  "amount": 1500,
  "frequency": "QUARTERLY"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Recurring plan updated"
}
```

### POST /recurring-plans/{id}/pause
Pause a recurring plan.

**Request:**
```json
{
  "pause_reason": "Temporary financial constraint"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Plan paused successfully"
}
```

### POST /recurring-plans/{id}/resume
Resume a paused recurring plan.

**Response:**
```json
{
  "success": true,
  "message": "Plan resumed successfully"
}
```

### POST /recurring-plans/{id}/cancel
Cancel a recurring plan.

**Request:**
```json
{
  "cancellation_reason": "Relocating to another country"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Plan cancelled successfully"
}
```

---

## Document Endpoints

### GET /documents/receipt/{donation_id}
Download receipt PDF for a donation.

**Response:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename="receipt-DND-20240122-00001.pdf"

[Binary PDF data]
```

### GET /documents/statement
Generate and download annual donation statement.

**Request:**
```
GET /documents/statement?fiscal_year=2024&include_campaigns=true
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename="statement-2024.pdf"

[Binary PDF data]
```

### GET /documents/statement/custom-range
Generate statement for custom date range.

**Request:**
```
GET /documents/statement/custom-range?start_date=2024-01-01&end_date=2024-03-31
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename="statement-2024-01-01-2024-03-31.pdf"

[Binary PDF data]
```

### GET /documents/certificate/{fiscal_year}
Download donation certificate for fiscal year.

**Request:**
```
GET /documents/certificate/2024
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename="certificate-2024.pdf"

[Binary PDF data]
```

### GET /documents/thank-you-letter/{donation_id}
Download thank you letter for a donation.

**Request:**
```
GET /documents/thank-you-letter/uuid-don-1
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```
Content-Type: application/pdf
Content-Disposition: attachment; filename="thank-you-letter.pdf"

[Binary PDF data]
```

### GET /documents/list
List all generated documents for current user.

**Request:**
```
GET /documents/list?type=RECEIPT&page=1&per_page=20
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-doc-1",
      "document_number": "RCP-20240122-00001",
      "document_type": "RECEIPT",
      "donation_number": "DND-20240122-00001",
      "amount": 5000,
      "generated_at": "2024-01-22T10:30:00Z",
      "download_url": "/api/documents/uuid-doc-1/download",
      "download_count": 2
    }
  ]
}
```

---

## Admin CMS Endpoints

### Donor Management

#### GET /admin/donors
List all donors with search and filters.

**Request:**
```
GET /admin/donors?page=1&per_page=20&search=john&kyc_status=VERIFIED&sort=lifetime_donation&order=desc
Authorization: Bearer {JWT_TOKEN}
```

**Authorization:** Requires ADMIN role

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-donor-1",
      "user_id": "uuid-user-1",
      "full_name": "John Doe",
      "email": "john@example.com",
      "phone": "+8801712345678",
      "city": "Dhaka",
      "kyc_status": "VERIFIED",
      "lifetime_donation_amount": 50000,
      "donation_count": 12,
      "last_donation_date": "2024-01-22",
      "created_at": "2023-06-15"
    }
  ],
  "meta": {
    "total_donors": 1234,
    "total_lifetime_donations": 5234000,
    "average_donation": 4245.23
  }
}
```

#### GET /admin/donors/{id}
Get detailed donor profile.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid-donor-1",
    "user": {
      "id": "uuid-user-1",
      "email": "john@example.com",
      "role": "DONOR",
      "status": "ACTIVE",
      "created_at": "2023-06-15"
    },
    "profile": {
      "full_name": "John Doe",
      "phone_primary": "+8801712345678",
      "address": "123 Main Street, Dhaka 1205",
      "kyc_status": "VERIFIED",
      "tax_id": "****5678",
      "organization_type": "CORPORATE"
    },
    "donation_stats": {
      "lifetime_amount": 50000,
      "donation_count": 12,
      "average_donation": 4166.67,
      "first_donation_date": "2023-06-20",
      "last_donation_date": "2024-01-22"
    },
    "recent_donations": [
      {
        "donation_number": "DND-20240122-00001",
        "campaign": "Education for Rural",
        "amount": 5000,
        "date": "2024-01-22",
        "status": "CONFIRMED"
      }
    ],
    "preferences": {
      "receive_email_updates": true,
      "preferred_language": "EN"
    }
  }
}
```

#### POST /admin/donors/{id}/suspend
Suspend a donor account.

**Request:**
```json
{
  "reason": "Fraudulent transaction detected"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Donor account suspended"
}
```

#### POST /admin/donors/{id}/reactivate
Reactivate suspended donor.

**Response:**
```json
{
  "success": true,
  "message": "Donor account reactivated"
}
```

#### POST /admin/donors/merge
Merge duplicate donor accounts.

**Request:**
```json
{
  "primary_donor_id": "uuid-donor-1",
  "secondary_donor_id": "uuid-donor-2",
  "merge_strategy": "KEEP_PRIMARY"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Donors merged successfully",
  "data": {
    "merged_donor_id": "uuid-donor-1",
    "donations_consolidated": 24,
    "total_lifetime_donation": 100000
  }
}
```

#### GET /admin/donors/export-csv
Export donors list to CSV.

**Request:**
```
GET /admin/donors/export-csv?kyc_status=VERIFIED
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```
Content-Type: text/csv
Content-Disposition: attachment; filename="donors-export-20240122.csv"

id,full_name,email,phone,city,lifetime_donation,donation_count
uuid-1,John Doe,john@example.com,+8801712345678,Dhaka,50000,12
uuid-2,Jane Smith,jane@example.com,+8801887654321,Chattagong,30000,8
```

---

### Donation Management

#### GET /admin/donations
List all donations with filters.

**Request:**
```
GET /admin/donations?page=1&per_page=20&status=CONFIRMED&start_date=2024-01-01&end_date=2024-12-31&campaign_id=uuid-camp-1
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-don-1",
      "donation_number": "DND-20240122-00001",
      "donor_name": "John Doe",
      "campaign": "Education for Rural",
      "amount": 5000,
      "status": "CONFIRMED",
      "payment_method": "CREDIT_CARD",
      "donation_date": "2024-01-22",
      "transaction_id": "TXN-20240122-00001"
    }
  ],
  "meta": {
    "total_donations": 5234,
    "total_amount": 23450000,
    "avg_donation": 4484.23
  }
}
```

#### POST /admin/donations/manual-entry
Manually create a donation (for offline/cheque donations).

**Request:**
```json
{
  "donor_id": "uuid-donor-1",
  "campaign_id": "uuid-camp-1",
  "amount": 10000,
  "currency": "BDT",
  "donation_date": "2024-01-15",
  "payment_method": "CHEQUE",
  "cheque_number": "CHQ123456",
  "bank_name": "Dhaka Bank",
  "is_tax_deductible": true,
  "notes": "Cheque cleared"
}
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Donation created successfully",
  "data": {
    "donation_id": "uuid-don-99",
    "donation_number": "DND-20240115-00002"
  }
}
```

#### POST /admin/donations/{id}/verify
Verify a donation (e.g., cheque clearance).

**Request:**
```json
{
  "verification_notes": "Cheque cleared successfully"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Donation verified"
}
```

#### POST /admin/donations/{id}/refund
Refund a donation.

**Request:**
```json
{
  "reason": "Duplicate charge",
  "refund_amount": 5000
}
```

**Response:**
```json
{
  "success": true,
  "message": "Refund initiated",
  "data": {
    "refund_id": "REF-20240122-00001",
    "status": "PENDING",
    "expected_completion": "2024-01-29"
  }
}
```

---

### Campaign Management

#### POST /admin/campaigns
Create a new campaign.

**Request:**
```json
{
  "name": "Emergency Relief Fund 2024",
  "slug": "emergency-relief-2024",
  "description": "Support for disaster victims",
  "impact_statement": "Help us reach 10000 affected families",
  "category": "EMERGENCY",
  "target_amount": 5000000,
  "start_date": "2024-02-01",
  "end_date": "2024-03-31",
  "is_recurring_allowed": true,
  "min_donation_amount": 500,
  "max_donation_amount": 1000000,
  "image_url": "https://cdn.example.com/campaign-image.jpg"
}
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Campaign created",
  "data": {
    "id": "uuid-camp-new",
    "name": "Emergency Relief Fund 2024",
    "slug": "emergency-relief-2024",
    "status": "DRAFT"
  }
}
```

#### PUT /admin/campaigns/{id}
Update campaign details.

**Response:**
```json
{
  "success": true,
  "message": "Campaign updated"
}
```

#### POST /admin/campaigns/{id}/publish
Publish a campaign (change from DRAFT to ACTIVE).

**Response:**
```json
{
  "success": true,
  "message": "Campaign published"
}
```

#### POST /admin/campaigns/{id}/pause
Pause a campaign (stop accepting donations).

**Response:**
```json
{
  "success": true,
  "message": "Campaign paused"
}
```

#### POST /admin/campaigns/{id}/complete
Mark campaign as completed.

**Response:**
```json
{
  "success": true,
  "message": "Campaign marked as completed"
}
```

---

### Payment & Recurring Management

#### GET /admin/recurring-plans
List all recurring plans.

**Request:**
```
GET /admin/recurring-plans?status=ACTIVE&page=1&per_page=20
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-rec-1",
      "plan_number": "REC-20240122-00001",
      "donor_name": "John Doe",
      "campaign": "Education for Rural",
      "amount": 1000,
      "frequency": "MONTHLY",
      "status": "ACTIVE",
      "start_date": "2024-02-01",
      "next_charge_date": "2024-02-01",
      "total_charges_completed": 0,
      "total_amount_collected": 0,
      "failure_count": 0
    }
  ],
  "meta": {
    "total_active_plans": 234,
    "total_monthly_projected": 345000,
    "failed_plans": 12
  }
}
```

#### POST /admin/recurring-plans/{id}/retry-failed
Retry a failed recurring charge.

**Response:**
```json
{
  "success": true,
  "message": "Retry scheduled for failed plan"
}
```

#### POST /admin/recurring-plans/{id}/force-charge
Manually trigger a charge (for testing/manual processing).

**Response:**
```json
{
  "success": true,
  "message": "Charge attempted",
  "data": {
    "transaction_id": "TXN-20240122-00099",
    "status": "CONFIRMED",
    "amount": 1000
  }
}
```

---

### Reports & Analytics

#### GET /admin/reports/daily-collections
Get daily collection report.

**Request:**
```
GET /admin/reports/daily-collections?start_date=2024-01-01&end_date=2024-01-31
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "report_date_range": "2024-01-01 to 2024-01-31",
    "total_days": 31,
    "total_collections": 1234567,
    "daily_breakdown": [
      {
        "date": "2024-01-01",
        "amount": 34567,
        "donation_count": 12,
        "payment_methods": {
          "CREDIT_CARD": 20000,
          "BANK_TRANSFER": 14567
        }
      }
    ]
  }
}
```

#### GET /admin/reports/campaign-performance
Campaign-wise performance report.

**Request:**
```
GET /admin/reports/campaign-performance?start_date=2024-01-01&end_date=2024-12-31
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "total_campaigns": 5,
    "total_amount_collected": 5234000,
    "campaigns": [
      {
        "id": "uuid-camp-1",
        "name": "Education for Rural",
        "target_amount": 1000000,
        "collected_amount": 450000,
        "collected_percentage": 45,
        "donation_count": 287,
        "avg_donation": 1567.94,
        "top_donor": "John Doe",
        "recurring_donations": 45
      }
    ]
  }
}
```

#### GET /admin/reports/donor-lifetime-value
Donor lifetime value analysis.

**Response:**
```json
{
  "success": true,
  "data": {
    "total_donors": 1234,
    "total_lifetime_value": 5234000,
    "avg_lifetime_value": 4245,
    "segments": [
      {
        "range": "0-1000",
        "donor_count": 567,
        "percentage": 45.95
      },
      {
        "range": "1001-5000",
        "donor_count": 345,
        "percentage": 27.95
      },
      {
        "range": "5001-10000",
        "donor_count": 156,
        "percentage": 12.64
      },
      {
        "range": "10000+",
        "donor_count": 166,
        "percentage": 13.46
      }
    ]
  }
}
```

#### GET /admin/reports/tax-year-statement
Fiscal year tax statement report.

**Request:**
```
GET /admin/reports/tax-year-statement/2024
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "fiscal_year": 2024,
    "period": "2024-07-01 to 2025-06-30",
    "total_donors": 1234,
    "tax_deductible_donations": 5100000,
    "non_tax_deductible_donations": 134000,
    "total_donations": 5234000,
    "avg_donation_amount": 4245,
    "export_url": "/admin/reports/tax-year-statement/2024/export-csv"
  }
}
```

---

### Template Management

#### GET /admin/templates
List all templates.

**Request:**
```
GET /admin/templates?type=EMAIL&email_type=RECEIPT
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-temp-1",
      "name": "Receipt Email Template",
      "template_type": "EMAIL",
      "email_type": "RECEIPT",
      "subject": "Your Donation Receipt - {{donation_number}}",
      "is_default": true,
      "version": 1,
      "is_active": true,
      "created_by": "admin@example.com",
      "updated_at": "2024-01-15"
    }
  ]
}
```

#### POST /admin/templates
Create a new template.

**Request:**
```json
{
  "name": "Custom Receipt Email",
  "template_type": "EMAIL",
  "email_type": "RECEIPT",
  "subject": "Thank you for your contribution - {{donation_number}}",
  "body": "Dear {{donor_name}},\n\nThank you for your donation of {{amount}} {{currency}}...",
  "html_content": "<html>...</html>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Template created"
}
```

#### PUT /admin/templates/{id}
Update template.

**Response:**
```json
{
  "success": true,
  "message": "Template updated"
}
```

---

### Email Logs

#### GET /admin/email-logs
View email sending logs.

**Request:**
```
GET /admin/email-logs?status=FAILED&page=1&per_page=20&start_date=2024-01-01&end_date=2024-01-31
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-email-1",
      "recipient_email": "donor@example.com",
      "recipient_name": "John Doe",
      "email_type": "RECEIPT",
      "subject": "Your Donation Receipt",
      "status": "FAILED",
      "error_message": "SMTP connection timeout",
      "sent_at": null,
      "created_at": "2024-01-22T10:30:00Z",
      "retry_count": 2,
      "next_retry_at": "2024-01-22T11:30:00Z"
    }
  ]
}
```

#### POST /admin/email-logs/{id}/resend
Manually resend a failed email.

**Response:**
```json
{
  "success": true,
  "message": "Email queued for resending"
}
```

---

### Audit Logs

#### GET /admin/audit-logs
View audit trail.

**Request:**
```
GET /admin/audit-logs?action=UPDATE&entity_type=DONATION&page=1&per_page=20&start_date=2024-01-01
Authorization: Bearer {JWT_TOKEN}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-audit-1",
      "audit_number": "AUD-20240122-00001",
      "user": {
        "id": "uuid-admin",
        "email": "admin@example.com"
      },
      "action": "UPDATE",
      "entity_type": "DONATION",
      "entity_id": "uuid-don-1",
      "description": "Updated donation status to CONFIRMED",
      "old_values": {
        "status": "PENDING"
      },
      "new_values": {
        "status": "CONFIRMED"
      },
      "ip_address": "192.168.1.100",
      "created_at": "2024-01-22T10:30:00Z"
    }
  ]
}
```

---

## Rate Limiting

All endpoints enforce rate limiting:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642854600

Default: 1000 requests per hour per user
Auth endpoints: 5 login attempts per 15 minutes
Payment endpoints: 10 per minute (idempotency protects duplicates)
```

---

## Webhooks (Payment Gateway Integration)

### Webhook: Payment Successful
```
POST /webhooks/payment-gateway/success
X-Webhook-Signature: hmac-sha256-signature
Content-Type: application/json

{
  "event": "payment.success",
  "payment_id": "pay_abc123",
  "donation_id": "uuid-don-1",
  "amount": 5000,
  "currency": "BDT",
  "timestamp": "2024-01-22T10:30:00Z"
}
```

### Webhook: Payment Failed
```
POST /webhooks/payment-gateway/failed
X-Webhook-Signature: hmac-sha256-signature
Content-Type: application/json

{
  "event": "payment.failed",
  "payment_id": "pay_xyz789",
  "donation_id": "uuid-don-1",
  "error": "Insufficient funds",
  "timestamp": "2024-01-22T10:30:00Z"
}
```

See [Background Jobs](05-BACKGROUND-JOBS.md) for payment processing flow.

---

## API Error Codes

| Code | Message | HTTP Status |
|------|---------|-------------|
| VALIDATION_ERROR | Input validation failed | 400 |
| UNAUTHORIZED | Missing or invalid token | 401 |
| FORBIDDEN | Insufficient permissions | 403 |
| NOT_FOUND | Resource not found | 404 |
| CONFLICT | Resource already exists | 409 |
| RATE_LIMIT_EXCEEDED | Too many requests | 429 |
| PAYMENT_ERROR | Payment processing failed | 402 |
| INTERNAL_ERROR | Server error | 500 |
| SERVICE_UNAVAILABLE | Service temporarily unavailable | 503 |

---

## Next Steps

See [Background Jobs](05-BACKGROUND-JOBS.md) for async task implementation.
