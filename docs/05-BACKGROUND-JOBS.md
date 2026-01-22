# Donation Management CMS - Background Jobs & Queues

## Overview

### Job Queue Architecture
- **Queue Driver:** Redis (via Laravel Queue)
- **Worker Processes:** Running in Docker container (`queue` service)
- **Scheduler:** Cron jobs (via Laravel Scheduler, running in `scheduler` service)
- **Retry Strategy:** Exponential backoff with configurable max retries
- **Monitoring:** Failed jobs table and alerts

### Job Processing Flow
```
Event Triggered
    ↓
Create Job → Serialize to Redis
    ↓
Queue Worker picks up job
    ↓
Execute job logic
    ↓
Success: Remove from queue / Failure: Retry or move to failed jobs
```

---

## Email Queue Jobs

### 1. SendDonationReceiptEmail
**Trigger:** DonationConfirmed event

**Job Details:**
```php
namespace App\Jobs;

use App\Models\Donation;
use App\Models\EmailLog;
use Illuminate\Mail\Mailable;

class SendDonationReceiptEmail
{
    public $donation;
    public $donation_id;
    public $tries = 3;
    public $timeout = 60;
    public $backoff = [10, 60, 300]; // 10s, 1m, 5m

    public function __construct($donation_id)
    {
        $this->donation_id = $donation_id;
    }

    public function handle()
    {
        $donation = Donation::findOrFail($this->donation_id);
        $donor = $donation->user->donor_profile;
        
        // Generate receipt PDF
        $pdfPath = app(DocumentService::class)
            ->generateReceiptPDF($donation);
        
        // Send email
        Mail::to($donor->user->email)
            ->queue(new DonationReceiptMail(
                $donor,
                $donation,
                $pdfPath
            ));
        
        // Log email
        EmailLog::create([
            'recipient_email' => $donor->user->email,
            'email_type' => 'RECEIPT',
            'donation_id' => $donation->id,
            'status' => 'SENT',
            'sent_at' => now()
        ]);
    }

    public function failed(Exception $exception)
    {
        // Log failure
        EmailLog::create([
            'recipient_email' => Donation::find($this->donation_id)->user->email,
            'email_type' => 'RECEIPT',
            'donation_id' => $this->donation_id,
            'status' => 'FAILED',
            'error_message' => $exception->getMessage()
        ]);
    }
}
```

**Email Template:**
```
Subject: Your Donation Receipt - {{donation_number}}

Body:
Dear {{donor_name}},

Thank you for your generous donation!

Donation Details:
- Amount: {{amount}} {{currency}}
- Campaign: {{campaign_name}}
- Date: {{donation_date}}
- Receipt Number: {{receipt_number}}

Your receipt is attached and has also been stored in your dashboard.

Best regards,
{{organization_name}}
```

---

### 2. SendThankYouLetterEmail
**Trigger:** DonationConfirmed event (after receipt)

**Job Details:**
```php
class SendThankYouLetterEmail
{
    public $donation_id;
    public $tries = 2;
    public $delay = 3600; // Send after 1 hour

    public function handle()
    {
        $donation = Donation::findOrFail($this->donation_id);
        $donor = $donation->user->donor_profile;
        
        // Retrieve or generate thank you letter
        $template = Template::where('email_type', 'THANK_YOU')
            ->where('is_default', true)
            ->first();
        
        $letterContent = $this->parseTemplate($template, $donation);
        
        // Generate PDF
        $pdfPath = app(DocumentService::class)
            ->generateThankYouLetterPDF($donation);
        
        // Send email
        Mail::to($donor->user->email)
            ->queue(new ThankYouLetterMail($donor, $letterContent, $pdfPath));
    }
}
```

**Email Template:**
```
Subject: Thank You for Your Support - {{donor_name}}

Body:
Dear {{donor_name}},

On behalf of our entire team and the communities we serve, 
we extend our heartfelt gratitude for your donation.

Your contribution of {{amount}} {{currency}} to {{campaign_name}} 
will make a real difference in the lives of those we serve.

Impact:
With your donation, we can:
- Provide {{impact_description}}
- Reach {{beneficiaries_count}} people
- Continue our mission for {{duration}}

We will keep you updated on how your donation creates impact 
through our regular newsletters and impact reports.

With deep appreciation,
{{organization_name}} Team

P.S. Your tax certificate is attached for your records.
```

