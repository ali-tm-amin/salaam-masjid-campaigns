# Salaam Masjid Campaign Workflow - Visual Diagram

## 🔄 Complete Workflow Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER JOURNEY                             │
└─────────────────────────────────────────────────────────────────┘

                    📱 User visits form
                           ↓
              ┌────────────────────────┐
              │  Fill Registration     │
              │  - Full Name           │
              │  - Email Address       │
              │  - Phone Number        │
              │  - Fundraising Goal    │
              └────────────────────────┘
                           ↓
                    Click "Submit"


┌─────────────────────────────────────────────────────────────────┐
│                       N8N WORKFLOW                              │
└─────────────────────────────────────────────────────────────────┘

    ┌──────────────────────────┐
    │  1. Receive Form Data    │
    │  (Webhook Trigger)       │
    │                          │
    │  POST /salaam-masjid     │
    └──────────┬───────────────┘
               │
               ↓
    ┌──────────────────────────┐
    │  2. Prepare Data         │
    │  (Code Node)             │
    │                          │
    │  - Generate unique ID    │
    │  - Extract first name    │
    │  - Format currency       │
    │  - Add timestamp         │
    │  - Set status: Pending   │
    └──────────┬───────────────┘
               │
               ↓
    ┌──────────────────────────┐
    │  3. Log to Google Sheets │
    │  (Google Sheets Append)  │
    │                          │
    │  Saves to "Pledges" tab: │
    │  - ID, Timestamp         │
    │  - Full Name, Email      │
    │  - Phone, Goal           │
    │  - Status: "Pending"     │
    └──────────┬───────────────┘
               │
               ↓
    ┌──────────────────────────┐
    │  4. Wait 24 Hours        │
    │  (Wait Node)             │
    │                          │
    │  ⏰ Pauses execution     │
    │  for 24 hours            │
    └──────────┬───────────────┘
               │
               ↓
    ┌──────────────────────────┐
    │  5. Check Campaign       │
    │     Status               │
    │  (Google Sheets Lookup)  │
    │                          │
    │  Search by unique ID     │
    │  Read current Status     │
    └──────────┬───────────────┘
               │
               ↓
    ┌──────────────────────────┐
    │  6. IF Campaign Not      │
    │     Created              │
    │  (Conditional Node)      │
    │                          │
    │  IF Status = "Pending"   │
    └──────┬──────────┬────────┘
           │          │
    YES    │          │  NO
           │          │
           ↓          ↓
    ┌──────────┐  ┌─────────────────┐
    │ Continue │  │ END             │
    │          │  │ (No reminder)   │
    └─────┬────┘  └─────────────────┘
          │
          ↓
    ┌──────────────────────────┐
    │  7. Send Reminder Email  │
    │  (Email Send Node)       │
    │                          │
    │  Sends personalized      │
    │  reminder with:          │
    │  - User's name           │
    │  - Fundraising goal      │
    │  - Campaign creation URL │
    └──────────┬───────────────┘
               │
               ↓
    ┌──────────────────────────┐
    │  8. Update Status to     │
    │     "Reminded"           │
    │  (Google Sheets Update)  │
    │                          │
    │  Marks as contacted      │
    │  Prevents duplicate      │
    │  reminders               │
    └──────────┬───────────────┘
               │
               ↓
           ┌───────┐
           │  END  │
           └───────┘


┌─────────────────────────────────────────────────────────────────┐
│                   GOOGLE SHEETS STRUCTURE                       │
└─────────────────────────────────────────────────────────────────┘

Sheet Name: "Pledges"

┌─────────────┬────────────┬───────────┬──────────────────┬────────────┬──────────┬───────────┬──────────────┐
│ ID          │ Timestamp  │ Full Name │ Email            │ Phone      │ Goal GBP │ Status    │ Campaign URL │
├─────────────┼────────────┼───────────┼──────────────────┼────────────┼──────────┼───────────┼──────────────┤
│ SM170000XYZ │ 14/03/2026 │ Ali Khan  │ ali@example.com  │ +44 7XXX   │ 500      │ Pending   │              │
│             │ 10:30:00   │           │                  │            │          │           │              │
├─────────────┼────────────┼───────────┼──────────────────┼────────────┼──────────┼───────────┼──────────────┤
│ SM170001ABC │ 14/03/2026 │ Sara Ahmed│ sara@example.com │ +44 7YYY   │ 750      │ Completed │ launchgood.. │
│             │ 11:15:00   │           │                  │            │          │           │              │
├─────────────┼────────────┼───────────┼──────────────────┼────────────┼──────────┼───────────┼──────────────┤
│ SM170002DEF │ 14/03/2026 │ Omar Ali  │ omar@example.com │ +44 7ZZZ   │ 1000     │ Reminded  │              │
│             │ 12:00:00   │           │                  │            │          │           │              │
└─────────────┴────────────┴───────────┴──────────────────┴────────────┴──────────┴───────────┴──────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                      STATUS DEFINITIONS                         │
└─────────────────────────────────────────────────────────────────┘

