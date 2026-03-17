# Two-Email System Guide

Complete guide for the friendly confirmation + reminder email system.

## 📧 Overview

Your workflow now sends **TWO emails** to help users complete their campaigns:

1. **Confirmation Email** (5 minutes after registration) - Friendly, helpful, instructional
2. **Reminder Email** (24 hours later) - Gentle reminder (only if still pending)

---

## 🔄 Complete User Journey

```
User submits registration form
    ↓
Data saved to Google Sheets (Status: Pending)
    ↓
User redirected to LaunchGood
    ↓
[5 MINUTES LATER] ← NEW!
    ↓
Email 1: Confirmation Email Arrives
    ✓ Thank you message
    ✓ Step-by-step instructions
    ✓ Completion link (email pre-filled)
    ✓ Create campaign button
    ↓
User creates campaign on LaunchGood
    ↓
User clicks completion link in email
    ↓
Status updated to "Completed"
    ↓
[24 HOURS AFTER REGISTRATION]
    ↓
Workflow checks status in Google Sheets
    ↓
    IF Status = "Completed"
    └─→ ✅ No reminder sent!

    IF Status = "Pending"
    └─→ Email 2: Gentle Reminder
        Status updated to "Reminded"
```

---

## 📧 Email 1: Confirmation Email

### When Sent
**5 minutes** after form submission

### Why 5 Minutes?
- Gives user time to be redirected to LaunchGood
- Arrives while they're creating their campaign
- Perfect timing reminder about completion link
- Not too early (would be buried)
- Not too late (they might forget)

### Subject
```
Your Salaam Masjid Campaign - Next Steps ✨
```

### Content Highlights

**Friendly Greeting**:
```
As-salamu alaykum [First Name],

JazakAllahu Khayran for registering to raise £[Goal] for
the Salaam Masjid extension!
```

**Simple Instructions**:
```
📝 Quick Steps to Create Your Campaign:

1. Click the button below to go to LaunchGood
2. Set your goal to £[Goal]
3. Publish your campaign!
```

**Call to Action**:
- Big green button: "Create My Campaign Now"
- Links to LaunchGood with community ID

**Important Completion Reminder**:
```
⏰ Important - After Creating Your Campaign:
Take 10 seconds to mark it complete so we can track your campaign:
[Mark Campaign Complete] ← Link with email pre-filled
```

**Friendly Closing**:
```
Need help? Just reply to this email. We're here to support you!

May Allah bless your fundraising efforts 🤲
```

---

## 📧 Email 2: Reminder Email

### When Sent
**24 hours** after form submission (only if Status = "Pending")

### Purpose
Gentle nudge for users who:
- Haven't created their campaign yet
- Created it but forgot to mark complete

### Subject
```
Did you launch your Salaam Masjid campaign, [First Name]?
```

### Content
- Gentle reminder about registration
- Their fundraising goal (big display)
- Button to create campaign
- Link to mark complete (if already created)

---

## 🎯 Email Workflow Visual

```
┌─────────────────────────────────────────────────────────────┐
│                    FORM SUBMISSION                          │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ↓
              ┌────────────────┐
              │ Save to Google │
              │ Sheets         │
              │ Status: Pending│
              └────────┬───────┘
                       │
              ┌────────┴────────┐
              │                 │
              ↓                 ↓
    ┌─────────────────┐   ┌──────────────┐
    │ Wait 5 Minutes  │   │ Wait 24 Hours│
    └────────┬────────┘   └──────┬───────┘
             │                    │
             ↓                    ↓
    ┌─────────────────┐   ┌──────────────┐
    │ Email 1:        │   │ Check Status │
    │ Confirmation    │   │ in Sheets    │
    │ - Instructions  │   └──────┬───────┘
    │ - Completion    │          │
    │   link          │          ↓
    └─────────────────┘   ┌──────────────┐
                          │ IF Pending?  │
                          └──────┬───────┘
                                 │
                        ┌────────┴────────┐
                        │                 │
                    YES │                 │ NO
                        ↓                 ↓
                 ┌──────────────┐   ┌─────────┐
                 │ Email 2:     │   │ No      │
                 │ Reminder     │   │ Action  │
                 │              │   └─────────┘
                 │ Status →     │
                 │ "Reminded"   │
                 └──────────────┘
```

---

## ⚙️ Configuration

### Workflow File
**File**: `workflows/salaam-masjid-workflow-with-confirmation.json`

### New Nodes Added
1. **Wait 5 Minutes** - Delays before confirmation email
2. **Send Confirmation Email** - Friendly welcome email

### Email Settings
**Both emails use**:
- From: `noreply@salaammasjid.org.uk`
- Reply-To: `admin@salaammasjid.org.uk`
- Same email credentials

**Completion link**: `https://masjidsalaam.netlify.app/campaign-complete.html?email={{ email }}`

---

## 🧪 Testing the Two-Email System

### Test 1: Confirmation Email
```
1. Submit test registration
2. Wait 5 minutes (or reduce to 1 minute for testing)
3. Check email inbox
4. Verify confirmation email received
5. Click "Create My Campaign Now" button
6. Click "Mark Campaign Complete" link
7. Verify email pre-filled in completion form
```

### Test 2: Reminder Email (Happy Path)
```
1. Submit test registration
2. Wait 5 minutes → Confirmation email arrives
3. Mark campaign complete via link
4. Wait 24 hours (or reduce for testing)
5. ✅ NO reminder email should arrive
6. Check Google Sheets: Status = "Completed"
```