---

### 3. SendMonthlyDonationSummaryEmail
**Trigger:** Monthly scheduler (1st of month at 08:00)

**Job Details:**
```php
class SendMonthlyDonationSummaryEmail
{
    public $backoff = [600, 1800]; // 10m, 30m

    public function handle()
    {
        // Get donors with preference enabled
        $donors = DonorPreferences::where('receive_monthly_summary_email', true)
            ->with('user.donor_profile')
            ->get();

        foreach ($donors as $preference) {
            $donor = $preference->user;
            $lastMonth = now()->subMonth();
            
            // Get donations from last month
            $donations = Donation::where('user_id', $donor->id)
                ->whereMonth('donation_date', $lastMonth->month)
                ->whereYear('donation_date', $lastMonth->year)
                ->get();

            if ($donations->isEmpty()) {
                continue; // Skip if no donations
            }

            $totalAmount = $donations->sum('amount');
            $donationCount = $donations->count();
            
            // Send email
            Mail::to($donor->email)
                ->queue(new MonthlySummaryMail(
                    $donor,
                    $donations,
                    $totalAmount,
                    $donationCount
                ));

            EmailLog::create([
                'user_id' => $donor->id,
                'email_type' => 'MONTHLY_SUMMARY',
                'recipient_email' => $donor->email,
                'status' => 'QUEUED'
            ]);
        }
    }
}
```

**Email Template:**
```
Subject: Your Monthly Giving Summary - {{month_name}} {{year}}

Body:
Hi {{donor_name}},

Here's a summary of your giving in {{month_name}} {{year}}:

Total Donated: {{total_amount}} {{currency}}
Number of Donations: {{donation_count}}
Campaigns Supported:
{{#donations}}
- {{campaign_name}}: {{amount}} {{currency}} ({{date}})
{{/donations}}

Your cumulative giving this year: {{ytd_amount}} {{currency}}

View full details in your dashboard: [Dashboard Link]

Thank you for your consistent support!

Best,
{{organization_name}}
```

---

### 4. SendAnnualStatementEmail
**Trigger:** Fiscal year end scheduler (1st of next fiscal year)

**Job Details:**
```php
class SendAnnualStatementEmail
{
    public function handle()
    {
        $pastFiscalYear = FiscalYear::where('status', 'CLOSED')->latest()->first();
        
        $donors = DonorPreferences::where('receive_annual_statement_email', true)
            ->with('user')
            ->get();

        foreach ($donors as $preference) {
            $donor = $preference->user;
            
            // Generate statement PDF
            $statementPath = app(DocumentService::class)
                ->generateAnnualStatementPDF($donor->id, $pastFiscalYear->fiscal_year);

            // Send email
            Mail::to($donor->email)
                ->queue(new AnnualStatementMail(
                    $donor,
                    $pastFiscalYear->fiscal_year,
                    $statementPath
                ));
        }
    }
}
```

---

### 5. SendPaymentFailureNotificationEmail
**Trigger:** RecurringPaymentFailed event

**Job Details:**
```php
class SendPaymentFailureNotificationEmail
{
    public function handle(RecurringPlan $plan)
    {
        $donor = $plan->user;
        
        $failureReason = $plan->failure_reason;
        
        // Send email
        Mail::to($donor->email)
            ->queue(new PaymentFailureMail(
                $donor,
                $plan,
                $failureReason
            ));

        // If final failure, also send urgent message
        if ($plan->failure_count >= $plan->max_failures) {
            Mail::to($donor->email)
                ->queue(new SubscriptionCancelledMail($donor, $plan));
        }
    }
}
```

**Email Template:**
```
Subject: ⚠ Your Monthly Donation Failed - Action Needed

Body:
Hi {{donor_name}},

We tried to charge your recurring donation of {{amount}} {{currency}} 
to {{campaign_name}} on {{charge_date}}, but it failed.

Reason: {{failure_reason}}

What to do:
1. Update your payment method: [Link to payment settings]
2. We'll try again in 24 hours
3. If still failing, your plan will be paused after 3 attempts

If you have questions, contact us: {{support_email}}

Update Payment Method: [Button Link]
```

---

## PDF Generation Jobs

### 6. GenerateReceiptPDF
**Trigger:** DonationConfirmed event

