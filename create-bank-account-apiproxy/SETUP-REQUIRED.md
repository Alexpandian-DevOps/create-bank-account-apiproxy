# ⚠️ SETUP REQUIRED BEFORE DEPLOYMENT

## Current Status: Workflows are DISABLED for automatic deployment

The GitHub Actions workflows are configured for **manual trigger only** until you complete the GCP/Apigee setup.

## ✅ Before You Can Deploy:

### 1. Setup GCP Trial Account
- [ ] Activate GCP trial at https://console.cloud.google.com
- [ ] Enable Apigee API
- [ ] Provision Apigee environment (takes 30-45 minutes)
- [ ] Note your GCP Project ID

### 2. Create Service Account
- [ ] Go to IAM & Admin → Service Accounts
- [ ] Create service account: `apigee-github-deploy`
- [ ] Grant roles: Apigee API Admin, Apigee Environment Admin
- [ ] Create and download JSON key

### 3. Configure GitHub Secrets
- [ ] Go to GitHub repo → Settings → Secrets and variables → Actions
- [ ] Add secret: `GCP_SERVICE_ACCOUNT_KEY` (paste entire JSON content)
- [ ] Add secret: `APIGEE_ORG` (your GCP project ID)
- [ ] Add secret: `APIGEE_ENV` (usually `eval` or `test`)

## 🚀 How to Deploy (After Setup):

### Option 1: Manual Deployment via GitHub Actions UI

1. Go to your GitHub repository
2. Click **Actions** tab
3. Select workflow from left sidebar:
   - **Setup KVM** (run this first)
   - **Deploy Shared Flow** (run this second)
   - **Deploy Apigee Proxy** (run this last)
4. Click **Run workflow** button
5. Choose environment (eval/test/prod)
6. Click **Run workflow**

### Option 2: Enable Automatic Deployment (After Testing)

Once everything is working, you can enable auto-deployment by uncommenting the workflows:

Edit `.github/workflows/deploy-apigee.yml`:
```yaml
on:
  push:              # Uncomment these lines
    branches:        # to enable auto-deploy
      - main         # on push to main/develop
      - develop
  workflow_dispatch:
```

## 📋 Deployment Order:

```
1. Setup KVM          (First - only once)
2. Deploy Shared Flow (Second - reusable across APIs)
3. Deploy API Proxy   (Third - your API)
```

## ⚠️ Why Workflows Failed:

The workflows tried to deploy automatically when you pushed code, but failed because:
- ❌ GCP service account not configured
- ❌ GitHub secrets not set
- ❌ Apigee environment not provisioned

This is **normal** and **expected** until you complete the setup!

## 📚 Complete Setup Instructions:

See these guides for detailed setup:
- `COMPLETE-DEPLOYMENT-GUIDE.md` - Full step-by-step guide
- `QUICK-DEPLOY.md` - Quick reference

## 🔧 Current Workflow Configuration:

All workflows are set to **manual trigger only**:
- ✅ Won't run automatically on push
- ✅ Must be triggered manually from Actions tab
- ✅ Safe to push code without deployments failing

## ✨ Next Steps:

1. Complete GCP/Apigee setup (see COMPLETE-DEPLOYMENT-GUIDE.md)
2. Configure GitHub secrets
3. Manually trigger workflows from Actions tab
4. Test your API
5. (Optional) Enable auto-deployment later

---

**Note**: You can safely push code to GitHub now. Workflows won't run automatically until you manually trigger them!
