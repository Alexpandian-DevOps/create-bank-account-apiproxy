# GitHub Actions CI/CD for Apigee

This repository contains GitHub Actions workflows to automatically deploy the Apigee proxy to Google Cloud Apigee.

## 🚀 Available Workflows

### 1. Deploy with apigeecli (Recommended)
**File**: `.github/workflows/deploy-apigee.yml`

- ✅ Fast and lightweight
- ✅ Automatic environment detection
- ✅ Built-in smoke testing
- ✅ Manual trigger support

### 2. Deploy with Maven
**File**: `.github/workflows/deploy-maven.yml`

- ✅ Traditional Java-based deployment
- ✅ Maven plugin integration
- ✅ Enterprise-grade tooling

## 📋 Setup Instructions

### Step 1: Create GitHub Repository

```bash
cd C:\Users\AS001028268\Downloads\create-bank-account-apiproxy
git init
git add .
git commit -m "Initial commit - Create Bank Account API Proxy"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/create-bank-account-apiproxy.git
git push -u origin main
```

### Step 2: Create Google Cloud Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Select your project
3. Navigate to **IAM & Admin** → **Service Accounts**
4. Click **Create Service Account**
5. Name: `apigee-deploy-sa`
6. Grant roles:
   - **Apigee API Admin**
   - **Apigee Environment Admin**
7. Click **Create Key** → **JSON**
8. Download the JSON key file

### Step 3: Configure GitHub Secrets

1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add these secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `GCP_SERVICE_ACCOUNT_KEY` | Contents of JSON key file | Service account credentials |
| `APIGEE_ORG` | Your Apigee organization name | e.g., `my-org-name` |
| `APIGEE_ENV` | Default environment | e.g., `test` or `prod` |

### Step 4: Enable GitHub Actions

1. Go to **Actions** tab in your repository
2. If prompted, click **I understand my workflows, go ahead and enable them**

## 🎯 How to Deploy

### Automatic Deployment

Push to branches:
- **Push to `develop`** → Deploys to `test` environment
- **Push to `main`** → Deploys to `prod` environment

```bash
# Deploy to test environment
git checkout develop
git add .
git commit -m "Update proxy"
git push

# Deploy to production
git checkout main
git merge develop
git push
```

### Manual Deployment

1. Go to **Actions** tab
2. Select **Deploy Apigee Proxy**
3. Click **Run workflow**
4. Choose environment (`test` or `prod`)
5. Click **Run workflow**

## 📊 Pipeline Stages

```
┌─────────────┐
│  Checkout   │
└──────┬──────┘
       │
┌──────▼──────┐
│ Authenticate│
└──────┬──────┘
       │
┌──────▼──────┐
│Create Bundle│
└──────┬──────┘
       │
┌──────▼──────┐
│   Deploy    │
└──────┬──────┘
       │
┌──────▼──────┐
│ Smoke Test  │
└─────────────┘
```

## 🧪 Testing After Deployment

The pipeline automatically runs a smoke test. You can also test manually:

```bash
# Get your deployment URL from GitHub Actions logs
curl -X POST https://YOUR-ORG-test.apigee.net/v1/banking/accounts \
  -H "Content-Type: application/json" \
  -d '{
    "accountType": "SAVINGS",
    "customerName": "Pipeline Test",
    "customerId": "PIPE001",
    "initialDeposit": 1000.00,
    "currency": "USD",
    "branchCode": "BR001"
  }'
```

## 📝 Workflow Configuration

### Environment Mapping

| Branch | Environment |
|--------|-------------|
| `develop` | `test` |
| `main` | `prod` |
| Manual trigger | User selected |

### Triggers

- ✅ Push to `main` or `develop`
- ✅ Pull request to `main`
- ✅ Manual workflow dispatch

## 🔧 Customization

### Change Environment Names

Edit `.github/workflows/deploy-apigee.yml`:

```yaml
- name: Set deployment environment
  run: |
    if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
      echo "ENVIRONMENT=production" >> $GITHUB_ENV  # Change here
    else
      echo "ENVIRONMENT=development" >> $GITHUB_ENV  # Change here
    fi
```

### Add More Environments

Update the workflow_dispatch inputs:

```yaml
workflow_dispatch:
  inputs:
    environment:
      type: choice
      options:
        - dev
        - test
        - uat
        - prod
```

### Add Approval Gates

For production deployments, add environment protection rules:

1. Go to **Settings** → **Environments**
2. Create environment: `prod`
3. Enable **Required reviewers**
4. Add reviewers

Update workflow:
```yaml
jobs:
  deploy:
    environment: prod  # Add this line
    runs-on: ubuntu-latest
```

## 🐛 Troubleshooting

### Deployment Failed

Check GitHub Actions logs:
1. Go to **Actions** tab
2. Click on failed workflow
3. Check error messages

Common issues:
- ❌ Invalid service account key → Check `GCP_SERVICE_ACCOUNT_KEY` secret
- ❌ Permission denied → Check service account roles
- ❌ Organization not found → Verify `APIGEE_ORG` secret

### Smoke Test Failed

- Check if proxy is actually deployed in Apigee console
- Verify the API endpoint URL format
- Check if API key verification is enabled

## 📚 Additional Resources

- [Apigee CLI Documentation](https://github.com/apigee/apigeecli)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Apigee Maven Plugin](https://github.com/apigee/apigee-deploy-maven-plugin)

## 🔐 Security Best Practices

1. ✅ Never commit service account keys to repository
2. ✅ Use GitHub Secrets for sensitive data
3. ✅ Enable branch protection rules
4. ✅ Require pull request reviews
5. ✅ Use environment-specific secrets
6. ✅ Rotate service account keys regularly

## 📈 Next Steps

1. Set up multiple environments (dev, test, uat, prod)
2. Add unit tests for policies
3. Implement API testing with Newman/Postman
4. Add deployment notifications (Slack, Email)
5. Set up monitoring and alerting