**Job Details:**
```php
class GenerateReceiptPDF
{
    public function handle(Donation $donation)
    {
        $donor = $donation->user->donor_profile;
        
        // Get template
        $template = Template::where('document_type', 'RECEIPT')
            ->where('is_default', true)
            ->first();

        $data = [
            'donor_name' => $donor->full_name,
            'donor_email' => $donation->user->email,
            'donor_phone' => $donor->phone_primary,
            'donor_address' => "{$donor->address_line_1}, {$donor->city}",
            'amount' => $donation->amount,
            'currency' => $donation->currency,
            'donation_date' => $donation->donation_date->format('d M Y'),
            'donation_number' => $donation->donation_number,
            'receipt_number' => "RCP-{$donation->donation_number}",
            'campaign_name' => $donation->campaign->name,
            'payment_method' => $donation->payment_method,
            'org_name' => config('donation.organization_name'),
            'org_address' => config('donation.organization_address'),
            'org_tax_id' => config('donation.organization_tax_id'),
            'org_phone' => config('donation.organization_phone'),
            'org_website' => config('donation.organization_website'),
            'qr_code_url' => $this->generateQRCode($donation),
            'verification_url' => route('verify-receipt', ['token' => hash('sha256', $donation->id)])
        ];

        // Render PDF using DomPDF
        $pdf = PDF::loadView('receipts.template', $data);
        
        // Store PDF
        $fileName = "receipt-{$donation->donation_number}.pdf";
        $filePath = "storage/app/pdf/receipts/{$fileName}";
        Storage::put($filePath, $pdf->output());

        // Create document record
        Document::create([
            'document_number' => "RCP-" . $donation->donation_number,
            'document_type' => 'RECEIPT',
            'user_id' => $donation->user_id,
            'donation_id' => $donation->id,
            'file_name' => $fileName,
            'file_path' => $filePath,
            'file_size' => strlen($pdf->output()),
            'mime_type' => 'application/pdf',
            'generated_at' => now(),
            'file_url' => Storage::url($filePath)
        ]);
    }

    private function generateQRCode($donation)
    {
        // Generate QR code linking to verification
        $data = "DND:{$donation->donation_number}";
        $qr = QrCode::format('png')->size(200)->generate($data);
        $path = "storage/app/qrcodes/{$donation->donation_number}.png";
        Storage::put($path, $qr);
        return Storage::url($path);
    }
}
```

**PDF Receipt Template (Blade):**
```html
<div class="receipt">
    <div class="header">
        <img src="{{ asset('images/logo.png') }}" class="logo">
        <h1>DONATION RECEIPT</h1>
        <p class="receipt-number">#{{ $receipt_number }}</p>
    </div>

    <div class="organization-info">
        <p><strong>{{ $org_name }}</strong></p>
        <p>{{ $org_address }}</p>
        <p>Tax ID: {{ $org_tax_id }}</p>
        <p>Phone: {{ $org_phone }}</p>
        <p>Website: {{ $org_website }}</p>
    </div>

    <div class="donor-info">
        <h3>DONOR DETAILS</h3>
        <p><strong>Name:</strong> {{ $donor_name }}</p>
        <p><strong>Email:</strong> {{ $donor_email }}</p>
        <p><strong>Phone:</strong> {{ $donor_phone }}</p>
        <p><strong>Address:</strong> {{ $donor_address }}</p>
    </div>

    <div class="donation-details">
        <h3>DONATION DETAILS</h3>
        <table>
            <tr>
                <td><strong>Campaign:</strong></td>
                <td>{{ $campaign_name }}</td>
            </tr>
            <tr>
                <td><strong>Amount:</strong></td>
                <td>{{ $currency }} {{ number_format($amount, 2) }}</td>
            </tr>
            <tr>
                <td><strong>Date:</strong></td>
                <td>{{ $donation_date }}</td>
            </tr>
            <tr>
                <td><strong>Payment Method:</strong></td>
                <td>{{ $payment_method }}</td>
            </tr>
            <tr>
                <td><strong>Tax Deductible:</strong></td>
                <td>Yes</td>
            </tr>
        </table>
    </div>

    <div class="qr-section">
        <p>Verify this receipt:</p>
        <img src="{{ $qr_code_url }}" class="qr-code">
        <p class="verification-url">{{ $verification_url }}</p>
    </div>

    <div class="footer">
        <p>This is a computer-generated receipt. No signature required.</p>
        <p>For inquiries, contact us at {{ $org_phone }} or {{ $org_email }}</p>
        <p style="text-align: center;">Generated on {{ now()->format('d M Y H:i:s') }}</p>
    </div>
</div>
```