┌──────────────┬────────────────────────────────────────────────┐
│ Pending      │ User registered but hasn't created campaign    │
│              │ → Reminder will be sent after 24 hours         │
├──────────────┼────────────────────────────────────────────────┤
│ Completed    │ User created their campaign                    │
│              │ → No reminder will be sent                     │
├──────────────┼────────────────────────────────────────────────┤
│ Reminded     │ Reminder email was sent                        │
│              │ → Shows follow-up action was taken             │
└──────────────┴────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                    DECISION FLOW CHART                          │
└─────────────────────────────────────────────────────────────────┘

                After 24 Hours:
                      │
                      ↓
            ┌─────────────────┐
            │ Check Status in │
            │ Google Sheets   │
            └────────┬────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
    Status =              Status =
   "Pending"?           "Completed"?
         │                       │
         ↓                       ↓
    ┌─────────┐           ┌──────────┐
    │   YES   │           │    NO    │
    └────┬────┘           └─────┬────┘
         │                      │
         ↓                      ↓
  ┌──────────────┐      ┌──────────────┐
  │ Send         │      │ Do NOT send  │
  │ Reminder     │      │ Reminder     │
  │              │      │              │
  │ Update to    │      │ END          │
  │ "Reminded"   │      │              │
  └──────────────┘      └──────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                     EMAIL TEMPLATE FLOW                         │
└─────────────────────────────────────────────────────────────────┘

IF Status = "Pending", email contains:

    ┌─────────────────────────────────────┐
    │  Subject:                           │
    │  "Did you launch your Salaam        │
    │   Masjid campaign, [FirstName]?"    │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │  Greeting:                          │
    │  "As-salamu alaykum [Full Name]"    │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │  Reminder:                          │
    │  "Yesterday you registered to       │
    │   raise £[Goal] for Salaam Masjid"  │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │  Goal Display Box:                  │
    │  Large formatted amount             │
    │  £[Goal Amount]                     │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │  Call to Action:                    │
    │  [Complete My Campaign] Button      │
    │  → Links to LaunchGood              │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │  Footer Note:                       │
    │  "Already completed? Please ignore" │
    └─────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                   ADMIN WORKFLOW GUIDE                          │
└─────────────────────────────────────────────────────────────────┘

When a user creates their campaign:

    User completes → Notify Admin → Open Google → Find user's → Update Status
    LaunchGood      (email/call)     Sheets        row by       to "Completed"
    campaign                                        email              +
                                                                  Add Campaign
                                                                      URL

    This prevents the reminder email from being sent!


┌─────────────────────────────────────────────────────────────────┐
│                     TIMELINE EXAMPLE                            │
└─────────────────────────────────────────────────────────────────┘

Day 1 - 10:00 AM
├─ User submits form
├─ Data saved to Google Sheets (Status: Pending)
└─ 24-hour timer starts

Day 1 - 10:00 AM to Day 2 - 10:00 AM
├─ Workflow is waiting
├─ User can create campaign during this time
└─ Admin can update status to "Completed"

Day 2 - 10:00 AM
├─ Workflow wakes up
├─ Checks Google Sheets status
│
├─ IF Status = "Pending"
│  ├─ Sends reminder email
│  └─ Updates status to "Reminded"
│
└─ IF Status = "Completed"
   └─ Does nothing (workflow ends)


┌─────────────────────────────────────────────────────────────────┐
│                   KEY FEATURES                                  │
└─────────────────────────────────────────────────────────────────┘

✅ Automatic reminder system
✅ Conditional logic (only remind if needed)
✅ Unique ID tracking for each registration
✅ Phone number collection for follow-up
✅ Beautiful email template with branding
✅ Status tracking (Pending → Completed/Reminded)
✅ Campaign URL storage
✅ No duplicate reminders
✅ Easy manual override (change status)
✅ Full audit trail in Google Sheets

```

---

## 🔧 Workflow Nodes Breakdown

| Node # | Name | Type | Purpose |
|--------|------|------|---------|
| 1 | Receive Form Data | Webhook | Receives POST request from HTML form |
| 2 | Prepare Data | Code | Processes form data, generates unique ID |
| 3 | Log to Google Sheets | Google Sheets | Saves registration to spreadsheet |
| 4 | Wait 24 Hours | Wait | Pauses execution for 24 hours |
| 5 | Check Campaign Status | Google Sheets | Looks up current status by ID |
| 6 | IF Campaign Not Created | IF | Checks if status is still "Pending" |
| 7 | Send Reminder Email | Email Send | Sends personalized reminder |
| 8 | Update Status | Google Sheets | Marks as "Reminded" |

---

## 📊 Data Flow

```
HTML Form Data:
{
  "fullName": "Ali Khan",
  "email": "ali@example.com",
  "phone": "+44 7XXX XXXXXX",
  "goalAmt": 500
}
        ↓
Processed Data:
{
  "uniqueId": "SM1710412345XYZ",
  "fullName": "Ali Khan",
  "firstName": "Ali",
  "email": "ali@example.com",
  "phone": "+44 7XXX XXXXXX",
  "goalAmt": 500,
  "goalFmt": "£500",
  "campaignUrl": "https://www.launchgood.com/create...",
  "submittedAt": "14/03/2026, 10:30:00",
  "status": "Pending"
}
        ↓
Google Sheets Row:
| SM1710412345XYZ | 14/03/2026 10:30:00 | Ali Khan | ali@example.com | +44 7XXX | 500 | Pending | |
```

---

**All systems integrated and ready to deploy! 🚀**
