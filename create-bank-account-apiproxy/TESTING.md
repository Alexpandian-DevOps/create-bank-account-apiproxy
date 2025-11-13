# Testing Guide - Create Bank Account API (Mock Mode)

This API proxy is configured to work **without a backend** for testing purposes. It returns mock responses directly.

## Testing with Postman

### Step 1: Import to Postman

1. Open Postman
2. Create a new request
3. Set method to **POST**
4. Enter URL: `https://<your-apigee-domain>/v1/banking/accounts`
   - Replace `<your-apigee-domain>` with your actual Apigee endpoint
   - Example: `https://myorg-test.apigee.net/v1/banking/accounts`

### Step 2: Configure Headers

Add these headers in Postman:

| Key | Value |
|-----|-------|
| `x-api-key` | `your-api-key-here` |
| `Content-Type` | `application/json` |

**Note**: If you don't have an API key yet, you can temporarily disable the VerifyAPIKey policy by commenting it out in `proxies/default.xml`

### Step 3: Set Request Body

Select **Body** → **raw** → **JSON** and paste:

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

### Step 4: Send Request

Click **Send**. You should receive a **201 Created** response:

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
    "createdAt": "2025-11-11T10:30:00Z",
    "requestId": "req-12345",
    "message": "Mock response - Account created successfully (no actual backend)"
}
```

## Testing with cURL

### Windows PowerShell:
```powershell
$headers = @{
    "x-api-key" = "your-api-key-here"
    "Content-Type" = "application/json"
}

$body = @{
    accountType = "SAVINGS"
    customerName = "John Doe"
    customerId = "CUST123456"
    initialDeposit = 1000.00
    currency = "USD"
    branchCode = "BR001"
} | ConvertTo-Json

Invoke-WebRequest -Uri "https://your-apigee-domain/v1/banking/accounts" `
    -Method POST `
    -Headers $headers `
    -Body $body
```

### Linux/Mac (bash):
```bash
curl -X POST https://your-apigee-domain/v1/banking/accounts \
  -H "x-api-key: your-api-key-here" \
  -H "Content-Type: application/json" \
  -d '{
    "accountType": "SAVINGS",
    "customerName": "John Doe",
    "customerId": "CUST123456",
    "initialDeposit": 1000.00,
    "currency": "USD",
    "branchCode": "BR001"
  }'
```

## Testing Without API Key (Temporary)

If you don't have an API key set up yet, you can disable the verification:

1. Open `apiproxy/proxies/default.xml`
2. Comment out or remove the VerifyAPIKey step:
```xml
<!-- Temporarily disabled for testing
<Step>
    <Name>VerifyAPIKey</Name>
</Step>
-->
```
3. Redeploy the proxy
4. Test without the `x-api-key` header

## Test Scenarios

### 1. Valid Request - SAVINGS Account
```json
{
    "accountType": "SAVINGS",
    "customerName": "Alice Smith",
    "customerId": "CUST001",
    "initialDeposit": 5000.00,
    "currency": "USD",
    "branchCode": "BR001"
}
```

### 2. Valid Request - CHECKING Account
```json
{
    "accountType": "CHECKING",
    "customerName": "Bob Johnson",
    "customerId": "CUST002",
    "initialDeposit": 2500.00,
    "currency": "EUR",
    "branchCode": "BR002"
}
```

### 3. Valid Request - BUSINESS Account
```json
{
    "accountType": "BUSINESS",
    "customerName": "Tech Startup Inc",
    "customerId": "CUST003",
    "initialDeposit": 50000.00,
    "currency": "USD",
    "branchCode": "BR003"
}
```

### 4. Invalid Request - Missing Fields
```json
{
    "accountType": "SAVINGS",
    "customerName": "Test User"
}
```
Expected: Error response

### 5. Invalid Request - Wrong HTTP Method
Try GET instead of POST - should fail

## Postman Collection

You can create a Postman Collection with these test cases:

1. **Create SAVINGS Account**
2. **Create CHECKING Account**
3. **Create BUSINESS Account**
4. **Invalid - Missing Fields**
5. **Invalid - Wrong Method**

## Expected Responses

### Success (201 Created)
- Status Code: 201
- Response includes all account details
- Account number is randomly generated
- Timestamp shows current time

### Error (400 Bad Request)
- Status Code: 400
- Response includes error details:
```json
{
    "error": {
        "code": "ErrorCode",
        "message": "Error description",
        "timestamp": "2025-11-11T10:30:00Z",
        "requestId": "req-12345"
    }
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| 401 Unauthorized | Check API key or disable VerifyAPIKey temporarily |
| 404 Not Found | Verify the URL path is `/v1/banking/accounts` |
| 405 Method Not Allowed | Ensure you're using POST method |
| Invalid JSON | Check request body formatting |

## Moving to Production

When you have a backend service ready:

1. Update `apiproxy/targets/default.xml` with backend URL
2. Replace `AssignMessage-MockResponse` with actual backend call
3. Re-enable all security policies
4. Add rate limiting and quota policies
5. Test with actual backend integration

## Import Postman Collection

Save this as `create-bank-account.postman_collection.json`:

```json
{
    "info": {
        "name": "Create Bank Account API",
        "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
    },
    "item": [
        {
            "name": "Create SAVINGS Account",
            "request": {
                "method": "POST",
                "header": [
                    {
                        "key": "x-api-key",
                        "value": "{{api_key}}",
                        "type": "text"
                    },
                    {
                        "key": "Content-Type",
                        "value": "application/json",
                        "type": "text"
                    }
                ],
                "body": {
                    "mode": "raw",
                    "raw": "{\n    \"accountType\": \"SAVINGS\",\n    \"customerName\": \"John Doe\",\n    \"customerId\": \"CUST123456\",\n    \"initialDeposit\": 1000.00,\n    \"currency\": \"USD\",\n    \"branchCode\": \"BR001\"\n}"
                },
                "url": {
                    "raw": "{{base_url}}/v1/banking/accounts",
                    "host": ["{{base_url}}"],
                    "path": ["v1", "banking", "accounts"]
                }
            }
        }
    ],
    "variable": [
        {
            "key": "base_url",
            "value": "https://your-apigee-domain",
            "type": "string"
        },
        {
            "key": "api_key",
            "value": "your-api-key-here",
            "type": "string"
        }
    ]
}
```