---

### 7. GenerateDonationStatementPDF
**Trigger:** Manual request or scheduled yearly

**Job Details:**
```php
class GenerateDonationStatementPDF
{
    public function handle($donor_id, $fiscal_year = null, $start_date = null, $end_date = null)
    {
        $donor = User::findOrFail($donor_id);
        $profile = $donor->donor_profile;

        // Determine date range
        if ($fiscal_year) {
            $fy = FiscalYear::where('fiscal_year', $fiscal_year)->first();
            $start_date = $fy->start_date;
            $end_date = $fy->end_date;
        } elseif (!$start_date || !$end_date) {
            $start_date = now()->startOfYear();
            $end_date = now()->endOfYear();
        }

        // Get donations
        $donations = Donation::where('user_id', $donor_id)
            ->whereBetween('donation_date', [$start_date, $end_date])
            ->with('campaign')
            ->orderBy('donation_date', 'desc')
            ->get();

        // Calculate stats
        $totalAmount = $donations->sum('amount');
        $donationCount = $donations->count();
        $avgAmount = $donationCount > 0 ? $totalAmount / $donationCount : 0;
        $campaigns = $donations->groupBy('campaign_id')
            ->map(fn($group) => [
                'name' => $group->first()->campaign->name,
                'amount' => $group->sum('amount'),
                'count' => $group->count()
            ]);

        $data = [
            'donor_name' => $profile->full_name,
            'donor_email' => $donor->email,
            'start_date' => $start_date->format('d M Y'),
            'end_date' => $end_date->format('d M Y'),
            'total_amount' => $totalAmount,
            'donation_count' => $donationCount,
            'avg_amount' => $avgAmount,
            'donations' => $donations,
            'campaigns' => $campaigns,
            'org_name' => config('donation.organization_name'),
            'org_logo' => asset('images/logo.png')
        ];

        // Generate PDF
        $pdf = PDF::loadView('statements.template', $data);
        
        // Store
        $fileName = "statement-{$donor_id}-{$start_date->format('Y-m-d')}-{$end_date->format('Y-m-d')}.pdf";
        $filePath = "storage/app/pdf/statements/{$fileName}";
        Storage::put($filePath, $pdf->output());

        // Create document record
        Document::create([
            'document_number' => "STM-" . now()->format('YmdHis'),
            'document_type' => 'STATEMENT',
            'user_id' => $donor_id,
            'file_name' => $fileName,
            'file_path' => $filePath,
            'file_size' => strlen($pdf->output()),
            'generated_at' => now(),
            'metadata' => [
                'start_date' => $start_date->toDateString(),
                'end_date' => $end_date->toDateString(),
                'fiscal_year' => $fiscal_year
            ]
        ]);
    }
}
```

---

### 8. GenerateDonationCertificatePDF
**Trigger:** Manual request or auto-generate for fiscal year end

**Job Details:**
```php
class GenerateDonationCertificatePDF
{
    public function handle($donor_id, $fiscal_year)
    {
        $donor = User::findOrFail($donor_id);
        $profile = $donor->donor_profile;

        // Get fiscal year config
        $fy = FiscalYear::where('fiscal_year', $fiscal_year)->first();
        
        // Get donations for fiscal year
        $donations = Donation::where('user_id', $donor_id)
            ->whereBetween('donation_date', [$fy->start_date, $fy->end_date])
            ->where('status', 'CONFIRMED')
            ->get();

        $totalAmount = $donations->sum('amount');
        $donationCount = $donations->count();

        $data = [
            'donor_name' => $profile->full_name,
            'total_amount' => $totalAmount,
            'donation_count' => $donationCount,
            'fiscal_year' => $fiscal_year,
            'period_start' => $fy->start_date->format('d M Y'),
            'period_end' => $fy->end_date->format('d M Y'),
            'certificate_number' => "CERT-{$fiscal_year}-" . str_pad($donor_id, 8, '0', STR_PAD_LEFT),
            'issue_date' => now()->format('d M Y'),
            'org_name' => config('donation.organization_name'),
            'org_seal' => asset('images/seal.png'),
            'authorized_signature' => asset('images/signature.png')
        ];

        // Generate certificate PDF
        $pdf = PDF::loadView('certificates.template', $data);
        
        // Store
        $fileName = "certificate-{$donor_id}-fy{$fiscal_year}.pdf";
        $filePath = "storage/app/pdf/certificates/{$fileName}";
        Storage::put($filePath, $pdf->output());

        // Create document record
        Document::create([
            'document_number' => $data['certificate_number'],
            'document_type' => 'CERTIFICATE',
            'user_id' => $donor_id,
            'file_name' => $fileName,
            'file_path' => $filePath,
            'file_size' => strlen($pdf->output()),
            'generated_at' => now(),
            'metadata' => ['fiscal_year' => $fiscal_year]
        ]);
    }
}
```

