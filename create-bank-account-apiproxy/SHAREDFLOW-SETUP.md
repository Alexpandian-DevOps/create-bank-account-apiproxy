# Shared Flow Setup Guide

This guide explains how to create and deploy the common security shared flow that can be reused across all banking APIs.

## 📋 Shared Flow Overview

**Name**: `common-security-flow`  
**Purpose**: Reusable security and validation logic for all banking APIs

### Features Included:

✅ **API Key Verification** - Validates API keys  
✅ **JSON Threat Protection** - Prevents malicious payloads  
✅ **Spike Arrest** - Rate limiting (100 requests/minute)  
✅ **Quota Management** - Monthly API call limits  
✅ **KVM Configuration** - Reads settings from KVM  
✅ **CORS Headers** - Cross-origin resource sharing  
✅ **Security Headers** - Removes sensitive headers  
✅ **Request Logging** - Logs API requests  

## 🏗️ Shared Flow Structure

```
sharedflows/common-security-flow/
└── sharedflowbundle/
    ├── common-security-flow.xml       # Main bundle definition
    ├── sharedflows/
    │   └── default.xml                # Flow logic
    └── policies/
        ├── SF-VerifyAPIKey.xml
        ├── SF-JSONThreatProtection.xml
        ├── SF-SpikeArrest.xml
        ├── SF-Quota.xml
        ├── SF-GetKVMConfig.xml
        ├── SF-CORS.xml
        ├── SF-RemoveHeaders.xml
        └── SF-LogRequest.xml
```

## 🚀 Deployment Methods

### Method 1: Via Apigee UI

#### Step 1: Create ZIP Bundle

```powershell
cd C:\Users\AS001028268\Downloads\create-bank-account-apiproxy
cd sharedflows\common-security-flow
Compress-Archive -Path .\sharedflowbundle\* -DestinationPath common-security-flow.zip -Force
```

#### Step 2: Upload to Apigee

1. Login to Apigee Console
2. Navigate to **Develop** → **Shared Flows**
3. Click **+ Create New** or **Create**
4. Select **Upload Bundle**
5. Choose `common-security-flow.zip`
6. Click **Next** → **Create**

#### Step 3: Deploy Shared Flow

1. After creation, click on the shared flow
2. Click **Deployment** dropdown
3. Select environment (test/prod)
4. Click **Deploy**
5. Wait for green checkmark ✅

### Method 2: Via Apigee CLI

```bash
# Navigate to shared flow directory
cd sharedflows/common-security-flow

# Create bundle
zip -r common-security-flow.zip sharedflowbundle/

# Deploy shared flow
apigeecli sharedflows create bundle \
  --name common-security-flow \
  --proxy-zip common-security-flow.zip \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token) \
  --ovr

# Deploy to environment
apigeecli sharedflows deploy \
  --name common-security-flow \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token) \
  --wait
```

### Method 3: Via GitHub Actions (Automated)

Add to `.github/workflows/deploy-shared-flow.yml`:

```yaml
name: Deploy Shared Flow

on:
  push:
    paths:
      - 'sharedflows/**'
  workflow_dispatch:

jobs:
  deploy-shared-flow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Authenticate to GCP
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SERVICE_ACCOUNT_KEY }}
      
      - name: Install apigeecli
        run: |
          curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh -
          sudo mv apigeecli /usr/local/bin/
      
      - name: Deploy Shared Flow
        run: |
          cd sharedflows/common-security-flow
          zip -r bundle.zip sharedflowbundle/
          
          apigeecli sharedflows create bundle \
            --name common-security-flow \
            --proxy-zip bundle.zip \
            --org ${{ secrets.APIGEE_ORG }} \
            --token $(gcloud auth print-access-token) \
            --ovr
          
          apigeecli sharedflows deploy \
            --name common-security-flow \
            --env test \
            --org ${{ secrets.APIGEE_ORG }} \
            --token $(gcloud auth print-access-token) \
            --wait
```

## 🔗 Using Shared Flow in API Proxies

### Step 1: Add FlowCallout Policy

Create `FlowCallout.common-security-flow.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<FlowCallout name="FlowCallout.common-security-flow">
    <DisplayName>Call Common Security Shared Flow</DisplayName>
    <SharedFlowBundle>common-security-flow</SharedFlowBundle>
</FlowCallout>
```

### Step 2: Reference in ProxyEndpoint

In `proxies/default.xml`:

```xml
<PreFlow name="PreFlow">
    <Request>
        <Step>
            <Name>FlowCallout.common-security-flow</Name>
        </Step>
    </Request>
    <Response/>
</PreFlow>
```

### Step 3: Update Proxy Bundle

Add to `apiproxy/your-proxy.xml`:

