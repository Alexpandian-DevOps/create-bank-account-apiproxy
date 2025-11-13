# KVM (Key-Value Maps) Setup Guide

This guide explains how to set up and manage Key-Value Maps (KVM) for the banking API proxy.

## 📋 KVM Overview

We use two KVM stores:

1. **banking-api-config** (Non-encrypted) - Configuration settings
2. **banking-api-secrets** (Encrypted) - Sensitive credentials

## 🔧 KVM Configuration Details

### 1. banking-api-config (Configuration)

**Scope**: Environment  
**Encrypted**: No

| Key | Value | Description |
|-----|-------|-------------|
| `api_key_enabled` | `true` | Enable/disable API key verification |
| `quota_enabled` | `true` | Enable/disable quota policy |
| `rate_limit` | `100` | Requests per minute limit |
| `backend_timeout` | `30000` | Backend timeout in milliseconds |
| `log_level` | `INFO` | Logging level (DEBUG, INFO, WARN, ERROR) |
| `enable_cors` | `true` | Enable CORS headers |
| `cors_origin` | `*` | Allowed CORS origins |
| `max_request_size` | `1048576` | Max request size in bytes |
| `enable_caching` | `false` | Enable response caching |
| `cache_ttl` | `300` | Cache TTL in seconds |

### 2. banking-api-secrets (Secrets)

**Scope**: Environment  
**Encrypted**: Yes

| Key | Description |
|-----|-------------|
| `backend_api_key` | Backend service API key |
| `backend_client_id` | Backend OAuth client ID |
| `backend_client_secret` | Backend OAuth client secret |
| `encryption_key` | Data encryption key |
| `jwt_secret` | JWT signing secret |
| `database_connection_string` | Database connection string |

## 🚀 Setup Methods

### Method 1: Via Apigee UI (Recommended for Initial Setup)

#### Create banking-api-config KVM:

1. Login to Apigee Console
2. Navigate to **Admin** → **Environments** → **Key Value Maps**
3. Select your environment (test/prod)
4. Click **+ Key Value Map**
5. Name: `banking-api-config`
6. Encrypted: **No**
7. Click **Add**
8. Add each entry from the table above:
   - Click **+ Entry**
   - Key: `api_key_enabled`
   - Value: `true`
   - Repeat for all keys

#### Create banking-api-secrets KVM:

1. Same steps as above
2. Name: `banking-api-secrets`
3. Encrypted: **Yes** ⚠️
4. Add sensitive entries
5. Values will be encrypted automatically

### Method 2: Via Apigee CLI

#### Create Configuration KVM:

```bash
# Create the KVM
apigeecli kvms create \
  --name banking-api-config \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)

# Add entries
apigeecli kvms entries create \
  --map banking-api-config \
  --key api_key_enabled \
  --value true \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)
```

#### Create Secrets KVM:

```bash
# Create encrypted KVM
apigeecli kvms create \
  --name banking-api-secrets \
  --env test \
  --org YOUR_ORG \
  --encrypted \
  --token $(gcloud auth print-access-token)

# Add encrypted entries
apigeecli kvms entries create \
  --map banking-api-secrets \
  --key backend_api_key \
  --value "your-secret-key" \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)
```

### Method 3: Via API (PowerShell)

```powershell
# Get access token
$TOKEN = gcloud auth print-access-token
$ORG = "your-org-name"
$ENV = "test"

# Create KVM
$body = @{
    name = "banking-api-config"
    encrypted = $false
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://apigee.googleapis.com/v1/organizations/$ORG/environments/$ENV/keyvaluemaps" `
    -Method POST `
    -Headers @{Authorization = "Bearer $TOKEN"; "Content-Type" = "application/json"} `
    -Body $body

