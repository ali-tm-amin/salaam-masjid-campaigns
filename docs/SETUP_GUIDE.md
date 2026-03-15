# Salaam Masjid Campaign Workflow - Setup Guide

This guide will help you set up the complete LaunchGood P2P Campaign Creator system with n8n automation.

## 📋 Overview

The system consists of:
1. **HTML Form** - Collects user details (name, email, phone, fundraising goal)
2. **n8n Workflow** - Automates the entire process
3. **Google Sheets** - Stores campaign registrations
4. **Email Reminders** - Automated follow-up system

## 🎯 Workflow Logic

```
User Submits Form
    ↓
Save to Google Sheets (Status: Pending)
    ↓
Wait 24 Hours
    ↓
Check Campaign Status in Google Sheets
    ↓
IF Status = "Pending" (campaign not created)
    ↓
Send Reminder Email
    ↓
Update Status to "Reminded"
```

### How Users Mark Campaign as Complete

When a user creates their campaign, they (or an admin) should:
1. Open the Google Sheet
2. Find their row by email/name
3. Update the **Status** column from "Pending" to "Completed"
4. Optionally add their Campaign URL

This prevents the reminder email from being sent.

---

## 🚀 Setup Instructions

### Step 1: Prepare Google Sheets

1. **Create a new Google Sheet** or use existing one
2. **Create a sheet tab named** "Pledges"
3. **Add the following column headers** in Row 1:
   - A: ID
   - B: Timestamp
   - C: Full Name
   - D: Email
   - E: Phone
   - F: Goal GBP
   - G: Status
   - H: Campaign URL

4. **Copy your Google Sheet ID**
   - From the URL: `https://docs.google.com/spreadsheets/d/[SHEET_ID]/edit`
   - Save this ID for later

### Step 2: Set Up n8n Workflow

1. **Open n8n** and create a new workflow

2. **Import the workflow**:
   - Click the 3-dot menu → Import from File
   - Select `salaam-masjid-workflow-improved.json`

3. **Configure Google Sheets nodes**:

   The workflow has 4 Google Sheets nodes that need configuration:

   a. **Log to Google Sheets** node:
      - Click the node
      - Set Document ID to your Google Sheet ID
      - Ensure Sheet Name is "Pledges"
      - Connect your Google Sheets account if not already connected

   b. **Check Campaign Status** node:
      - Click the node
      - Set Document ID to your Google Sheet ID
      - Ensure Sheet Name is "Pledges"

   c. **Update Status to Reminded** node:
      - Click the node
      - Set Document ID to your Google Sheet ID
      - Ensure Sheet Name is "Pledges"

4. **Configure Email node**:
   - Click "Send Reminder Email" node
   - Update `fromEmail` to your actual sender email
   - Update `replyTo` to your admin email
   - Configure your email credentials (SMTP or email service)

5. **Get the Webhook URL**:
   - Click on "Receive Form Data" node
   - Copy the "Production URL" (it will look like: `https://your-n8n-instance.com/webhook/salaam-masjid-campaign`)
   - Save this URL

6. **Save and Activate**:
   - Click "Save" button
   - Toggle the workflow to "Active"

### Step 3: Configure HTML Form

1. **Open** `campaign-form.html` in a text editor

2. **Update the Webhook URL**:
   - Find line with: `const WEBHOOK_URL = 'YOUR_N8N_WEBHOOK_URL_HERE';`
   - Replace with your actual webhook URL from Step 2.5
   - Example: `const WEBHOOK_URL = 'https://your-n8n-instance.com/webhook/salaam-masjid-campaign';`

3. **Optional customizations**:
   - Update colors in the CSS (search for `#0d5c36` and `#22b068` to change brand colors)
   - Modify text content as needed
   - Change minimum goal amount (currently £100)

4. **Deploy the form**:
   - Upload to your web hosting
   - Or embed in your existing website
   - Ensure it's accessible via HTTPS for security

### Step 4: Test the Workflow

1. **Submit a test registration**:
   - Open the HTML form
   - Fill in test details:
     - Name: Test User
     - Email: your-email@example.com
     - Phone: +44 7XXX XXXXXX
     - Goal: 500

2. **Verify in Google Sheets**:
   - Check that a new row appears with Status "Pending"
   - Verify all data is captured correctly

3. **Test the 24-hour wait** (optional):
   - In n8n, you can manually execute nodes
   - Or temporarily change wait time to 1 minute for testing
   - Check if reminder email is sent when status is "Pending"

4. **Test the conditional logic**:
   - Change Status to "Completed" in Google Sheets
   - Trigger the workflow again
   - Verify NO reminder email is sent

---

## 🔄 Managing Campaigns

### When Someone Creates Their Campaign

