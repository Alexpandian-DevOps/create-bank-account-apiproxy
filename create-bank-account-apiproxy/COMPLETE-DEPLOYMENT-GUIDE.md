# Complete Deployment Guide - GCP Trial Account + GitHub + Apigee

This guide walks you through deploying the Create Bank Account API to Apigee using your GCP trial account via GitHub Actions pipeline.

## 📋 Prerequisites Checklist

- ✅ GCP Trial Account (with Apigee provisioned)
- ✅ GitHub Account
- ✅ Git installed on your computer
- ✅ Access to GCP Console

## 🎯 Complete Deployment Flow

```
Local Files → GitHub Repository → GitHub Actions → GCP/Apigee → Test in Postman
```

---

## Step 1️⃣: Setup GCP Trial Account with Apigee

### 1.1 Activate GCP Trial Account

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Sign in with your Google account
3. Click **Activate** $300 free trial
4. Complete billing information (credit card required but won't be charged)
5. Accept terms and activate

### 1.2 Enable Apigee API

1. In GCP Console, search for **Apigee** in the search bar
2. Click **Apigee** or go to **APIs & Services** → **Library**
3. Search for "Apigee API"
4. Click **Enable**

### 1.3 Provision Apigee Organization

1. Go to **Apigee** → **API Management**
2. Click **Provision Apigee**
3. Choose:
   - **Analytics Region**: Select closest to you (e.g., us-central1)
   - **Runtime Location**: Same as analytics region
4. Click **Create**
5. ⏱️ Wait 30-45 minutes for provisioning

### 1.4 Note Your Apigee Details

After provisioning, note these values:
- **Organization Name**: Usually your GCP project ID
- **Environment Name**: Usually `eval` or `test` (auto-created)
- **API Endpoint**: `https://YOUR-PROJECT-ID-eval.apigee.net`

---

## Step 2️⃣: Create Service Account in GCP

### 2.1 Create Service Account

1. In GCP Console, go to **IAM & Admin** → **Service Accounts**
2. Click **+ CREATE SERVICE ACCOUNT**
3. Enter details:
   - **Name**: `apigee-github-deploy`
   - **Description**: `Service account for GitHub Actions to deploy Apigee proxies`
4. Click **CREATE AND CONTINUE**

### 2.2 Grant Permissions

Add these roles:
1. Click **+ ADD ANOTHER ROLE** and add:
   - ✅ **Apigee API Admin** (`roles/apigee.apiAdmin`)
   - ✅ **Apigee Environment Admin** (`roles/apigee.environmentAdmin`)
   - ✅ **Service Account User** (`roles/iam.serviceAccountUser`)
2. Click **CONTINUE** → **DONE**

### 2.3 Create JSON Key

1. Find your new service account in the list
2. Click on it
3. Go to **KEYS** tab
4. Click **ADD KEY** → **Create new key**
5. Choose **JSON**
6. Click **CREATE**
7. 📥 **Save the JSON file** (it will download automatically)
   - ⚠️ **IMPORTANT**: Keep this file secure! Don't share it!
   - File name: `apigee-github-deploy-xxxxx.json`

---

## Step 3️⃣: Create GitHub Repository

### 3.1 Initialize Git Repository Locally

Open PowerShell and run:

```powershell
# Navigate to your project
cd C:\Users\AS001028268\Downloads\create-bank-account-apiproxy

# Initialize git repository
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit - Create Bank Account API with KVM and Shared Flow"

# Set main branch
git branch -M main
```

### 3.2 Create GitHub Repository

1. Go to [GitHub](https://github.com)
2. Click **+** (top right) → **New repository**
3. Enter details:
   - **Repository name**: `create-bank-account-apiproxy`
   - **Description**: `Apigee API Proxy for creating bank accounts`
   - **Visibility**: Private (recommended) or Public
   - ⚠️ **DO NOT** initialize with README
4. Click **Create repository**

### 3.3 Push Code to GitHub

Copy the commands from GitHub and run:

```powershell
# Add GitHub remote (replace YOUR-USERNAME)
git remote add origin https://github.com/YOUR-USERNAME/create-bank-account-apiproxy.git

# Push to GitHub
git push -u origin main
```

---

## Step 4️⃣: Configure GitHub Secrets

### 4.1 Add Service Account Key

1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add secret:
   - **Name**: `GCP_SERVICE_ACCOUNT_KEY`
   - **Value**: Open the JSON key file you downloaded, copy **ALL** content, paste here
5. Click **Add secret**

### 4.2 Add Apigee Organization

1. Click **New repository secret**
2. Add:
   - **Name**: `APIGEE_ORG`
   - **Value**: Your GCP project ID (find in GCP Console top bar)
   - Example: `my-project-12345`
3. Click **Add secret**

### 4.3 Add Apigee Environment

1. Click **New repository secret**
2. Add:
   - **Name**: `APIGEE_ENV`
   - **Value**: `eval` (or `test` - check in Apigee console)
3. Click **Add secret**

### 4.4 Verify Secrets

You should now have 3 secrets:
- ✅ `GCP_SERVICE_ACCOUNT_KEY`
- ✅ `APIGEE_ORG`
- ✅ `APIGEE_ENV`

---

## Step 5️⃣: Deploy Through GitHub Pipeline

### 5.1 Deploy KVM (First Time Only)

1. Go to your GitHub repository
2. Click **Actions** tab
3. Click **Setup KVM** workflow (left sidebar)
4. Click **Run workflow** button
5. Select:
   - **Branch**: `main`
   - **Environment**: `eval` (or your environment name)
6. Click **Run workflow**
7. ⏱️ Wait for completion (green checkmark ✅)

### 5.2 Deploy Shared Flow

1. In **Actions** tab
2. Click **Deploy Shared Flow** workflow
3. Click **Run workflow**
4. Select:
   - **Branch**: `main`
   - **Environment**: `eval`
5. Click **Run workflow**
6. ⏱️ Wait for completion (green checkmark ✅)

### 5.3 Deploy API Proxy

1. In **Actions** tab
2. Click **Deploy Apigee Proxy** workflow
3. Click **Run workflow**
4. Select:
   - **Branch**: `main`
   - **Environment**: `eval`
5. Click **Run workflow**
6. ⏱️ Wait for completion (green checkmark ✅)

### 5.4 Check Deployment Status

1. Click on the running/completed workflow
2. Click on the job name (e.g., "Deploy to Apigee")
3. Expand each step to see logs
4. Look for:
   ```
   ✅ Deployment successful!
   Proxy: create-bank-account
   Environment: eval
   ```

---

## Step 6️⃣: Verify in Apigee Console

### 6.1 Check Shared Flow

1. Go to [Apigee Console](https://apigee.google.com)
2. Login with your GCP account
3. Navigate to **Develop** → **Shared Flows**
4. Verify `common-security-flow` is listed
5. Click on it → **Deployments** → Should show deployed to `eval`

### 6.2 Check KVM

1. Navigate to **Admin** → **Environments** → **Key Value Maps**
2. Select your environment (`eval`)
3. Verify both KVMs exist:
   - ✅ `banking-api-config`
   - ✅ `banking-api-secrets`
4. Click on `banking-api-config` → Verify entries

### 6.3 Check API Proxy

1. Navigate to **Develop** → **API Proxies**
2. Verify `create-bank-account` is listed
3. Click on it → **Deployments** → Should show deployed to `eval`
4. Note the **Base Path**: `/v1/banking`

### 6.4 Get API Endpoint URL

Your API endpoint will be:
```
https://YOUR-PROJECT-ID-eval.apigee.net/v1/banking/accounts
```

Example:
```
https://my-project-12345-eval.apigee.net/v1/banking/accounts
```

---

## Step 7️⃣: Test API with Postman

### 7.1 Get Your API URL

Format: `https://[PROJECT-ID]-[ENV].apigee.net/v1/banking/accounts`

Find your project ID:
1. Go to GCP Console
2. Top navigation bar shows project name and ID
3. Click to copy project ID

### 7.2 Option A: Disable API Key for Testing

Since you don't have API keys set up yet:

1. In Apigee Console → **Develop** → **API Proxies** → `create-bank-account`
2. Click **Develop** tab
3. Click **Proxy Endpoints** → `default`
4. Find the FlowCallout step and comment it out:
   ```xml
   <!-- Temporarily disabled for testing
   <Step>
       <Name>FlowCallout.common-security-flow</Name>
   </Step>
   -->
   ```
5. Click **Save**
6. **Redeploy**: Click **Deploy** → Select `eval` → **Deploy**

### 7.3 Test with Postman

#### Import Collection:
1. Open Postman
2. Click **Import**
3. Drag `create-bank-account.postman_collection.json` from your local folder
4. Click **Import**

#### Update Environment Variables:
1. Click **Environments** (left sidebar)
2. Click **Create Environment** or click on imported environment
3. Set variables:
   - `base_url`: `https://YOUR-PROJECT-ID-eval.apigee.net`
   - `api_key`: Leave empty (if API key disabled)
4. Click **Save**
5. Select environment (dropdown top-right)

#### Send Test Request:
1. Select **Create SAVINGS Account** request
2. Verify URL shows correctly
3. Click **Send**
4. ✅ Expect **201 Created** response with mock data

### 7.4 Manual Test (PowerShell)

```powershell
# Replace with your actual URL
$url = "https://YOUR-PROJECT-ID-eval.apigee.net/v1/banking/accounts"

$body = @{
    accountType = "SAVINGS"
    customerName = "John Doe"
    customerId = "CUST123456"
    initialDeposit = 1000.00
    currency = "USD"
    branchCode = "BR001"
} | ConvertTo-Json

Invoke-RestMethod -Uri $url `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

### 7.5 Expected Response

```json
{
    "accountNumber": "ACC123456789",
    "accountType": "SAVINGS",
    "customerName": "John Doe",
    "customerId": "CUST123456",
    "balance": 1000.00,
    "currency": "USD",
    "branchCode": "BR001",
    "status": "ACTIVE",
    "createdAt": "2025-11-12T...",
    "requestId": "...",
    "message": "Mock response - Account created successfully (no actual backend)"
}
```

---

## Step 8️⃣: Future Deployments (Auto)

### 8.1 Make Changes Locally

```powershell
# Edit files as needed
# Example: Update a policy

git add .
git commit -m "Updated API key verification policy"
```

### 8.2 Push to GitHub

```powershell
# Push to develop branch for test environment
git checkout -b develop
git push -u origin develop

# Or push to main for production
git checkout main
git push
```

### 8.3 Automatic Deployment

- Push to `develop` → Auto-deploys to test/eval environment
- Push to `main` → Auto-deploys to production environment
- GitHub Actions runs automatically
- Check Actions tab for deployment status

---

## 🐛 Troubleshooting

### Issue: Apigee Not Provisioned

**Solution**: 
- Wait for provisioning to complete (30-45 min)
- Check GCP Console → Apigee → Status

### Issue: GitHub Actions Failed - Authentication Error

**Solution**:
- Verify `GCP_SERVICE_ACCOUNT_KEY` secret contains full JSON
- Check service account has correct roles
- Regenerate JSON key if needed

### Issue: 404 Not Found When Testing

**Solution**:
- Verify proxy is deployed in Apigee console
- Check URL format: `https://PROJECT-ID-ENV.apigee.net/v1/banking/accounts`
- Verify base path is `/v1/banking`

### Issue: 401 Unauthorized

**Solution**:
- Disable API key verification temporarily
- Or create API Product and Developer App in Apigee

### Issue: Shared Flow Not Found

**Solution**:
- Deploy shared flow first before deploying API proxy
- Check shared flow is deployed to same environment

### Issue: KVM Not Found

**Solution**:
- Run "Setup KVM" workflow first
- Verify KVM exists in same environment as proxy

---

## 📊 Deployment Summary

| Component | Deployment Order | Workflow |
|-----------|-----------------|----------|
| 1. KVM | First | Setup KVM |
| 2. Shared Flow | Second | Deploy Shared Flow |
| 3. API Proxy | Third | Deploy Apigee Proxy |

---

## 🎓 GCP Trial Account Limits

Your GCP trial includes:
- ✅ $300 free credit
- ✅ 90 days validity
- ✅ Apigee evaluation environment
- ✅ Limited to eval environment (not production)
- ⚠️ Some quotas and limits apply

**Note**: Apigee on GCP trial is sufficient for development and testing!

---

## ✅ Success Checklist

- [ ] GCP trial account activated
- [ ] Apigee provisioned and accessible
- [ ] Service account created with correct roles
- [ ] JSON key downloaded and secured
- [ ] GitHub repository created
- [ ] Code pushed to GitHub
- [ ] GitHub secrets configured (3 secrets)
- [ ] KVM deployed via GitHub Actions
- [ ] Shared flow deployed via GitHub Actions
- [ ] API proxy deployed via GitHub Actions
- [ ] Verified in Apigee console
- [ ] Tested in Postman - 201 response received

---

## 🎯 Quick Reference

### Your Details:
```
GCP Project ID: ___________________
Apigee Org: ___________________
Apigee Environment: eval
API Endpoint: https://___________-eval.apigee.net/v1/banking/accounts
GitHub Repo: https://github.com/___________/create-bank-account-apiproxy
```

### Key Commands:
```powershell
# Push changes
git add .
git commit -m "Your message"
git push

# Check logs
# Go to GitHub → Actions → Click on workflow run
```

---

## 📚 Next Steps

1. ✅ Complete this deployment
2. Create API Products in Apigee
3. Create Developer Apps to get API keys
4. Enable API key verification
5. Add more banking APIs using same shared flow
6. Set up monitoring and analytics

---

## 🆘 Need Help?

- **GCP Issues**: Check [GCP Support](https://cloud.google.com/support)
- **Apigee Docs**: [Apigee Documentation](https://cloud.google.com/apigee/docs)
- **GitHub Actions**: Check workflow logs in Actions tab
- **Test Endpoints**: Use Apigee Trace tool in console

Good luck with your deployment! 🚀
