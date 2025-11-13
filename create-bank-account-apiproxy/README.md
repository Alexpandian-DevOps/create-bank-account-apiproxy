# Create Bank Account API Proxy

Apigee API Proxy for creating new bank accounts with automated CI/CD deployment via GitHub Actions.

## 🚀 Quick Start

### Local Testing
1. See [QUICKSTART.md](QUICKSTART.md) for immediate testing without backend
2. Use [TESTING.md](TESTING.md) for comprehensive test scenarios
3. Import `create-bank-account.postman_collection.json` into Postman

### CI/CD Deployment
1. See [CICD-SETUP.md](CICD-SETUP.md) for GitHub Actions setup
2. Push to `develop` → Auto-deploy to test environment
3. Push to `main` → Auto-deploy to production

## 📁 Project Structure

```
create-bank-account-apiproxy/
├── .github/workflows/          # GitHub Actions CI/CD
│   ├── deploy-apigee.yml      # apigeecli deployment
│   └── deploy-maven.yml       # Maven deployment
├── apiproxy/                   # Apigee proxy bundle
│   ├── policies/              # Policy configurations
│   ├── proxies/               # ProxyEndpoint
│   ├── targets/               # TargetEndpoint
│   └── create-bank-account.xml
├── pom.xml                    # Maven configuration
├── CICD-SETUP.md             # CI/CD documentation
├── TESTING.md                # Testing guide
└── README.md                 # This file
```

## 🔧 API Endpoint

**POST** `/v1/banking/accounts`

**Headers:**
- `Content-Type: application/json`
- `x-api-key: YOUR_API_KEY` (optional for testing)

**Request Body:**
```json
{
    "accountType": "SAVINGS",
    "customerName": "John Doe",
    "customerId": "CUST123456",
    "initialDeposit": 1000.00,
    "currency": "USD",
    "branchCode": "BR001"
}
```

## 📖 Documentation

- [CI/CD Setup Guide](CICD-SETUP.md) - GitHub Actions deployment
- [Testing Guide](TESTING.md) - Postman and testing instructions
- [Quick Start](QUICKSTART.md) - Fast setup for local testing

## 🛠️ Deployment Methods

1. **GitHub Actions** (Recommended) - Automated CI/CD
2. **Manual Upload** - Zip and upload via Apigee UI
3. **Maven** - Traditional enterprise deployment
4. **Apigee CLI** - Command-line deployment

## Security Features

- **API Key Verification**: Optional API key validation
- **JSON Threat Protection**: Prevents malicious payloads
- **Mock Response**: No backend required for testing

## Policies Included

- **VerifyAPIKey**: API key validation
- **ValidateRequest**: JSON threat protection
- **AssignMessage-MockResponse**: Returns test data
- **AssignMessage-ErrorResponse**: Error handling