**Option A: User Self-Service** (Recommended)
- Create a simple form/button on your website
- Allow users to submit their LaunchGood campaign URL
- Use another n8n webhook to update the Google Sheet status

**Option B: Manual Update**
- Admin opens Google Sheet
- Finds the user's row
- Changes Status from "Pending" to "Completed"
- Adds their Campaign URL in column H

### Monitoring

**Google Sheets columns explained**:
- **ID**: Unique tracking ID (auto-generated)
- **Timestamp**: When they registered
- **Full Name**: User's full name
- **Email**: Contact email
- **Phone**: Contact phone
- **Goal GBP**: Their fundraising target
- **Status**: Current status
  - `Pending` = Waiting for campaign creation
  - `Completed` = Campaign created (no reminder sent)
  - `Reminded` = Reminder email was sent
- **Campaign URL**: Their LaunchGood campaign link (optional)

---

## 📧 Email Customization

The reminder email template is embedded in the workflow. To customize:

1. Open the n8n workflow
2. Click "Send Reminder Email" node
3. Click "HTML" field
4. Modify the HTML template
5. Available variables:
   - `{{ $json['Full Name'] }}` - User's full name
   - `{{ $json['Email'] }}` - User's email
   - `{{ $json['Goal GBP'] }}` - Their goal amount
   - `{{ $json['Full Name'].split(' ')[0] }}` - First name

---

## 🛠️ Troubleshooting

### Form doesn't submit
- Check browser console for errors
- Verify webhook URL is correct and accessible
- Ensure n8n workflow is Active
- Check CORS settings in webhook node

### Data not appearing in Google Sheets
- Verify Google Sheets credentials in n8n
- Check sheet name is exactly "Pledges"
- Confirm column headers match exactly
- Check n8n execution log for errors

### Reminder emails not sending
- Verify email credentials in n8n
- Check spam folder
- Review n8n execution history
- Ensure Status is "Pending" in Google Sheets

### Reminders sent even after campaign created
- Ensure Status is updated to "Completed" (not "Complete" or other variations)
- Check the IF node conditions
- Review execution log in n8n

---

## 🎨 Customization Ideas

### Extend the Workflow

1. **Send confirmation email immediately**
   - Add email node after "Log to Google Sheets"
   - Send welcome email with instructions

2. **Multiple reminders**
   - Add another Wait node (48 hours)
   - Check status again
   - Send second reminder if still pending

3. **Slack/Discord notifications**
   - Add webhook nodes
   - Notify admin channel when new registration
   - Alert when campaigns are created

4. **Analytics tracking**
   - Add HTTP request node
   - Send events to Google Analytics
   - Track conversion rates

### Form Enhancements

1. **Add validation**
   - Email format validation
   - Phone number formatting
   - Minimum/maximum goal amounts

2. **Multi-step form**
   - Break into multiple pages
   - Add progress indicator
   - Collect additional details

3. **Integration with existing site**
   - Match your website's design
   - Use your CSS framework
   - Add to existing pages

---

## 📊 Monitoring Success

Track these metrics in Google Sheets:
- **Total Registrations**: Count of all rows
- **Pending Campaigns**: Count where Status = "Pending"
- **Completed Campaigns**: Count where Status = "Completed"
- **Reminder Rate**: Count where Status = "Reminded"
- **Conversion Rate**: Completed / Total Registrations

You can create a dashboard in Google Sheets using formulas:
```
=COUNTIF(G:G,"Pending")
=COUNTIF(G:G,"Completed")
=COUNTIF(G:G,"Reminded")
```

---

## 🔐 Security Considerations

1. **Webhook Security**:
   - Use HTTPS for form hosting
   - Consider adding authentication to webhook
   - Implement rate limiting if needed

2. **Data Privacy**:
   - Comply with GDPR/data protection laws
   - Add privacy policy to form
   - Secure access to Google Sheets

3. **Email Security**:
   - Use reputable email service
   - Configure SPF/DKIM records
   - Monitor for spam complaints

---

## 📞 Support

If you encounter issues:
1. Check n8n execution history for errors
2. Review Google Sheets for data accuracy
3. Test each node individually in n8n
4. Check email logs and delivery status

---

## 📝 Files Included

1. **campaign-form.html** - User-facing registration form
2. **salaam-masjid-workflow-improved.json** - Complete n8n workflow
3. **SETUP_GUIDE.md** - This setup guide
4. **Salaam Masjid — LaunchGood P2P Campaign Creator.json** - Original workflow (for reference)

---

## ✅ Next Steps

After setup:
1. ✅ Test with real data
2. ✅ Train team on status updates
3. ✅ Monitor first few registrations
4. ✅ Set up analytics tracking
5. ✅ Create admin dashboard (optional)
6. ✅ Document your specific processes

---

**May Allah bless your fundraising efforts! 🤲**

*Created for Salaam Masjid Kempston*
