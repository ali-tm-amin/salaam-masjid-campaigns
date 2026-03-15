# Salaam Masjid - LaunchGood P2P Campaign Creator

Complete automated system for managing LaunchGood peer-to-peer fundraising campaigns with intelligent tracking and reminders.

## 📋 System Overview

This system automates the entire campaign creation workflow:

1. ✅ **User Registration** - Collects name, email, phone, fundraising goal
2. ✅ **Auto-Redirect** - Sends users to LaunchGood to create campaign
3. ✅ **Campaign Tracking** - Knows when users complete their campaigns
4. ✅ **Smart Reminders** - Only sends reminders to users who haven't completed
5. ✅ **Google Sheets Logging** - Tracks all registrations and status

## 🎯 Key Features

- **Form redirects to LaunchGood** after submission ← NEW!
- **Self-reporting completion form** for users ← NEW!
- **Automatic status tracking** with webhook integration ← NEW!
- **Conditional reminder system** - no duplicate reminders
- **Manual admin override** - full control via Google Sheets
- **Phone number collection** for follow-up
- **Beautiful branded emails** with Salaam Masjid colors
- **Campaign URL tracking** in Google Sheets

## 📁 Files Included

### Core Files

| File | Purpose |
|------|---------|
| **index.html** | Main registration form (redirects to LaunchGood) |
| **campaign-complete.html** | Self-reporting completion form (NEW!) |
| **salaam-masjid-workflow-complete.json** | Complete n8n workflow with tracking (NEW!) |
| **salaam-masjid-workflow-improved.json** | Basic workflow (manual tracking only) |

### Documentation

| File | Purpose |
|------|---------|
| **README.md** | This file - quick overview |
| **SETUP_GUIDE.md** | Step-by-step setup instructions |
| **CAMPAIGN_TRACKING_GUIDE.md** | Complete guide on how tracking works (NEW!) |
| **WORKFLOW_DIAGRAM.md** | Visual workflow diagrams |

### Archive

| File | Purpose |
|------|---------|
| **Salaam Masjid — LaunchGood P2P Campaign Creator.json** | Original workflow (reference) |

## 🚀 Quick Start

### Option 1: Full System with Tracking (Recommended)

1. **Deploy Forms**:
   - Upload `index.html` to your website root directory
   - Upload `campaign-complete.html` to your website

2. **Import Workflow**:
   - Import `salaam-masjid-workflow-complete.json` to n8n
   - Configure both webhook URLs
   - Update Google Sheet ID (4 places)

3. **Update Google Sheets**:
   - Add column headers (see setup guide)
   - Add "Completed At" column (column I)

4. **Configure URLs**:
   - Update webhook URL in index.html (already configured)
   - Update webhook URL in campaign-complete.html
   - Update completion form URL in reminder email

5. **Test**:
   - Submit test registration
   - Use completion form to mark as complete
   - Verify status updates in Google Sheets

📖 **Full guide**: See [CAMPAIGN_TRACKING_GUIDE.md](CAMPAIGN_TRACKING_GUIDE.md)

### Option 2: Basic System (Manual Tracking)

1. **Deploy Form**:
   - Upload `index.html` to your website root directory

2. **Import Workflow**:
   - Import `salaam-masjid-workflow-improved.json` to n8n
   - Configure webhook URL
   - Update Google Sheet ID (4 places)

3. **Manual Tracking**:
   - When users create campaigns, manually update status in Google Sheets

📖 **Full guide**: See [SETUP_GUIDE.md](SETUP_GUIDE.md)

## ❓ FAQs

### How does it redirect to LaunchGood?

After form submission, users see a 3-second success message, then automatically redirect to:
```
https://www.launchgood.com/create#!/create/new/basics?raising_for=community&community_id=21505
```

### How does the system know when campaigns are created?

**Two methods**:

1. **Self-Reporting (Automatic)**: User fills completion form → Status updates automatically
2. **Manual Update (Backup)**: Admin changes status in Google Sheets

