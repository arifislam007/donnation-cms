# Donation Management CMS - User Journeys & Screens

## Donor Portal User Journeys

### Journey 1: First-Time Donor Registration & One-Time Donation

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Visit Website                                                    │
│    ↓                                                                │
│    User lands on homepage → Sees campaign highlights               │
│    ↓                                                                │
│ 2. Click "Sign Up" Button                                           │
│    ↓                                                                │
│    Show Donor Registration Form                                    │
│    ├─ Email input (validation for existing account)               │
│    ├─ Password input (strength meter)                             │
│    ├─ Password confirmation                                        │
│    ├─ Full name input                                             │
│    ├─ Phone number input (WhatsApp icon)                          │
│    ├─ Terms & Conditions checkbox                                 │
│    └─ "Create Account" button                                     │
│    ↓                                                                │
│ 3. Form Validation Passed                                          │
│    ↓                                                                │
│    API creates user account (status: INACTIVE)                     │
│    OTP sent to email (expires in 10 minutes)                       │
│    ↓                                                                │
│ 4. Show OTP Verification Screen                                    │
│    ├─ "We sent a code to your email"                              │
│    ├─ 6-digit OTP input fields                                    │
│    ├─ "Resend OTP" link (after 30 seconds)                        │
│    ├─ Countdown timer                                              │
│    └─ "Verify & Continue" button                                  │
│    ↓                                                                │
│ 5. OTP Verified Successfully                                       │
│    ↓                                                                │
│    User account activated (status: ACTIVE)                         │
│    JWT token generated and stored                                  │
│    ↓                                                                │
│ 6. Redirect to Donor Dashboard                                     │
│    ├─ Welcome message: "Hi {{full_name}}, welcome!"               │
│    ├─ Profile completion prompt (50% profile)                     │
│    ├─ Featured campaigns carousel                                 │
│    ├─ "Make a Donation" CTA button                                │
│    └─ Quick stats (0 donations so far)                            │
│    ↓                                                                │
│ 7. Click "Make a Donation"                                         │
│    ↓                                                                │
│    Show Campaign Selection Screen                                 │
│    ├─ List of active campaigns with:                             │
│    │  ├─ Campaign banner image                                   │
│    │  ├─ Campaign name & description                            │
│    │  ├─ Progress bar (collected/target)                        │
│    │  ├─ Number of donors                                        │
│    │  ├─ "Donate Now" button per campaign                       │
│    │  └─ "Learn More" link                                      │
│    └─ Filters: Category, Status, Recurring allowed              │
│    ↓                                                                │
│ 8. Select a Campaign                                               │
│    ↓                                                                │
│    Show Donation Form                                             │
│    ├─ Campaign details (banner, description)                     │
│    ├─ Amount input (with suggestions: 500, 1000, 5000, etc.)     │
│    ├─ Currency selector (BDT, USD, etc.)                         │
│    ├─ Payment method selector:                                   │
│    │  ├─ Credit Card (VISA, Mastercard)                         │
│    │  ├─ Debit Card                                              │
│    │  ├─ Bank Transfer                                            │
│    │  ├─ Mobile Wallet (bKash, Nagad, Rocket)                   │
│    │  └─ UPI                                                     │
│    ├─ Donation type (One-Time | Recurring - hidden for now)      │
│    ├─ Anonymous checkbox                                          │
│    ├─ Tax deductible checkbox (pre-checked)                      │
│    ├─ Custom message textarea (optional)                          │
│    └─ "Continue to Payment" button                               │
│    ↓                                                                │
│ 9. Payment Processing                                              │
│    ↓                                                                │
│    Donation record created (status: PENDING)                      │
│    Payment gateway initialized (Stripe/Razorpay)                  │
│    ↓                                                                │
│    Redirect to Payment Gateway Checkout                           │
│    └─ User enters card/payment details                            │
│    ↓                                                                │
│ 10. Payment Successful (Webhook)                                   │
│    ↓                                                                │
│    Donation status → CONFIRMED                                    │
│    Queue async jobs:                                              │
│    ├─ GenerateReceiptPDF                                          │
│    ├─ SendReceiptEmail                                            │
│    ├─ SendThankYouLetter (optional)                              │
│    └─ UpdateDonorStats                                            │
│    ↓                                                                │
│ 11. Show Success Screen                                            │
│    ├─ ✓ Payment successful message                                │
│    ├─ Donation number: DND-20240122-00001                        │
│    ├─ Amount donated: ৳5,000                                      │
│    ├─ Campaign: Education for Rural Children                     │
│    ├─ "Download Receipt" button                                  │
│    ├─ "View Donation History" link                               │
│    └─ "Make Another Donation" button                             │
│    ↓                                                                │
│ 12. Email Notification Sent                                        │
│    ├─ Subject: "Your Donation Receipt - DND-20240122-00001"     │
│    ├─ Body: Thank you + donation details + receipt PDF           │
│    └─ CTA: "View Impact" link to campaign                        │
│    ↓                                                                │
│ 13. Donor Completes Journey                                        │
│    └─ User can now:                                               │
│       ├─ Complete profile                                          │
│       ├─ View donation history                                    │
│       ├─ Download documents                                       │
│       └─ Manage communication preferences                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Journey 2: Recurring (Monthly) Donation Setup

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Logged-In Donor Views Campaign                                   │
│    ↓                                                                │
│    Click "Donate Regularly" or "Monthly Donation"                 │
│    ↓                                                                │
│ 2. Show Recurring Donation Form                                    │
│    ├─ Amount input (e.g., ৳1,000)                                 │
│    ├─ Frequency selector:                                         │
│    │  ├─ Weekly                                                    │
│    │  ├─ Monthly (default)                                        │
│    │  ├─ Quarterly                                                │
│    │  ├─ Semi-Annual                                              │
│    │  └─ Annual                                                   │
│    ├─ Start date picker (default: tomorrow or next month)         │
│    ├─ End date picker (optional, leave blank for indefinite)      │
│    ├─ Planned number of charges (e.g., 12 for a year)            │
│    ├─ Payment method selector                                     │
│    ├─ Enable auto-retry on failure (default: ON)                 │
│    ├─ Notification preference:                                    │
│    │  ├─ Remind me 3 days before each charge                     │
│    │  └─ Send receipt after each charge                          │
│    ├─ Summary box showing:                                        │
│    │  ├─ "Total commitment: ৳12,000 over 12 months"            │
│    │  └─ First charge: {{start_date}}                            │
│    └─ "Set Up Recurring Donation" button                         │
│    ↓                                                                │
│ 3. Payment Setup (First Charge Authorization)                      │
│    ├─ Show payment form or redirect to gateway                    │
│    └─ Authorize mandate/subscription                              │
│    ↓                                                                │
│ 4. Recurring Plan Created                                          │
│    ├─ Plan number: REC-20240122-00001                            │
│    ├─ Status: ACTIVE                                              │
│    └─ Next charge date: {{start_date}}                            │
│    ↓                                                                │
│ 5. Confirmation Screen                                             │
│    ├─ ✓ Your monthly donation is set up                          │
│    ├─ You'll be charged ৳1,000 every month starting {{date}}    │
│    ├─ Plan number: REC-20240122-00001                            │
│    ├─ You can manage this from your dashboard anytime            │
│    ├─ "View My Plans" button                                     │
│    └─ "Back to Campaigns" button                                 │
│    ↓                                                                │
│ 6. Email Notification                                              │
│    ├─ Confirmation email with plan details                       │
│    ├─ Link to manage/cancel plan                                 │
│    └─ Reminder: "Your first charge will be on {{date}}"         │
│    ↓                                                                │
│ 7. Monthly Scheduler Runs (Every night at 00:00)                   │
│    ├─ Find all plans with next_charge_date = TODAY               │
│    ├─ For each plan:                                              │
│    │  ├─ Call payment gateway to charge                          │
│    │  └─ Create donation record with type: RECURRING_INSTANCE   │
│    ├─ If success:                                                │
│    │  ├─ Update plan: last_charge_date = today                 │
│    │  ├─ Set next_charge_date = today + 1 month                │
│    │  ├─ Queue: Send receipt & thank you email                 │
│    │  └─ Increment charge counter                               │
│    ├─ If failure (e.g., card declined):                          │
│    │  ├─ Increment failure_count                                │
│    │  ├─ Set next_retry_date = tomorrow                         │
│    │  ├─ If failure_count < max_retries (3):                   │
│    │  │  └─ Retry tomorrow                                     │
│    │  └─ Else if failure_count >= 3:                           │
│    │     ├─ Set plan status = FAILED                           │
│    │     ├─ Send "Donation Failed" email to donor              │
│    │     └─ Suggest: Update payment method                     │
│    └─ Generate email reports (optional)                          │
│    ↓                                                                │
│ 8. Donor Receives Monthly Emails                                   │
│    ├─ (Optional) 3 days before: "Your monthly donation reminder"  │
│    ├─ Next day after charge: "Your monthly donation receipt"      │
│    └─ End of month: "Your monthly giving summary"                │
│    ↓                                                                │
│ 9. Donor Can Manage Plan Anytime                                   │
│    ├─ View "My Recurring Plans" in dashboard                     │
│    ├─ For each plan, can:                                         │
│    │  ├─ View next charge date                                   │
│    │  ├─ View total charged so far                              │
│    │  ├─ Update amount/frequency                                │
│    │  ├─ Pause temporarily (with reason)                       │
│    │  ├─ Resume after pause                                     │
│    │  └─ Cancel (with reason)                                   │
│    └─ All changes are audit-logged                               │
│    ↓                                                                │
│ 10. Donor Cancels Plan (Optional)                                  │
│    ├─ Show cancellation confirmation dialog                      │
│    ├─ Reason selector (optional):                                │
│    │  ├─ Financial constraint                                    │
│    │  ├─ Duplicate plan                                          │
│    │  ├─ Changed my mind                                         │
│    │  └─ Other                                                   │
│    ├─ Confirmation: "You won't be charged after {{date}}"       │
│    └─ Cancellation email sent                                    │
│    ↓                                                                │
│ 11. Journey Ends - Recurring Donations Now Active                  │
│    └─ Donor can track all transactions in history               │
└─────────────────────────────────────────────────────────────────────┘
```

### Journey 3: Download Annual Donation Statement & Certificate

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Logged-In Donor Views Dashboard                                  │
│    ├─ Sees "Your Giving" summary widget                            │
│    ├─ "Download Statement" button highlighted                      │
│    └─ "View Certificates" section                                  │
│    ↓                                                                │
│ 2. Click "Download Annual Statement"                               │
│    ↓                                                                │
│ 3. Show Statement Generation Screen                                │
│    ├─ Fiscal year selector (dropdown):                            │
│    │  ├─ FY 2024-25 (Jul 2024 - Jun 2025)  ← Current            │
│    │  ├─ FY 2023-24 (Jul 2023 - Jun 2024)                        │
│    │  └─ FY 2022-23 (Jul 2022 - Jun 2023)                        │
│    ├─ OR Custom date range:                                       │
│    │  ├─ Start date picker                                        │
│    │  ├─ End date picker                                          │
│    │  └─ "Use Custom Range" button                               │
│    ├─ Include campaigns breakdown (checkbox)                       │
│    ├─ Include monthly breakdown (checkbox)                         │
│    ├─ Include payment methods (checkbox)                           │
│    └─ "Generate Statement" button                                │
│    ↓                                                                │
│ 4. System Generates PDF                                            │
│    ├─ Query donations for date range                              │
│    ├─ Generate PDF with:                                          │
│    │  ├─ Organization logo & name                                │
│    │  ├─ "Donation Statement" title                              │
│    │  ├─ Donor name & ID                                         │
│    │  ├─ Statement period: {{start_date}} to {{end_date}}       │
│    │  ├─ Total donated amount                                    │
│    │  ├─ Number of donations                                     │
│    │  ├─ Campaign-wise breakdown (if selected)                  │
│    │  ├─ Monthly breakdown (if selected)                        │
│    │  ├─ Payment methods used (if selected)                     │
│    │  ├─ Average donation amount                                │
│    │  ├─ Recurring vs One-time breakdown                        │
│    │  ├─ Tax deductible amount (for tax filing)                │
│    │  ├─ Organization stamp & signature area                    │
│    │  └─ Footer: "Generated on {{date}}"                        │
│    └─ Store PDF in storage/app/statements/                       │
│    ↓                                                                │
│ 5. Show Download Screen                                            │
│    ├─ ✓ Statement generated successfully!                         │
│    ├─ File name: "Statement-FY2024-JohnDoe.pdf"                 │
│    ├─ File size: 2.3 MB                                          │
│    ├─ "Download PDF" button (blue CTA)                           │
│    ├─ "Email to me" button (optional)                            │
│    ├─ "View in browser" link                                     │
│    └─ "Back to Dashboard" link                                   │
│    ↓                                                                │
│ 6. (Optional) Email Statement                                      │
│    ├─ Send email with:                                            │
│    │  ├─ Subject: "Your Annual Donation Statement - FY 2024"   │
│    │  ├─ Greeting: "Dear {{donor_name}}"                        │
│    │  ├─ "Attached is your donation statement..."              │
│    │  ├─ PDF attachment: statement-fy2024.pdf                  │
│    │  └─ CTA: "Login to view or download again"                │
│    └─ Confirmation: "Statement emailed!"                         │
│    ↓                                                                │
│ 7. Download Donation Certificate                                   │
│    ├─ System queries donations for fiscal year                    │
│    ├─ Generate certificate PDF:                                   │
│    │  ├─ Organization logo & name                                │
│    │  ├─ "Donation Certificate" title                            │
│    │  ├─ Border decoration / design                              │
│    │  ├─ "This is to certify that" section                      │
│    │  ├─ Donor full name                                         │
│    │  ├─ "Has made a generous donation of {{total_amount}}"    │
│    │  ├─ "During the fiscal year {{fiscal_year}}"              │
│    │  ├─ Certificate number: CERT-2024-0001234                  │
│    │  ├─ Verification QR code (links to verify.site)            │
│    │  ├─ Organization signature & seal                          │
│    │  ├─ Issue date: {{generated_date}}                         │
│    │  └─ Footer: "This certificate is issued for tax purposes"  │
│    ├─ Store PDF in storage/app/certificates/                     │
│    └─ Create document record in database                          │
│    ↓                                                                │
│ 8. Show Certificate Download                                       │
│    ├─ ✓ Certificate generated!                                    │
│    ├─ "Download Certificate" button                              │
│    ├─ "Email Certificate" button                                 │
│    └─ View list of all certificates (historical)                 │
│    ↓                                                                │
│ 9. Manage Preferences (Optional)                                   │
│    ├─ Option to automatically email next year's statement        │
│    ├─ Option to receive certificates yearly                      │
│    └─ Save preferences                                            │
│    ↓                                                                │
│ 10. Journey Ends                                                    │
│    └─ Donor has downloadable/printable tax documents              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Screen Specifications

### Screen 1: Homepage / Landing Page

**Layout:**
- Header: Navigation bar with Logo, Campaign list link, Dashboard link, Login/Sign Up buttons
- Hero Section:
  - Background image or video
  - Title: "Make a Difference Today"
  - Subtitle: "Support causes you care about"
  - CTA Button: "Explore Campaigns" or "Donate Now"
  - Stats row: "{{total_donors}} Donors | {{total_collected}} Donated | {{active_campaigns}} Active Campaigns"
- Featured Campaigns Carousel:
  - 3-4 campaigns displayed horizontally
  - Each card shows: Image, Name, Description, Progress bar, "Donate Now" button
- Benefits Section:
  - Icon + Text: "Secure Payments"
  - Icon + Text: "Instant Receipts"
  - Icon + Text: "Track Impact"
  - Icon + Text: "Tax Deductible"
- Impact Stories Section:
  - Testimonials from donors
  - Before/After images
- Call-to-Action Section:
  - "Join {{total_donors}} donors making an impact"
  - "Sign Up" button
- Footer: About, Contact, FAQ, Privacy Policy, Terms, Social links

### Screen 2: Donor Registration Form

**Fields:**
- Email (email input, unique validation)
- Password (password input, strength indicator: Weak/Fair/Good/Strong)
- Confirm Password (password input)
- Full Name (text input)
- Phone Number (tel input, formatted with country code +880)
- Terms & Conditions (checkbox with link)
- "Create Account" button
- "Already have account? Login" link

**Validation Rules:**
- Email: Valid format, unique in system
- Password: Min 8 chars, 1 uppercase, 1 number, 1 special char
- Phone: Valid Bangladesh number (+88017xxx or +88018xxx)
- Terms: Must be checked

**Error States:**
- Email already exists → "This email is already registered. Try Login"
- Weak password → Show strength feedback
- Invalid phone → "Enter valid Bangladesh phone number"

### Screen 3: OTP Verification

**Elements:**
- Message: "We've sent a 6-digit code to {{email}}"
- 6 input fields for digit entry (auto-move to next field)
- "Verify" button
- "Resend Code" link (appears after 30 seconds)
- Countdown timer: "Resend available in 00:30"
- "Change Email" link

**Validation:**
- All 6 digits must be entered
- OTP must be valid and not expired
- Max 3 failed attempts → Lock for 15 minutes

### Screen 4: Donor Dashboard (Main Hub)

**Layout:**
- Header with: Logo, "Hello {{donor_name}}" greeting, Notification bell, Profile menu
- Left Sidebar Navigation:
  - Dashboard (icon + text)
  - Make Donation (icon + text)
  - My Donations (icon + text)
  - Recurring Plans (icon + text)
  - Documents & Statements (icon + text)
  - Profile (icon + text)
  - Preferences (icon + text)
  - Logout

- Main Content Area:
  1. **Welcome Widget:**
     - "Welcome back, {{donor_name}}!"
     - Profile completion percentage (e.g., "50% Complete")
     - "Complete Your Profile" CTA

  2. **Your Giving Summary:**
     - Card showing:
       - Total donated: {{total_amount}}
       - Number of donations: {{count}}
       - Active recurring plans: {{active_plans}}
       - This month's giving: {{monthly_amount}}
     - "View History" button

  3. **Featured Campaigns:**
     - Horizontal carousel of 3-4 campaigns
     - Each card: Image, name, progress bar, "Donate" button
     - "View All Campaigns" link

  4. **Active Recurring Plans:**
     - List of current recurring plans
     - Each plan shows: Campaign name, Amount, Frequency, Next charge date, Status
     - "Manage Plans" button for each
     - "Manage All Plans" link

  5. **Recent Documents:**
     - Latest 3 receipts/certificates
     - Each shows: Type, Date, File size, Download button
     - "View All Documents" link

  6. **Quick Actions:**
     - Buttons: "Make a Donation", "Download Statement", "View Certificates"

  7. **Impact Updates (Optional):**
     - Updates from campaigns donor has supported
     - Images and brief descriptions
     - "Learn More" links

### Screen 5: Campaign List & Details

**Campaign List Page:**
- Filter sidebar (left):
  - Category dropdown (Education, Healthcare, etc.)
  - Status selector (Active, Completed)
  - Recurring allowed (toggle)
  - Amount range slider
  
- Main content:
  - Grid or list view toggle
  - Sorting: "Featured", "Most Donated", "Ending Soon"
  - Campaign cards (grid):
    - Large banner image
    - Campaign name (h3)
    - Category badge
    - Description (2 lines)
    - Progress bar with collected/target amounts
    - Number of donors
    - "Donate Now" button
    - "Learn More" link

**Campaign Detail Page:**
- Hero section: Large banner image
- Campaign info box:
  - Name (h1)
  - Category badge
  - Status (Active/Completed)
  - Created date and creator
  
- Description & Impact:
  - Long description (p)
  - Impact statement
  - Video player (if available)
  
- Stats section:
  - Total collected
  - Progress bar
  - Target amount
  - Number of donors
  - Days remaining
  
- Donation CTAs:
  - "Make One-Time Donation" button (prominent)
  - "Set Up Monthly Giving" button
  
- Impact updates (carousel):
  - Dated updates with images
  - Brief descriptions
  - "Read More" links
  
- Transparency:
  - Breakdown: "Total 50,000 donors invested {{amount}}"
  - Top 5 donors (anonymous or named)
  - "See all donors" link

### Screen 6: Donation Form (One-Time)

**Form Fields:**
1. **Campaign Selection** (pre-filled if coming from campaign page)
   - Campaign name (read-only)
   - Campaign image (thumbnail)
   
2. **Donation Amount**
   - Input field with currency symbol (৳)
   - Quick amount buttons: 500, 1000, 5000, 10000, "Custom"
   - Slider for amount (min 100, max 500000)
   
3. **Currency Selector**
   - Dropdown: BDT (৳), USD ($), EUR (€), GBP (£)
   
4. **Payment Method**
   - Radio buttons:
     - Credit Card (VISA, Mastercard icon)
     - Debit Card (icon)
     - Bank Transfer (icon)
     - Mobile Wallet:
       - bKash
       - Nagad
       - Rocket
     - UPI
   
5. **Donor Information** (if not logged in):
   - Full name (text)
   - Email (email)
   - Phone (tel)
   - Checkbox: "Save info for future donations"
   
6. **Additional Options:**
   - Anonymous checkbox: "Make this donation anonymous"
   - Tax deductible checkbox: "This donation is tax deductible" (info tooltip)
   - Message textarea: "Add a personal message (optional)" (max 500 chars)
   
7. **Summary Box:**
   - Donation amount: {{amount}} {{currency}}
   - Currency conversion (if not BDT): "≈ {{converted_amount}} BDT"
   - Processing fee: {{fee}} (if applicable)
   - Total: {{total}}
   
8. **CTAs:**
   - "Continue to Payment" button (prominent blue)
   - "Cancel" link

### Screen 7: Recurring Donation Form

**Form Fields:**
1. **Campaign** (pre-filled)
2. **Amount**
   - Input field with currency
   - Suggestions based on campaign min/max
   
3. **Frequency Selector:**
   - Radio buttons:
     - Weekly
     - Monthly (default)
     - Quarterly
     - Semi-Annual
     - Annual
   
4. **Duration:**
   - Start date picker (default: tomorrow or next 1st of month)
   - End date picker (optional, label: "Leave blank for indefinite giving")
   - OR planned charges: "I want to give for {{input}} months"
   
5. **Payment Method** (same as one-time)

6. **Auto-Retry Settings:**
   - Toggle: "Retry failed charges automatically"
   - Text: "We'll try up to 3 times if a charge fails"
   
7. **Notifications:**
   - Checkboxes:
     - [ ] Remind me 3 days before each charge
     - [ ] Send receipt after each charge
     - [ ] Monthly summary email
   
8. **Commitment Summary:**
   - Box showing:
     - "Your commitment:"
     - "{{amount}} {{currency}} every {{frequency}}"
     - "Starting {{start_date}}"
     - "Total {{total_commitment}} over {{duration}}"
     - "You can pause or cancel anytime"
   
9. **CTAs:**
   - "Set Up Recurring Donation" (blue)
   - "Cancel" link

### Screen 8: Donation Success

**Elements:**
- ✓ Large green checkmark icon
- "Thank You!" heading
- "Your donation has been processed successfully"
- Donation details box:
  - Donation number: {{donation_number}}
  - Amount: {{amount}} {{currency}}
  - Campaign: {{campaign_name}}
  - Date: {{donation_date}}
  - Receipt: "Your receipt has been sent to {{email}}"
  
- Quick actions:
  - "Download Receipt" button
  - "View Campaign Impact" button
  - "Back to Dashboard" button
  - "Make Another Donation" button (secondary)
  
- Message:
  - "A receipt has been sent to {{email}}"
  - "A thank you letter will follow"

### Screen 9: Donation History / List

**Table columns:**
- Date
- Donation Number
- Campaign Name
- Amount
- Status (badge: Confirmed/Pending/Failed)
- Payment Method (icon)
- Actions (View, Receipt, Refund if applicable)

**Filters:**
- Date range picker
- Campaign dropdown
- Status dropdown
- Amount range slider
- Payment method selector

**Sorting:**
- By date (newest first)
- By amount
- By campaign

**Pagination:**
- Per page selector (10, 20, 50)
- Page indicators

**Row Actions:**
- View details
- Download receipt
- View certificate (if available)
- Request refund button (if applicable)

### Screen 10: Recurring Plans Management

**List View:**
- Table with columns:
  - Campaign Name
  - Amount & Frequency
  - Status (badge: Active/Paused/Failed)
  - Next Charge Date
  - Total Charged So Far
  - Actions (Edit, Pause/Resume, Cancel)
  
**For each plan, show:**
- Inline status indicator
- "Edit" button → Opens modal to change amount/frequency
- "Pause" button → Modal for pause reason
- "Resume" button (if paused)
- "Cancel" button → Confirmation dialog with reason

**Summary:**
- Total active recurring plans
- Total monthly commitment
- Projected annual giving

### Screen 11: Profile Management

**Sections:**

1. **Personal Information:**
   - Full name (text input)
   - Email (read-only display + verified badge)
   - Phone (tel input)
   - Gender (radio: Male/Female/Other/Prefer not to say)
   - Date of Birth (date picker)
   
2. **Address:**
   - Address Line 1 (text)
   - Address Line 2 (text, optional)
   - City (text)
   - State/Province (text)
   - Postal Code (text)
   - Country (dropdown, pre-filled: Bangladesh)
   
3. **Organization Info (Optional):**
   - Organization Name (text)
   - Organization Type (dropdown: Individual/Corporate/NGO/Trust)
   
4. **Tax Information:**
   - Tax ID (text, encrypted)
   - Tax ID Verification Status (badge: Verified/Pending/Rejected)
   - "Upload Tax Document" button
   - PAN Number (optional, text)
   
5. **Bank Information (Optional):**
   - Bank Account Number (text, masked display: ****7890)
   - Bank Name (text)
   - IFSC Code (text)
   
6. **KYC Status:**
   - Badge showing: Pending/Verified/Rejected
   - If rejected: Show rejection reason
   - "Upload KYC Document" button
   - Document upload modal

**Actions:**
- "Save Changes" button
- "Cancel" button
- "Change Password" link

### Screen 12: Communication Preferences

**Email Notifications Section:**
- Checkboxes:
  - [ ] Donation receipt email (toggle)
  - [ ] Monthly giving summary (toggle)
  - [ ] Annual statement email (toggle)
  - [ ] Campaign updates (toggle)
  - [ ] Thank you letters (toggle)
  - [ ] Payment failure notifications (toggle)
  - [ ] Upcoming charge reminders (toggle, only for recurring)
  - [ ] Promotional emails (toggle, pre-unchecked)

**SMS Notifications:**
- Checkbox: [ ] SMS notifications (for one-time alerts)
- Phone number display (edit option)

**Preferred Language:**
- Dropdown: English / Bangla (বাংলা)

**Newsletter:**
- Frequency selector: Weekly / Monthly / Quarterly / Annually
- Topics selector: [ ] Impact Stories, [ ] Campaign News, [ ] Events

**Unsubscribe All:**
- Button: "Unsubscribe from all emails"
- Confirmation: "You will only receive essential notifications"

**Save Button:**
- "Save Preferences" button

---

## Admin CMS Screens

### Screen A1: Admin Dashboard

**Header:**
- Logo, "Admin Console" title, Admin name, Notification bell, Settings menu

**Sidebar Navigation:**
- Dashboard
- Donors Management
- Donations Management
- Campaigns Management
- Recurring Plans
- Templates
- Email Logs
- Reports
- Audit Logs
- Settings
- Logout

**Main Content:**
1. **Key Metrics (Cards):**
   - Total Donors (number, +X% this month)
   - Total Donations (amount, +X% this month)
   - Active Campaigns (count)
   - Active Recurring Plans (count)

2. **Charts:**
   - Daily donations trend (line chart, last 30 days)
   - Campaign performance (bar chart)
   - Donation methods (pie chart)

3. **Recent Transactions:**
   - Table: Latest 10 donations
   - Columns: Date, Donor, Campaign, Amount, Status

4. **Failed Recurring Plans:**
   - Alert box if > 0
   - List of failed plans with "Retry" button

5. **Email Queue Status:**
   - Pending emails count
   - Failed emails count (alert if > 0)

### Screen A2: Donor Management List

**Filters (Left Sidebar):**
- Search box (name, email, phone)
- KYC Status dropdown
- Registration date range
- Lifetime donation amount range
- Donation count range
- Last donation date range

**Main Table:**
- Columns: Name, Email, Phone, City, KYC Status, Lifetime Donation, Donations Count, Last Donation, Actions
- Sorting: By name, email, lifetime donation, last donation
- Pagination: 20 per page

**Row Actions:**
- View profile (icon link)
- Edit (icon link)
- Suspend (icon link with confirmation)
- Merge (checkbox select multiple, then "Merge Selected" button)
- Export to CSV (bulk action)

**Top Actions:**
- "New Donor" button
- "Export CSV" button
- "Merge Duplicates" button
- Filters panel toggle

### Screen A3: Donation Management

**Filters:**
- Date range
- Donor name search
- Campaign dropdown
- Status dropdown
- Payment method dropdown
- Amount range

**Table:**
- Columns: Date, Donation #, Donor Name, Campaign, Amount, Status, Payment Method, Actions

**Row Actions:**
- View details
- Edit (manual entries only)
- Refund button
- Download receipt
- Resend receipt email

**Bulk Actions:**
- Mark as verified (for cheques/transfers)
- Export selected to CSV

**Top Actions:**
- "Manual Entry" button (create offline donation)
- "Generate Report" button

### Screen A4: Campaign Management

**List View:**
- Table with: Name, Category, Status, Target, Collected, % Progress, Donors, Actions
- Sorting and filtering

**Row Actions:**
- View campaign
- Edit campaign
- View analytics
- Publish/Pause/Complete
- Delete

**Create/Edit Campaign Modal:**
- Name
- Slug (auto-generate from name)
- Category dropdown
- Description (rich text editor)
- Impact statement (textarea)
- Target amount
- Start date
- End date
- Image upload (banner)
- Recurring allowed (toggle)
- Min/Max donation amounts
- "Save" button

### Screen A5: Reports

**Report Types (Tabs/Menu):**
1. Daily Collections - Date picker, table with: Date, Amount, Count, Methods breakdown
2. Campaign Performance - Date range, table with campaign names, collections, %, donors
3. Donor Lifetime Value - Segments, bar chart, table
4. Tax Year Statement - Fiscal year dropdown, stats, export button
5. Payment Methods - Pie chart, table with method, count, amount, %
6. Email Analytics - Emails sent, opened, failed, bounce rate

**Export Options:**
- CSV download button
- PDF report button
- Email report button

### Screen A6: Template Management

**Filters:**
- Template type dropdown (Email / Document)
- Email type dropdown (if Email selected)
- Search by name

**Table:**
- Columns: Name, Type, Email/Doc Type, Version, Status, Active, Actions

**Row Actions:**
- Preview (modal)
- Edit
- Deactivate/Activate
- Create version (if editing)
- Delete

**Create/Edit Template Modal:**
- Name (text)
- Template type (radio: Email / Document)
- Email type or Document type (dropdown)
- Subject (textarea, for emails)
- Body (rich text editor)
- Placeholder helper (dropdown of available placeholders)
- HTML editor (for emails, optional)
- Version note (textarea)
- "Save as Draft" / "Save & Publish" buttons

**Placeholder Help Section:**
- List of available placeholders:
  - {{donor_name}}
  - {{donor_email}}
  - {{donation_amount}}
  - {{currency}}
  - {{donation_date}}
  - {{campaign_name}}
  - {{organization_name}}
  - {{fiscal_year}}
  - {{receipt_number}}
  - {{certificate_number}}

---

## Responsive Design Considerations

All screens should be responsive:

**Desktop (1920px+):**
- Full sidebar navigation
- Multi-column layouts
- Full tables

**Tablet (768px - 1024px):**
- Collapsible sidebar
- 2-column layouts
- Table pagination emphasized

**Mobile (< 768px):**
- Hamburger menu (sidebar)
- Single-column layouts
- Stacked cards
- Simplified tables (show key columns, swipe for more)
- Touch-friendly buttons (min 44px height)

---

## Accessibility Features

- WCAG 2.1 AA compliance
- Semantic HTML5
- ARIA labels for form inputs
- Color contrast ratios ≥ 4.5:1
- Keyboard navigation support
- Screen reader compatibility
- Skip to main content link
- Focus indicators visible
- Error messages linked to form fields
- Form labels associated with inputs

---

## Next Steps

See [Background Jobs](05-BACKGROUND-JOBS.md) for job queue implementation details.
