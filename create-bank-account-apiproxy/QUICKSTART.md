# Quick Start - Testing Without Backend

## Option 1: Disable API Key (Fastest for Testing)

1. Open `apiproxy/proxies/default.xml`
2. Find this section in the PreFlow:
```xml
<Step>
    <Name>VerifyAPIKey</Name>
</Step>
```
3. Comment it out:
```xml
<!-- Temporarily disabled for testing
<Step>
    <Name>VerifyAPIKey</Name>
</Step>
-->
```
4. Deploy the proxy to Apigee
5. Test with Postman without the `x-api-key` header

## Option 2: Use Postman Collection (Recommended)

1. Import `create-bank-account.postman_collection.json` into Postman
2. Set environment variables:
   - `base_url`: Your Apigee endpoint (e.g., `https://myorg-test.apigee.net`)
   - `api_key`: Your API key (or leave blank if disabled)
3. Run any request from the collection
4. Check the response - should be 201 Created with mock data

## Option 3: Use PowerShell Script

Run this in PowerShell (update the URL):

```powershell
$body = @{
    accountType = "SAVINGS"
    customerName = "Test User"
    customerId = "CUST001"
    initialDeposit = 1000.00
    currency = "USD"
    branchCode = "BR001"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://YOUR-APIGEE-URL/v1/banking/accounts" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body
```

## What Changed?

✅ **No backend required** - Returns mock responses directly  
✅ **Works with Postman** - Ready to test immediately  
✅ **Postman collection included** - Pre-configured test requests  
✅ **Can disable API key** - For easier local testing  

## Next Steps

1. Deploy to Apigee
2. Test with Postman
3. Verify mock responses work
4. When backend is ready, update `targets/default.xml`

See `TESTING.md` for detailed testing instructions!