See [CAMPAIGN_TRACKING_GUIDE.md](CAMPAIGN_TRACKING_GUIDE.md) for detailed explanation.

### What prevents duplicate reminders?

The workflow checks Google Sheets status before sending reminders:
- Status = "Pending" → Send reminder
- Status = "Completed" → No reminder
- Status = "Reminded" → Already sent

### Can I customize the emails?

Yes! Edit the "Send Reminder Email" node in n8n:
- Update text content
- Change colors (search for `#0d5c36` and `#22b068`)
- Modify layout
- Add your logo

### What about GDPR/data privacy?

- Secure Google Sheets access (configure permissions)
- Use HTTPS for all forms
- Add privacy policy to forms
- Only collect necessary data
- Allow users to request deletion

## 📊 Google Sheets Structure

| Column | Header | Description |
|--------|--------|-------------|
| A | ID | Unique tracking ID (auto-generated) |
| B | Timestamp | Registration date/time |
| C | Full Name | User's full name |
| D | Email | Contact email |
| E | Phone | Contact phone number |
| F | Goal GBP | Fundraising target amount |
| G | Status | Pending / Completed / Reminded |
| H | Campaign URL | LaunchGood campaign link |
| I | Completed At | When campaign was marked complete |

## 🔄 Workflow Logic

```
Form Submission
    ↓
Save to Google Sheets (Status: Pending)
    ↓
Redirect to LaunchGood
    ↓
[User creates campaign]
    ↓
User marks complete via form
    ↓
Status → "Completed"
    ↓
Wait 24 Hours
    ↓
Check Status
    ↓
IF "Pending" → Send Reminder
IF "Completed" → No Action
```

## 🎨 Customization

### Brand Colors

Current: Green theme (#0d5c36, #22b068)

To change:
1. **index.html**: Update CSS (lines 20-30)
2. **campaign-complete.html**: Update CSS (lines 20-30)
3. **Email template**: Update inline styles in n8n node

### Minimum Goal Amount

Current: £100

To change:
1. **campaign-form.html**: Line 279 (`min="100"`)
2. **campaign-form.html**: Line 340 (validation message)

### Wait Time

Current: 24 hours

To change:
1. **n8n workflow**: "Wait 24 Hours" node
2. Change `amount` parameter (1-999)
3. Change `unit` (minutes, hours, days)

## 📞 Support & Troubleshooting

### Common Issues

**Form doesn't submit**: Check webhook URL, verify n8n is active

**Status not updating**: Check Google Sheets credentials, verify email matching

**Reminders still sent**: Ensure status is exactly "Completed" (case-sensitive)

**Redirect not working**: Clear browser cache, check JavaScript console

### Testing Checklist

- [ ] Form submission saves to Google Sheets
- [ ] Form redirects to LaunchGood after submission
- [ ] Completion form updates status to "Completed"
- [ ] Completion form saves campaign URL
- [ ] Wait node pauses for correct duration
- [ ] Status check reads from Google Sheets correctly
- [ ] Reminder only sent when status = "Pending"
- [ ] Status updates to "Reminded" after email sent

## 🤝 Contributing

To improve this system:

1. **Test thoroughly** before deploying changes
2. **Update documentation** when adding features
3. **Keep backups** of working configurations
4. **Version control** your customizations

## 📄 License

This project is created for Salaam Masjid Kempston.

## 🤲 Du'a

May Allah accept your efforts and bless the Salaam Masjid extension project.

---

**Quick Links**:
- [Setup Guide](SETUP_GUIDE.md) - Step-by-step installation
- [Campaign Tracking Guide](CAMPAIGN_TRACKING_GUIDE.md) - How tracking works
- [Workflow Diagram](WORKFLOW_DIAGRAM.md) - Visual workflow documentation

**Need help?** Review the troubleshooting sections in each guide.