### Test 3: Reminder Email (Pending Path)
```
1. Submit test registration
2. Wait 5 minutes → Confirmation email arrives
3. DON'T mark campaign complete
4. Wait 24 hours (or reduce for testing)
5. ✅ Reminder email should arrive
6. Check Google Sheets: Status = "Reminded"
```

### Quick Testing (Reduce Wait Times)
To test quickly, temporarily change wait times in n8n:
- **Wait 5 Minutes** → Change to 1 minute
- **Wait 24 Hours** → Change to 2 minutes

**Remember**: Change back to normal times for production!

---

## 📊 Expected Outcomes

### Scenario A: User Completes Campaign
```
Time 0:      Registration submitted
Time 5min:   Confirmation email received
Time 10min:  User creates campaign
Time 12min:  User clicks completion link
Time 24hrs:  NO reminder email (status = "Completed")
```

### Scenario B: User Forgets
```
Time 0:      Registration submitted
Time 5min:   Confirmation email received
Time 24hrs:  Reminder email sent (status = "Reminded")
```

### Scenario C: User Creates But Doesn't Mark Complete
```
Time 0:      Registration submitted
Time 5min:   Confirmation email received
Time 10min:  User creates campaign (but doesn't mark complete)
Time 24hrs:  Reminder email sent with completion link
Time 24hr+5: User clicks completion link
Result:      Status changes to "Completed"
```

---

## 💡 Best Practices

### 1. Email Timing
- **5 minutes** is optimal for confirmation
- Arrives while user is still engaged
- Not so fast they haven't started yet
- Not so slow they forget

### 2. Clear Call to Actions
Both emails have:
- ✅ Big, obvious buttons
- ✅ Clear next steps
- ✅ Email pre-filled links
- ✅ Friendly, encouraging tone

### 3. Completion Link Everywhere
The completion link appears in:
- ✅ Form success message
- ✅ Confirmation email (5 min)
- ✅ Reminder email (24 hr)
- ✅ Thank you page (campaign-created.html)

### 4. Friendly Tone Throughout
- Islamic greetings (As-salamu alaykum)
- Arabic phrases (JazakAllahu Khayran, Masha'Allah)
- Encouraging language
- Offer of help ("Need help? Just reply")

---

## 🎨 Email Preview

### Confirmation Email Visual
```
┌─────────────────────────────────────────┐
│ ✨ Thank You for Registering!           │
│ Let's get your Salaam Masjid           │
│ campaign live                           │
├─────────────────────────────────────────┤
│                                         │
│ As-salamu alaykum Ali,                  │
│                                         │
│ JazakAllahu Khayran for registering    │
│ to raise £500 for the Salaam Masjid    │
│ extension!                              │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📝 Quick Steps to Create Your       │ │
│ │    Campaign:                        │ │
│ │                                     │ │
│ │ 1. Click the button below to       │ │
│ │    go to LaunchGood                │ │
│ │ 2. Set your goal to £500           │ │
│ │ 3. Publish your campaign!          │ │
│ └─────────────────────────────────────┘ │
│                                         │
│     [Create My Campaign Now]            │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ ⏰ Important - After Creating:      │ │
│ │                                     │ │
│ │ Mark it complete:                  │ │
│ │ [Mark Campaign Complete]           │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Need help? Just reply to this email.   │
│                                         │
├─────────────────────────────────────────┤
│ Salaam Masjid Kempston                 │
│ May Allah bless your fundraising 🤲    │
└─────────────────────────────────────────┘
```

---

## 🔧 Troubleshooting

### Confirmation Email Not Arriving

**Check**:
- Wait 5 minutes node is active
- Email credentials configured
- Spam folder
- n8n execution history for errors

**Solution**:
- Verify email node has correct credentials
- Check n8n logs
- Test with different email address

### Both Emails Arriving

**Problem**: User receives both confirmation and reminder

**Check**:
- Did user mark campaign complete?
- Is Google Sheets status "Completed"?
- Is IF node checking correct status?

**Solution**:
- Manually update status to "Completed" in Google Sheets
- Verify IF node condition is `Status = "Pending"`

### No Emails Arriving

**Check**:
- n8n workflow is Active
- Email credentials valid
- Workflow executed successfully

**Solution**:
- Check n8n execution history
- Verify email server settings
- Test email credentials

---

## 📈 Success Metrics

Track these in Google Sheets:

| Metric | How to Calculate | Target |
|--------|------------------|--------|
| **Confirmation Open Rate** | Users who opened confirmation email | >60% |
| **Completion Rate (5 min)** | Users who marked complete within 5 min of confirmation | >40% |
| **Completion Rate (24 hr)** | Users who completed before reminder | >70% |
| **Reminder Effectiveness** | Users who completed after reminder | >30% |
| **Overall Completion** | Total completed / Total registered | >80% |

---

## 🎯 Summary

**Two-Email System Benefits**:
- ✅ **Higher completion rates** - Users get immediate guidance
- ✅ **Better UX** - Friendly, helpful, instructional
- ✅ **Reduced support** - Clear instructions prevent confusion
- ✅ **Timely reminders** - 5-minute email arrives at perfect moment
- ✅ **Gentle nudges** - 24-hour reminder as backup
- ✅ **Pre-filled links** - Email parameter makes completion easy

**Result**: More users complete their campaigns, less manual tracking needed!

---

**May your campaign system run smoothly! 🚀**
