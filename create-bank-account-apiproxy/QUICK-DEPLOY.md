# Quick Start - Deploy to GCP Trial Account

## ⚡ Super Quick Setup (30 Minutes)

### 1. GCP Setup (10 min)
```
1. Go to console.cloud.google.com
2. Activate $300 trial
3. Enable Apigee API
4. Provision Apigee (wait 30-45 min while you do other steps)
```

### 2. Create Service Account (5 min)
```
1. IAM & Admin → Service Accounts
2. Create: apigee-github-deploy
3. Roles: Apigee API Admin, Apigee Environment Admin
4. Create JSON key → Download
```

### 3. GitHub Setup (5 min)
```powershell
# In PowerShell
cd C:\Users\AS001028268\Downloads\create-bank-account-apiproxy
git init
git add .
git commit -m "Initial commit"
git branch -M main

# Create repo on github.com, then:
git remote add origin https://github.com/YOUR-USERNAME/create-bank-account-apiproxy.git
git push -u origin main
```

### 4. GitHub Secrets (3 min)
```
Settings → Secrets → Actions → New secret
1. GCP_SERVICE_ACCOUNT_KEY = [paste entire JSON file content]
2. APIGEE_ORG = [your GCP project ID]
3. APIGEE_ENV = eval
```

### 5. Deploy via GitHub Actions (5 min)
```
Actions tab → Run workflows in order:
1. Setup KVM (choose 'eval' environment)
2. Deploy Shared Flow (choose 'eval')
3. Deploy Apigee Proxy (choose 'eval')
```

### 6. Test (2 min)
```
URL: https://YOUR-PROJECT-ID-eval.apigee.net/v1/banking/accounts

Postman → Import collection → Update base_url → Send request
```

## 🎯 Your API Endpoint

```
https://[YOUR-GCP-PROJECT-ID]-eval.apigee.net/v1/banking/accounts
```

## 🔍 Find Your Project ID

GCP Console → Top navigation bar (shows project name and ID)

## ✅ Verify Success

- GitHub Actions shows green checkmarks ✅
- Apigee Console shows deployed proxy
- Postman returns 201 Created

## 🆘 Quick Fixes

**Can't find project ID?**
→ GCP Console, top bar

**GitHub Actions failing?**
→ Check all 3 secrets are set correctly

**404 error?**
→ Wait for Apigee provisioning to complete

**401 error?**
→ Comment out FlowCallout step in proxy, redeploy

See `COMPLETE-DEPLOYMENT-GUIDE.md` for detailed instructions!