---

## Recurring Billing Jobs

### 9. ProcessRecurringDonations
**Trigger:** Daily scheduler at 00:00 (midnight)
**Duration:** Runs for ~2-5 minutes depending on load

**Job Details:**
```php
class ProcessRecurringDonations
{
    public function handle(PaymentService $paymentService)
    {
        // Find all plans due for charging today
        $dueToday = RecurringPlan::where('next_charge_date', Carbon::today())
            ->where('status', 'ACTIVE')
            ->with('user.donor_profile', 'campaign', 'paymentTransactions')
            ->get();

        Log::info("Processing {$dueToday->count()} recurring donations");

        foreach ($dueToday as $plan) {
            try {
                // Create idempotency key to prevent double charges
                $idempotencyKey = hash('sha256', "{$plan->id}-{$plan->next_charge_date}");

                // Check if already charged today (idempotency)
                $existingCharge = PaymentTransaction::where(
                    'idempotency_key',
                    $idempotencyKey
                )->exists();

                if ($existingCharge) {
                    Log::warning("Duplicate charge attempted for plan {$plan->id}");
                    continue;
                }

                // Attempt payment
                $transaction = $paymentService->chargeRecurringPlan(
                    $plan,
                    $idempotencyKey
                );

                // If successful
                if ($transaction->status === 'CAPTURED') {
                    // Create donation record
                    $donation = Donation::create([
                        'donation_number' => $this->generateDonationNumber(),
                        'user_id' => $plan->user_id,
                        'campaign_id' => $plan->campaign_id,
                        'recurring_plan_id' => $plan->id,
                        'amount' => $plan->amount,
                        'currency' => $plan->currency,
                        'donation_type' => 'RECURRING_INSTANCE',
                        'payment_method' => $plan->payment_method,
                        'status' => 'CONFIRMED',
                        'donation_date' => Carbon::today(),
                        'fiscal_year' => app(FiscalYearService::class)->getCurrentFiscalYear()
                    ]);

                    // Update plan
                    $plan->update([
                        'last_charge_date' => Carbon::today(),
                        'next_charge_date' => Carbon::today()->addMonthsNoOverflow($this->getFrequencyMonths($plan->frequency)),
                        'total_charges_completed' => $plan->total_charges_completed + 1,
                        'failure_count' => 0,
                        'failure_reason' => null,
                        'next_retry_date' => null
                    ]);

                    // Queue async jobs
                    dispatch(new SendDonationReceiptEmail($donation->id));
                    dispatch(new SendThankYouLetterEmail($donation->id));

                    // Create recurring log
                    RecurringPaymentLog::create([
                        'recurring_plan_id' => $plan->id,
                        'charge_date' => Carbon::today(),
                        'status' => 'SUCCESS',
                        'payment_transaction_id' => $transaction->id
                    ]);

                    Log::info("Successfully charged recurring plan {$plan->id}");
                }
            } catch (PaymentFailedException $e) {
                // Handle payment failure
                $this->handleRecurringPaymentFailure($plan, $e->getMessage());
            } catch (Exception $e) {
                Log::error("Error processing recurring plan {$plan->id}: " . $e->getMessage());
                // Don't fail the entire job - continue to next plan
                continue;
            }
        }

        // Check for final failures and pause plans
        $this->checkAndPauseFinalFailures();
    }

    private function handleRecurringPaymentFailure(RecurringPlan $plan, $errorMessage)
    {
        $plan->increment('failure_count');
        $plan->update(['failure_reason' => $errorMessage]);

        // Log the attempt
        RecurringPaymentLog::create([
            'recurring_plan_id' => $plan->id,
            'charge_date' => Carbon::today(),
            'status' => 'FAILED',
            'error_message' => $errorMessage,
            'retry_attempt' => $plan->failure_count
        ]);

        if ($plan->failure_count < $plan->max_failures) {
            // Schedule retry for next day
            $plan->update(['next_retry_date' => Carbon::tomorrow()]);
            
            // Send email
            dispatch(new SendPaymentFailureNotificationEmail($plan));
        } else {
            // Final failure - pause plan
            $plan->update(['status' => 'FAILED']);
            
            // Send cancellation email
            dispatch(new SendSubscriptionCancelledEmail($plan));
        }

        Log::warning("Recurring charge failed for plan {$plan->id}, attempt {$plan->failure_count}/{$plan->max_failures}");
    }

    private function checkAndPauseFinalFailures()
    {
        $finalFailures = RecurringPlan::where('failure_count', '>=', RecurringPlan::MAX_FAILURES)
            ->where('status', 'ACTIVE')
            ->update(['status' => 'FAILED']);

        if ($finalFailures > 0) {
            Log::warning("Paused {$finalFailures} recurring plans due to max failures");
        }
    }

    private function getFrequencyMonths($frequency)
    {
        return match($frequency) {
            'WEEKLY' => 0, // Handle weekly separately
            'MONTHLY' => 1,
            'QUARTERLY' => 3,
            'SEMI_ANNUAL' => 6,
            'ANNUAL' => 12
        };
    }
}
```

