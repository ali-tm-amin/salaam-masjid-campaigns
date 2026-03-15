# Netlify Deployment Guide

Complete guide to deploy the Salaam Masjid Campaign System to Netlify.

## 📋 Prerequisites

- GitHub account
- Netlify account (free tier works perfectly)
- n8n instance running (for webhook endpoints)

---

## 🚀 Deployment Steps

### Step 1: Push to GitHub

1. **Create a new repository on GitHub**:
   - Go to https://github.com/new
   - Repository name: `salaam-masjid-campaigns`
   - Visibility: Public or Private (your choice)
   - DO NOT initialize with README (we already have one)
   - Click "Create repository"

2. **Add remote and push**:
   ```bash
   cd "/Users/ali/production/Salaam Masjid"
   git remote add origin https://github.com/YOUR_USERNAME/salaam-masjid-campaigns.git
   git branch -M main
   git push -u origin main
   ```

3. **Verify on GitHub**:
   - Refresh your GitHub repository page
   - You should see all files uploaded

### Step 2: Deploy to Netlify

1. **Go to Netlify**:
   - Visit https://app.netlify.com/
   - Sign in or create account

2. **Import from Git**:
   - Click "Add new site" → "Import an existing project"
   - Choose "GitHub"
   - Authorize Netlify (if first time)
   - Select your repository: `salaam-masjid-campaigns`

3. **Configure build settings**:
   ```
   Build command: [leave empty]
   Publish directory: .
   ```
   (Netlify will auto-detect settings from netlify.toml)

4. **Click "Deploy site"**:
   - Netlify will assign a random subdomain like `random-name-123456.netlify.app`
   - Deployment takes ~30 seconds

5. **Wait for deployment**:
   - Watch the deploy log
   - When complete, you'll see "Site is live" ✅

### Step 3: Configure Custom Domain (Optional)

1. **Go to Site settings** → **Domain management**

2. **Add custom domain**:
   - Click "Add custom domain"
   - Enter: `campaigns.salaammasjid.org.uk` (or your preferred subdomain)

3. **Configure DNS** (at your domain registrar):
   ```
   Type: CNAME
   Name: campaigns
   Value: random-name-123456.netlify.app
   ```

4. **Enable HTTPS** (automatic):
   - Netlify provides free SSL via Let's Encrypt
   - Takes ~1 minute to provision

### Step 4: Update Webhook URLs

Your site is now live! Update the webhook URLs in your forms:

1. **Get your Netlify URL**:
   ```
   https://your-site-name.netlify.app
   ```

2. **Forms are already configured**:
   - index.html: Webhook already set to `https://mega.srv967585.hstgr.cloud/webhook/salaam-masjid-campaign`
   - campaign-complete.html: Update webhook URL (line 310)

3. **Update completion form URL in n8n**:
   - In "Send Reminder Email" node
   - Update link to: `https://your-site-name.netlify.app/campaign-complete.html?email={{ $json['Email'] }}`

---

## 🔧 Post-Deployment Configuration

### Environment-Specific Settings

Since n8n webhooks are already configured to your server, no environment variables needed!

### Test Your Deployment

1. **Visit your site**:
   ```
   https://your-site-name.netlify.app
   ```

2. **Test registration form**:
   - Fill in test details
   - Submit
   - Should see success message
   - Should redirect to LaunchGood after 3 seconds

3. **Check n8n**:
   - Check execution history
   - Verify data received

4. **Check Google Sheets**:
   - New row should appear
   - Status should be "Pending"

5. **Test completion form**:
   - Visit: `https://your-site-name.netlify.app/campaign-complete.html`
   - Enter test email + campaign URL
   - Submit
   - Check Google Sheets - status should change to "Completed"

---

## 📊 Netlify Features You're Using

### 1. Automatic Deployments
- Every git push triggers auto-deployment
- See deploy log in Netlify dashboard

### 2. SSL/HTTPS
- Free automatic SSL certificate
- Always encrypted connections

### 3. CDN Distribution
- Global CDN for fast loading
- Auto-caching of static assets

### 4. Custom Headers
- Security headers configured in netlify.toml
- Protection against XSS, clickjacking, etc.

### 5. Branch Previews (Optional)
- Create preview deployments for testing
- Each branch gets its own URL

---

## 🔄 Updating Your Site

### Option 1: Via Git (Recommended)

