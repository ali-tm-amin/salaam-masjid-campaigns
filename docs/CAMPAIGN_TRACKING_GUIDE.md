# Campaign Tracking Guide - How the System Knows When Campaigns Are Created

## 🤔 The Problem

**Question**: "How does the workflow know if the user created their campaign on LaunchGood?"

**Answer**: LaunchGood doesn't automatically notify us, so we've built multiple tracking solutions.

---

## 🎯 Tracking Solutions

I've implemented **3 methods** for tracking campaign completion:

### Method 1: Self-Reporting Form (RECOMMENDED) ⭐
### Method 2: Manual Admin Update
### Method 3: LaunchGood Webhook (Advanced - requires LaunchGood API access)

---

## Method 1: Self-Reporting Form (RECOMMENDED) ⭐

### How It Works

```
User creates campaign on LaunchGood
       ↓
User clicks "Mark Campaign Complete" link
       ↓
Fills simple form with email + campaign URL
       ↓
n8n receives webhook
       ↓
Google Sheets updated automatically
       ↓
Status changes to "Completed"
       ↓
No reminder email is sent
```

### Files Involved

1. **[campaign-complete.html](campaign-complete.html)** - User-facing completion form
2. **[salaam-masjid-workflow-complete.json](salaam-masjid-workflow-complete.json)** - Complete n8n workflow with tracking

### Setup Steps

#### Step 1: Upload campaign-complete.html

Upload the file to your website, e.g.:
- `https://salaammasjid.org.uk/campaign-complete.html`
- Or any hosting service (Netlify, GitHub Pages, etc.)

#### Step 2: Import the Complete Workflow

1. Import `salaam-masjid-workflow-complete.json` into n8n
2. This workflow has **TWO webhooks**:
   - `/salaam-masjid-campaign` - For initial registration
   - `/campaign-completed` - For marking campaigns complete

#### Step 3: Configure Workflow

1. **Click "Campaign Completed Webhook" node**
2. Copy the Production URL (e.g., `https://mega.srv967585.hstgr.cloud/webhook/campaign-completed`)
3. **Update campaign-complete.html**:
   - Line 310: Change `WEBHOOK_URL` to your campaign-completed webhook URL

4. **Update Google Sheets nodes** (4 total):
   - Replace `YOUR_GOOGLE_SHEET_ID_HERE` with your actual Sheet ID
   - Ensure all use the "Pledges" sheet

#### Step 4: Update Google Sheets Structure

Add one more column to your "Pledges" sheet:

| Column | Header Name |
|--------|-------------|
| A | ID |
| B | Timestamp |
| C | Full Name |
| D | Email |
| E | Phone |
| F | Goal GBP |
| G | Status |
| H | Campaign URL |
| **I** | **Completed At** ← NEW!

#### Step 5: Add Link to Reminder Email

The reminder email already includes a link to the completion form:

```html
Already completed it? Click here to update your status.
```

**Update the URL** in the "Send Reminder Email" node to match your campaign-complete.html location:
- Replace: `https://your-website.com/campaign-complete.html?email={{ $json['Email'] }}`
- With: `https://salaammasjid.org.uk/campaign-complete.html?email={{ $json['Email'] }}`

The `?email=` parameter pre-fills their email address!

### User Journey with Self-Reporting

```
Day 1 - 10:00 AM
├─ User submits registration form
├─ Redirected to LaunchGood to create campaign
├─ User creates their campaign on LaunchGood
├─ User clicks "Mark Campaign Complete" link (sent in email or on website)
├─ User enters email + campaign URL
├─ Status automatically updated to "Completed" in Google Sheets
└─ Campaign URL saved

Day 2 - 10:00 AM
├─ Workflow wakes up
├─ Checks status in Google Sheets
├─ Status = "Completed" ✅
└─ No reminder sent!
```

### Advantages ✅

- **Automatic** - No manual admin work required
- **Immediate** - Status updated as soon as user completes form
- **User-friendly** - Simple 2-field form
- **Captures campaign URL** - Saves their LaunchGood link
- **Pre-filled email** - Email parameter makes it even easier

### Disadvantages ❌

- Relies on user self-reporting
- User might forget to mark it complete
- Extra step for users

---