---

### 10. RetryFailedRecurringCharges
**Trigger:** Daily scheduler at 14:00 (2 PM)
**Purpose:** Retry failed recurring charges

**Job Details:**
```php
class RetryFailedRecurringCharges
{
    public function handle()
    {
        // Find plans with failed charges to retry
        $plansToRetry = RecurringPlan::where('status', 'ACTIVE')
            ->where('failure_count', '>', 0)
            ->where('next_retry_date', '<=', Carbon::today())
            ->where('failure_count', '<', RecurringPlan::MAX_FAILURES)
            ->get();

        Log::info("Retrying {$plansToRetry->count()} failed recurring charges");

        foreach ($plansToRetry as $plan) {
            try {
                $transaction = app(PaymentService::class)->chargeRecurringPlan($plan);

                if ($transaction->status === 'CAPTURED') {
                    // Create donation and reset failures
                    $donation = Donation::create([
                        'donation_number' => $this->generateDonationNumber(),
                        'user_id' => $plan->user_id,
                        'campaign_id' => $plan->campaign_id,
                        'recurring_plan_id' => $plan->id,
                        'amount' => $plan->amount,
                        'currency' => $plan->currency,
                        'donation_type' => 'RECURRING_INSTANCE',
                        'status' => 'CONFIRMED',
                        'donation_date' => Carbon::today()
                    ]);

                    $plan->update([
                        'failure_count' => 0,
                        'failure_reason' => null,
                        'next_retry_date' => null
                    ]);

                    dispatch(new SendDonationReceiptEmail($donation->id));

                    Log::info("Successfully retried plan {$plan->id}");
                }
            } catch (Exception $e) {
                Log::error("Retry failed for plan {$plan->id}: " . $e->getMessage());
                continue;
            }
        }
    }
}
```

---

### 11. CheckRecurringSubscriptionExpiry
**Trigger:** Daily scheduler at 06:00 (6 AM)
**Purpose:** Check if plans have ended and update status

**Job Details:**
```php
class CheckRecurringSubscriptionExpiry
{
    public function handle()
    {
        // Find plans that have ended
        $expiredPlans = RecurringPlan::where('status', 'ACTIVE')
            ->where(function ($query) {
                $query->where('end_date', '<', Carbon::today())
                    ->orWhere(function ($q) {
                        $q->whereNotNull('total_charges_planned')
                            ->whereRaw('total_charges_completed >= total_charges_planned');
                    });
            })
            ->get();

        foreach ($expiredPlans as $plan) {
            $plan->update(['status' => 'EXPIRED']);
            
            // Send completion email
            dispatch(new SendSubscriptionCompleteEmail($plan));

            Log::info("Marked recurring plan {$plan->id} as EXPIRED");
        }
    }
}
```

---

## Data Cleanup Jobs

### 12. CleanupExpiredOTPs
**Trigger:** Hourly scheduler