```xml
<Policies>
    <Policy>FlowCallout.common-security-flow</Policy>
    <!-- other policies -->
</Policies>
```

## 📝 Policy Configuration

### 1. Spike Arrest (Rate Limiting)

**Default**: 100 requests per minute

To modify:
```xml
<Rate>200pm</Rate>  <!-- 200 per minute -->
<Rate>10ps</Rate>   <!-- 10 per second -->
```

### 2. Quota Policy

**Default**: 1000 requests per month

Quota is read from API Product settings:
- Can be overridden in API Product configuration
- Distributed across multiple message processors
- Resets monthly

### 3. KVM Integration

Reads configuration from `banking-api-config` KVM:
- `api_key_enabled` - Enable/disable API key check
- `quota_enabled` - Enable/disable quota
- `enable_cors` - Enable/disable CORS headers

### 4. CORS Configuration

Default CORS headers:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, x-api-key
Access-Control-Max-Age: 3600
```

## 🎯 Example: Multiple APIs Using Shared Flow

### API 1: Create Bank Account
```xml
<Step>
    <Name>FlowCallout.common-security-flow</Name>
</Step>
```

### API 2: Transfer Money
```xml
<Step>
    <Name>FlowCallout.common-security-flow</Name>
</Step>
```

### API 3: Get Account Balance
```xml
<Step>
    <Name>FlowCallout.common-security-flow</Name>
</Step>
```

All three APIs get the same security policies automatically! 🎉

## 🔧 Customization Per API

### Conditional Execution

```xml
<Step>
    <Name>FlowCallout.common-security-flow</Name>
    <Condition>request.verb != "OPTIONS"</Condition>
</Step>
```

### Skip Specific Policies

Set KVM values:
```json
{
  "api_key_enabled": "false",  // Skip API key check
  "quota_enabled": "false"      // Skip quota
}
```

## 🧪 Testing Shared Flow

### Test 1: Rate Limiting

```bash
# Send 150 requests quickly
for i in {1..150}; do
  curl -X POST https://YOUR-ORG-test.apigee.net/v1/banking/accounts \
    -H "Content-Type: application/json" \
    -d '{"accountType":"SAVINGS"}' &
done
```

Expected: Some requests should get 429 (Too Many Requests)

### Test 2: CORS Headers

```bash
curl -I https://YOUR-ORG-test.apigee.net/v1/banking/accounts
```

Should include:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
```

### Test 3: Security Headers Removed

Should NOT include:
```
X-Powered-By
Server
```

## 📊 Shared Flow Benefits

| Benefit | Description |
|---------|-------------|
| **Reusability** | Write once, use in all APIs |
| **Consistency** | Same security across all APIs |
| **Maintainability** | Update in one place |
| **Version Control** | Easy to track changes |
| **Testing** | Test once, applies to all |

## 🔄 Update Shared Flow

1. Modify policies in `sharedflowbundle/`
2. Create new ZIP bundle
3. Upload to Apigee (creates new revision)
4. Deploy new revision
5. All APIs using it get updated automatically

## 🗑️ Undeploy Shared Flow

```bash
apigeecli sharedflows undeploy \
  --name common-security-flow \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)
```

⚠️ **Warning**: Undeploy will break any API proxies using it!

## 📚 Advanced Usage

### Multiple Shared Flows

```xml
<!-- Security -->
<Step>
    <Name>FlowCallout.common-security-flow</Name>
</Step>

<!-- Logging -->
<Step>
    <Name>FlowCallout.common-logging-flow</Name>
</Step>

<!-- Analytics -->
<Step>
    <Name>FlowCallout.common-analytics-flow</Name>
</Step>
```

### Shared Flow with Parameters

```xml
<FlowCallout name="FlowCallout.common-security-flow">
    <Parameters>
        <Parameter name="skip_quota">true</Parameter>
    </Parameters>
    <SharedFlowBundle>common-security-flow</SharedFlowBundle>
</FlowCallout>
```

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| Shared flow not found | Verify deployment in correct environment |
| Policies not executing | Check conditions in shared flow |
| KVM values not loading | Ensure KVM exists in same environment |
| CORS not working | Check `enable_cors` in KVM |

## 📈 Best Practices

1. ✅ Use descriptive names (SF- prefix for shared flow policies)
2. ✅ Version your shared flows
3. ✅ Test in lower environments first
4. ✅ Document all policies
5. ✅ Use KVM for configuration
6. ✅ Handle errors gracefully
7. ✅ Monitor shared flow usage
8. ✅ Keep shared flows focused (single responsibility)

## 🔗 Related Files

- `sharedflows/common-security-flow/` - Shared flow bundle
- `apiproxy/policies/FlowCallout.common-security-flow.xml` - FlowCallout policy
- `KVM-SETUP.md` - KVM configuration guide