## Method 2: Manual Admin Update

### How It Works

```
User creates campaign on LaunchGood
       ↓
User emails/calls admin with campaign URL
       ↓
Admin opens Google Sheets
       ↓
Admin finds user's row by email
       ↓
Admin changes Status from "Pending" to "Completed"
       ↓
Admin adds Campaign URL (optional)
       ↓
No reminder email is sent
```

### Setup Steps

1. Use the original workflow: `salaam-masjid-workflow-improved.json`
2. When user notifies you they created campaign:
   - Open Google Sheets
   - Find their row (search by email or name)
   - Change **Status** column from "Pending" to "Completed"
   - Add their **Campaign URL** in column H (optional)

### Advantages ✅

- **Simple** - No additional setup required
- **Reliable** - Admin has full control
- **No extra user steps** - User just needs to notify you

### Disadvantages ❌

- **Manual work** - Admin must update every entry
- **Delayed** - Depends on admin availability
- **Scalability** - Difficult with many registrations

---

## Method 3: LaunchGood Webhook (Advanced)

### How It Works

```
User creates campaign on LaunchGood
       ↓
LaunchGood sends webhook notification to n8n
       ↓
n8n receives campaign creation event
       ↓
Status automatically updated to "Completed"
       ↓
No reminder email is sent
```

### Requirements

- LaunchGood API access
- LaunchGood webhook configuration permissions
- LaunchGood campaign creation webhooks support

### Setup Steps (If LaunchGood Supports Webhooks)

1. **Contact LaunchGood Support**
   - Ask if they provide webhooks for campaign creation events
   - Request API documentation

2. **Configure LaunchGood Webhook**
   - Set webhook URL to: `https://your-n8n.com/webhook/launchgood-campaign-created`
   - Subscribe to "campaign.created" event
   - Map your community ID

3. **Add Webhook Node to n8n**
   ```json
   {
     "name": "LaunchGood Campaign Created",
     "type": "webhook",
     "path": "launchgood-campaign-created"
   }
   ```

4. **Process LaunchGood Event**
   - Extract user email from webhook payload
   - Match email to Google Sheets row
   - Update status to "Completed"

### Advantages ✅

- **Fully automatic** - Zero manual work
- **Real-time** - Instant updates
- **Scalable** - Handles unlimited registrations
- **Most accurate** - Direct from LaunchGood

### Disadvantages ❌

- **Requires LaunchGood API** - May not be available
- **Complex setup** - More technical configuration
- **Dependent on LaunchGood** - Relies on their webhook reliability

### Note

As of my knowledge, LaunchGood may not provide public webhooks for campaign creation events. You would need to:
1. Contact LaunchGood support
2. Ask if they have API/webhook capabilities
3. Request access if available

---

## 📊 Comparison Table

| Feature | Self-Reporting Form | Manual Admin Update | LaunchGood Webhook |
|---------|---------------------|---------------------|---------------------|
| **Setup Complexity** | Medium | Easy | Hard |
| **User Effort** | Low (1 form) | None (just notify) | None |
| **Admin Effort** | None | High (manual updates) | None |
| **Real-time Updates** | Yes | No (depends on admin) | Yes |
| **Scalability** | Excellent | Poor | Excellent |
| **Reliability** | Good (user-dependent) | Good (admin-dependent) | Excellent |
| **Campaign URL Capture** | Yes | Optional | Yes |
| **Recommended For** | Most situations | Small scale (<10) | Large scale (100+) |

---

## 🎯 Recommended Approach

### For Most Organizations: Hybrid Approach

Combine **Method 1 (Self-Reporting)** + **Method 2 (Manual Admin)**:

1. **Primary**: Encourage users to use self-reporting form
   - Include link in reminder emails
   - Add button on your website
   - Mention in initial confirmation

2. **Backup**: Allow manual admin updates
   - For users who have issues with form
   - For older registrations
   - For admin verification

### Implementation Checklist

- [ ] Deploy campaign-complete.html to website
- [ ] Import salaam-masjid-workflow-complete.json to n8n
- [ ] Configure both webhook URLs
- [ ] Update Google Sheets column headers (add "Completed At")
- [ ] Update reminder email with completion form link
- [ ] Test both self-reporting and manual update methods
- [ ] Train team on Google Sheets manual updates
- [ ] Add completion form link to website