**Job Details:**
```php
class CleanupExpiredOTPs
{
    public function handle()
    {
        User::whereNotNull('otp_expires_at')
            ->where('otp_expires_at', '<', now())
            ->update(['otp_code' => null, 'otp_expires_at' => null]);

        Log::info("Cleaned up expired OTP codes");
    }
}
```

---

### 13. ArchiveOldAuditLogs
**Trigger:** Monthly scheduler (1st of month)

**Job Details:**
```php
class ArchiveOldAuditLogs
{
    public function handle()
    {
        // Move logs older than 1 year to archive table
        AuditLog::where('created_at', '<', now()->subYear())
            ->moveToArchive();

        Log::info("Archived old audit logs");
    }
}
```

---

## Error Handling & Monitoring

### Job Failure Handling

```php
// In job class
public function failed(Exception $exception)
{
    Log::error("Job failed: " . class_basename($this), [
        'exception' => $exception->getMessage(),
        'trace' => $exception->getTraceAsString()
    ]);

    // Send alert to admin if critical
    if ($exception instanceof CriticalException) {
        Mail::to(config('admin.alert_email'))
            ->queue(new JobFailureAlert($exception));
    }
}
```

### Failed Jobs Table

```sql
CREATE TABLE failed_jobs (
    id BIGSERIAL PRIMARY KEY,
    connection VARCHAR(255),
    queue VARCHAR(255),
    payload LONGTEXT,
    exception LONGTEXT,
    failed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Retry Configuration

```php
// config/queue.php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'database'),
    'database' => env('DB_CONNECTION', 'pgsql'),
    'table' => 'failed_jobs',
],

// In artisan commands:
// php artisan queue:retry
// php artisan queue:forget
// php artisan queue:flush
```

---

## Scheduler Configuration

### Laravel Scheduler Setup

```php
// app/Console/Kernel.php

protected function schedule(Schedule $schedule)
{
    // Recurring donations - process daily at midnight
    $schedule->call(function () {
        dispatch(new ProcessRecurringDonations());
    })
    ->daily()
    ->at('00:00')
    ->timezone('Asia/Dhaka')
    ->runInBackground()
    ->withoutOverlapping();

    // Retry failed charges - daily at 2 PM
    $schedule->call(function () {
        dispatch(new RetryFailedRecurringCharges());
    })
    ->daily()
    ->at('14:00')
    ->timezone('Asia/Dhaka')
    ->runInBackground()
    ->withoutOverlapping();

    // Check expiry - daily at 6 AM
    $schedule->call(function () {
        dispatch(new CheckRecurringSubscriptionExpiry());
    })
    ->daily()
    ->at('06:00')
    ->timezone('Asia/Dhaka')
    ->runInBackground();

    // Monthly summary emails - 1st of month at 8 AM
    $schedule->call(function () {
        dispatch(new SendMonthlyDonationSummaryEmail());
    })
    ->monthly()
    ->at('08:00')
    ->timezone('Asia/Dhaka');

    // Annual statement emails - 1st of next fiscal year
    $schedule->call(function () {
        dispatch(new SendAnnualStatementEmail());
    })
    ->cron('0 8 1 7 *')  // July 1st at 8 AM
    ->timezone('Asia/Dhaka');

    // Cleanup expired OTPs - hourly
    $schedule->call(function () {
        dispatch(new CleanupExpiredOTPs());
    })
    ->hourly()
    ->runInBackground();

    // Archive logs - monthly
    $schedule->call(function () {
        dispatch(new ArchiveOldAuditLogs());
    })
    ->monthly()
    ->runInBackground();

    // Health check - every 5 minutes
    $schedule->call(function () {
        HealthCheck::run();
    })
    ->everyFiveMinutes()
    ->runInBackground();
}
```

---

## Monitoring & Alerts

### Queue Dashboard (optional add-on)

Install Laravel Horizon for monitoring:
```bash
composer require laravel/horizon
php artisan horizon:install
```

Access at: `/horizon`

### Manual Queue Monitoring

```bash
# Check failed jobs
php artisan queue:failed

# List specific failed jobs
php artisan queue:failed --count=20

# Retry a failed job
php artisan queue:retry {id}

# Flush all failed jobs
php artisan queue:flush
```

---

## Next Steps

See [Security & Compliance](06-SECURITY-COMPLIANCE.md) for data protection details.