# Add entry
$entry = @{
    name = "api_key_enabled"
    value = "true"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://apigee.googleapis.com/v1/organizations/$ORG/environments/$ENV/keyvaluemaps/banking-api-config/entries" `
    -Method POST `
    -Headers @{Authorization = "Bearer $TOKEN"; "Content-Type" = "application/json"} `
    -Body $entry
```

### Method 4: Import from JSON (Bulk)

```bash
# Import configuration KVM
cat kvm/banking-api-config.json | \
  apigeecli kvms import \
    --env test \
    --org YOUR_ORG \
    --token $(gcloud auth print-access-token)

# Import secrets KVM
cat kvm/banking-api-secrets.json | \
  apigeecli kvms import \
    --env test \
    --org YOUR_ORG \
    --encrypted \
    --token $(gcloud auth print-access-token)
```

## 🔍 Verify KVM Setup

### Via Apigee UI:
1. Navigate to **Admin** → **Environments** → **Key Value Maps**
2. Select environment
3. Click on KVM name
4. Verify all entries are present

### Via CLI:
```bash
# List KVMs
apigeecli kvms list \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)

# Get KVM entries
apigeecli kvms entries list \
  --map banking-api-config \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)
```

## 📝 Usage in Proxy

### Get Configuration Value:

```xml
<KeyValueMapOperations name="GetConfig" mapIdentifier="banking-api-config">
    <Scope>environment</Scope>
    <Get assignTo="kvm.config.api_key_enabled">
        <Key>
            <Parameter>api_key_enabled</Parameter>
        </Key>
    </Get>
</KeyValueMapOperations>
```

### Get Secret Value:

```xml
<KeyValueMapOperations name="GetSecret" mapIdentifier="banking-api-secrets">
    <Scope>environment</Scope>
    <Get assignTo="private.backend.api.key">
        <Key>
            <Parameter>backend_api_key</Parameter>
        </Key>
    </Get>
</KeyValueMapOperations>
```

### Use in Conditional Flow:

```xml
<Step>
    <Name>VerifyAPIKey</Name>
    <Condition>kvm.config.api_key_enabled = "true"</Condition>
</Step>
```

## 🔒 Security Best Practices

1. ✅ **Always encrypt sensitive data** - Use encrypted KVMs for secrets
2. ✅ **Environment-specific KVMs** - Different values for test/prod
3. ✅ **Use private variables** - Prefix with `private.` for secrets
4. ✅ **Cache KVM reads** - Use ExpiryTimeInSecs to reduce lookups
5. ✅ **Limit scope** - Use environment scope instead of organization
6. ✅ **Audit access** - Monitor KVM access in audit logs
7. ✅ **Rotate secrets** - Regularly update encrypted values

## 🔄 Update KVM Values

### Via UI:
1. Navigate to KVM
2. Click on entry
3. Update value
4. Save

### Via CLI:
```bash
apigeecli kvms entries update \
  --map banking-api-config \
  --key rate_limit \
  --value 200 \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)
```

## 🗑️ Delete KVM Entry

```bash
apigeecli kvms entries delete \
  --map banking-api-config \
  --key old_key \
  --env test \
  --org YOUR_ORG \
  --token $(gcloud auth print-access-token)
```

## 📊 Environment-Specific Values

| Setting | Test Environment | Production |
|---------|-----------------|------------|
| `rate_limit` | 100 | 1000 |
| `backend_timeout` | 30000 | 10000 |
| `log_level` | DEBUG | WARN |
| `enable_cors` | true | false |
| `cors_origin` | * | https://app.example.com |

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| KVM not found | Verify KVM name and environment scope |
| Permission denied | Check service account has KVM admin role |
| Encrypted value not decrypting | Ensure KVM is created with encrypted flag |
| Variable not resolving | Check ExpiryTimeInSecs and cache settings |

## 📚 Related Files

- `kvm/banking-api-config.json` - Configuration KVM template
- `kvm/banking-api-secrets.json` - Secrets KVM template  
- `apiproxy/policies/KVM-GetSecrets.xml` - Policy to retrieve secrets
- `sharedflows/.../policies/SF-GetKVMConfig.xml` - Shared flow KVM policy