---

## 🔄 Updated Workflow Visualization

```
┌─────────────────────────────────────────────────────────────┐
│              REGISTRATION FLOW                              │
└─────────────────────────────────────────────────────────────┘

User fills form → n8n webhook → Save to Sheets (Pending)
                                        ↓
                    Redirect to LaunchGood to create campaign
                                        ↓
                              User creates campaign
                                        ↓
                        ┌───────────────┴────────────────┐
                        │                                │
               OPTION A: Self-Report        OPTION B: Notify Admin
                        │                                │
                        ↓                                ↓
            Fill completion form              Admin manually updates
                        │                       Google Sheets
                        ↓                                │
            n8n webhook receives                         │
            completion data                              │
                        │                                │
                        └───────────┬────────────────────┘
                                    ↓
                    Status updated to "Completed"
                    Campaign URL saved
                                    ↓
                              Wait 24 hours
                                    ↓
                            Check Status
                                    ↓
                    IF Status = "Completed" → No reminder
                    IF Status = "Pending" → Send reminder
```

---

## 🚀 Quick Start Guide

**Want to get started quickly?**

1. **Use salaam-masjid-workflow-complete.json** (has both webhooks)
2. **Deploy campaign-complete.html** to your website
3. **Update these 3 places**:
   - campaign-form.html: webhook URL for registration
   - campaign-complete.html: webhook URL for completion
   - Reminder email: completion form link
4. **Add "Completed At" column** to Google Sheets
5. **Test with yourself**:
   - Submit registration
   - Get redirected to LaunchGood
   - Go to completion form
   - Submit with your email + any URL
   - Check Google Sheets - status should be "Completed"

---

## 📧 Email Flow with Tracking

### Registration Email (Optional - add if you want)
```
Subject: Welcome! Create your Salaam Masjid Campaign

Body:
- Thank you for registering
- Click here to create your campaign (LaunchGood link)
- After creating it, mark it complete here (completion form link)
```

### Reminder Email (After 24 hours if status = "Pending")
```
Subject: Did you launch your campaign?

Body:
- Gentle reminder about creating campaign
- Your goal: £XXX
- Click to create campaign (LaunchGood link)
- Already created? Click here to update (completion form link)
```

---

## 💡 Tips & Best Practices

### Increase Self-Reporting Completion Rate

1. **Add completion form link everywhere**:
   - Registration confirmation page
   - Website header/footer
   - Social media posts
   - WhatsApp group messages

2. **Make it easy**:
   - Pre-fill email via URL parameter
   - Simple 2-field form
   - Clear instructions

3. **Incentivize**:
   - "Mark your campaign complete to be featured on our website"
   - "Complete this to track your progress"

4. **Follow up**:
   - Phone call to users who haven't marked complete
   - WhatsApp message with direct link

### Google Sheets Tips

1. **Use conditional formatting**:
   - Green: Status = "Completed"
   - Yellow: Status = "Pending"
   - Orange: Status = "Reminded"

2. **Add formula columns**:
   - Time since registration: `=NOW()-B2`
   - Days until reminder: `=24-(NOW()-B2)*24`

3. **Create dashboard**:
   - Total registrations: `=COUNTA(A:A)-1`
   - Completed: `=COUNTIF(G:G,"Completed")`
   - Pending: `=COUNTIF(G:G,"Pending")`
   - Conversion rate: `=Completed/Total`

---

## 🛠️ Troubleshooting

### User says they marked it complete but still got reminder

1. Check Google Sheets - is status "Completed"?
2. Check "Completed At" timestamp
3. Verify email address matches exactly (case-insensitive)
4. Check n8n execution history for completion webhook

### Completion form not working

1. Check browser console for errors
2. Verify webhook URL is correct
3. Test webhook directly with curl/Postman
4. Check n8n execution history

### Status not updating

1. Verify Google Sheets credentials in n8n
2. Check email matching logic (lowercase comparison)
3. Ensure "Completed At" column exists
4. Review n8n execution log

---

**Your tracking system is now complete! 🎉**

Users can self-report campaign creation, and you have full control with manual updates as backup.