1. **Make changes locally**
2. **Commit and push**:
   ```bash
   git add .
   git commit -m "Update: description of changes"
   git push
   ```
3. **Auto-deployment**:
   - Netlify detects push
   - Rebuilds and redeploys automatically
   - Takes ~30 seconds

### Option 2: Via Netlify UI

1. Go to Netlify dashboard
2. Click "Deploys" → "Trigger deploy" → "Deploy site"
3. Rebuilds from latest git commit

### Option 3: Manual Upload

1. Go to Netlify dashboard
2. Drag and drop updated files
3. (Not recommended - use git instead)

---

## 🎯 Project Structure

```
salaam-masjid-campaigns/
├── index.html                          ← Main form (deployed)
├── campaign-complete.html              ← Completion form (deployed)
├── netlify.toml                        ← Netlify configuration
├── README.md                           ← Project documentation
├── .gitignore                          ← Git ignore rules
│
├── docs/                               ← Documentation (not deployed, but accessible)
│   ├── SETUP_GUIDE.md
│   ├── CAMPAIGN_TRACKING_GUIDE.md
│   ├── WORKFLOW_DIAGRAM.md
│   └── IMPROVEMENTS.md
│
└── workflows/                          ← n8n workflows (reference only)
    ├── salaam-masjid-workflow-complete.json
    ├── salaam-masjid-workflow-improved.json
    └── Salaam Masjid — LaunchGood P2P Campaign Creator.json
```

**Deployed files**: Only HTML files in root + netlify.toml
**Everything else**: Accessible via GitHub but not served by Netlify

---

## 🛡️ Security Checklist

- [x] HTTPS enabled (automatic via Netlify)
- [x] Security headers configured (netlify.toml)
- [x] No sensitive data in repository
- [x] Webhook URLs use HTTPS
- [x] Google Sheets access properly secured
- [x] Email credentials stored securely in n8n

---

## 📈 Monitoring & Analytics

### Netlify Analytics (Optional)

1. Go to Site settings → Analytics
2. Enable Netlify Analytics ($9/month)
3. Track page views, bandwidth, top pages

### Google Analytics (Optional)

Add to your HTML files before `</head>`:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

---

## 🔍 Troubleshooting

### Forms not submitting

**Check**:
- Webhook URLs are correct
- n8n is running and accessible
- Check browser console for errors
- Verify CORS settings in n8n webhook nodes

### Site not updating after push

**Solutions**:
- Check Netlify deploy log for errors
- Trigger manual deploy in Netlify
- Clear browser cache
- Check git push was successful

### SSL certificate issues

**Solutions**:
- Wait 5-10 minutes for provisioning
- Check DNS settings are correct
- Contact Netlify support if persists

### Custom domain not working

**Check**:
- DNS records properly configured
- Allow 24-48 hours for DNS propagation
- Use https:// not http://

---

## 💡 Pro Tips

### 1. Use Deploy Previews

Create a `dev` branch for testing:
```bash
git checkout -b dev
# Make changes
git push origin dev
```
Netlify creates preview URL for testing before merging to main.

### 2. Set Up Notifications

- Go to Site settings → Build & deploy → Deploy notifications
- Add Slack/email notifications for deploys
- Get notified of successful/failed deployments

### 3. Optimize Performance

- Images: Already using CDN
- HTML: Minified on deploy
- Caching: Configured in netlify.toml

### 4. Rollback if Needed

- Go to Deploys
- Find previous successful deploy
- Click "Publish deploy"
- Instantly rollback to working version

---

## 🎉 You're Live!

Your campaign forms are now deployed and accessible worldwide via Netlify's CDN!

**Next steps**:
1. Share the URL with your community
2. Monitor registrations in Google Sheets
3. Check n8n execution logs regularly
4. Update reminder email with your actual domain

**Your URLs**:
- Registration: `https://your-site.netlify.app/`
- Completion: `https://your-site.netlify.app/campaign-complete.html`

---

## 📞 Support Resources

- **Netlify Docs**: https://docs.netlify.com/
- **Netlify Support**: https://www.netlify.com/support/
- **Project README**: [README.md](README.md)
- **Setup Guide**: [docs/SETUP_GUIDE.md](docs/SETUP_GUIDE.md)
- **Tracking Guide**: [docs/CAMPAIGN_TRACKING_GUIDE.md](docs/CAMPAIGN_TRACKING_GUIDE.md)

---

**May Allah bless your deployment! 🤲**
