# Zoho Books FAWRI API Integration Plugin

This plugin integrates Zoho Books with the FAWRI API, automatically triggering the API endpoint when a new payment is created.

## Overview

- **Trigger**: Payment Creation in Zoho Books
- **Action**: POST request to FAWRI API endpoint
- **Authentication**: OAuth 2.0
- **API Endpoint**: `https://{{Env_URL}}/ic/api/integration/v1/flows/rest/CHANNELS_FAWRI_FT/1.0/fawri_execute`

## Quick Start Checklist

Before the plugin works, ensure you have completed:

- [ ] Registered OAuth client at Zoho Developer Console
- [ ] Created OAuth connection (`fawri_oauth_connection`) in Zoho Books
- [ ] Found your Organization ID from Settings > Organization Profile
- [ ] Created custom function with `ORGANIZATION_ID` configured
- [ ] Created and activated workflow rule

## Payload Structure

The following payment information is sent to the API:

```json
{
  "payment_id": "123456789",
  "amount": 1000.00,
  "currency_code": "SAR",
  "date": "2024-01-15",
  "reference_number": "REF-001",
  "payment_mode": "Bank Transfer",
  "description": "Payment description",
  "customer_id": "987654321",
  "customer_name": "Customer Name",
  "triggered_at": "2024-01-15T10:30:00Z"
}
```

## Setup Instructions

### Step 1: Register OAuth Client (Required for Custom Services)

Before creating a connection, register your OAuth client:

1. Go to [Zoho Developer Console](https://accounts.zoho.com/developerconsole)
2. Click **Add Client ID**
3. Fill in the details:
   - **Client Name**: FAWRI Integration
   - **Client Domain**: Your domain
   - **Redirect URI**: `https://deluge.zoho.com/delugeauth/callback`
   - **Client Type**: Web based
4. Click **Create**
5. Note down the **Client ID** and **Client Secret**

### Step 2: Create OAuth 2.0 Connection

1. Log in to your Zoho Books account
2. Click the **Settings** gear icon (top-right corner)
3. Scroll down to **Developer Space** section
4. Click **Connections**
5. Click **+ New Connection** (top-right corner)
6. Go to the **Custom Services** tab
7. Click **+ Create New Service**
8. Configure the service:
   - **Service Name**: FAWRI API
   - **Authentication Type**: OAuth 2.0
   - **Client ID**: (from Step 1)
   - **Client Secret**: (from Step 1)
   - **Authorize URL**: `https://{{Env_URL}}/oauth2/authorize` (update with actual URL)
   - **Access Token URL**: `https://{{Env_URL}}/oauth2/token` (update with actual URL)
   - **Refresh Token URL**: `https://{{Env_URL}}/oauth2/token` (update with actual URL)
9. Click **Create**
10. Now create the connection:
    - **Connection Name**: `fawri_oauth_connection`
    - **Scope**: (add required scopes, comma-separated)
11. Click **Create and Connect**
12. Authorize when prompted

### Step 3: Find Your Organization ID

1. Click the **Settings** gear icon (top-right corner)
2. Click **Organization Profile**
3. Copy your **Organization ID** (you'll need this in the next step)

> **Note**: The Organization ID is typically a numeric value like `60005123456`

### Step 4: Create Custom Function

1. Click the **Settings** gear icon (top-right corner)
2. Select **Automation** from the left sidebar
3. Click **Workflow Actions**
4. In the Workflow Actions pane, select **Custom Functions**
5. Click **+ New Custom Function** (top-right corner)
6. Configure the function:
   - **Function Name**: `triggerFawriApi`
   - **Module**: Payments Received
7. Add the function argument (use the left sidebar to drag/drop or manually add):
   - Click **+ Add Argument**
   - Add `paymentId` (Type: String)
8. Copy the code from `scripts/trigger_fawri_api.ds` into the script editor
9. **IMPORTANT - Configure the script**: Update line 23 with your Organization ID:
   ```deluge
   ORGANIZATION_ID = "60005123456";  // Replace with your actual Organization ID from Step 3
   ```
10. Click **Save**

### Step 5: Create Workflow Rule

1. Click the **Settings** gear icon (top-right corner)
2. Select **Automation** from the left sidebar
3. Click **Workflow Rules**
4. Click **+ New Workflow Rule** (top-right corner)
5. Configure the workflow:
   - **Workflow Rule Name**: Trigger FAWRI API on Payment Creation
   - **Module**: Payments Received
6. Under **Workflow Rule Execution Condition**:
   - **Workflow Type**: Event Based
   - **When a Payment Received is**: Created
7. Under **Conditions** (optional):
   - Select **All Payments Received** or configure specific criteria
8. Under **Actions**:
   - Click **+ Add New Action**
   - Select **Immediate Actions**
   - Choose **Custom Functions** from the dropdown
   - Select `triggerFawriApi`
   - Map the parameter:
     - `paymentId` → Select **Payment ID** from the field picker
9. Click **Save**
10. Ensure the workflow rule is **Active** (toggle switch)

## Configuration Summary

| Configuration Item | Location | Value to Set |
|-------------------|----------|--------------|
| Organization ID | Script line 23 | Your Zoho Books Organization ID |
| Connection Name | Script line 64 | `fawri_oauth_connection` |
| API Endpoint | Script line 27 | Update if different |

## File Structure

```
zoho-books-fawri-plugin/
├── README.md                              # This documentation file
├── config/
│   ├── oauth_connection.json              # OAuth connection configuration reference
│   └── workflow_rule.json                 # Workflow rule configuration reference
└── scripts/
    ├── trigger_fawri_api.ds               # Main Deluge script (uses Zoho connection)
    └── trigger_fawri_api_standalone.ds    # Alternative script (manual OAuth handling)
```

## Script Versions

| Script | Description | When to Use |
|--------|-------------|-------------|
| `trigger_fawri_api.ds` | Uses Zoho connection for OAuth | **Recommended** - Simpler, automatic token management |
| `trigger_fawri_api_standalone.ds` | Manual OAuth token handling | When you need direct control over OAuth flow |

## Testing

1. Create a test payment in Zoho Books
2. Check the workflow execution logs:
   - Go to **Settings** (gear icon) > **Automation** > **Workflow Rules**
   - Click on your workflow rule name
   - View the **Execution History** tab
3. Check custom function logs:
   - Go to **Settings** > **Automation** > **Workflow Actions** > **Custom Functions**
   - Click on `triggerFawriApi`
   - View **Execution Logs**
4. Verify the API was called successfully

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `Variable 'organizationId' is not defined` | Organization ID not configured | Update `ORGANIZATION_ID` on line 23 of the script with your actual Organization ID |
| `Variable 'paymentId' is not defined` | Argument not added to function | Add `paymentId` (String) argument in custom function settings |
| Connection Error | Invalid OAuth credentials | Verify Client ID, Client Secret, and token URLs in the connection |
| API Call Failed | Wrong endpoint or payload | Check API endpoint URL and view execution logs for details |
| Workflow Not Triggering | Workflow inactive | Ensure workflow rule is Active (green toggle) |
| Function Not Executing | Parameter not mapped | Verify `paymentId` is mapped to Payment ID in workflow action |

### Viewing Logs

1. **Custom Function Logs**:
   - **Settings** > **Automation** > **Workflow Actions** > **Custom Functions**
   - Select the function > **Execution Logs**

2. **Workflow Execution History**:
   - **Settings** > **Automation** > **Workflow Rules**
   - Select the rule > **Execution History**

3. Use `info` statements in the script for debugging (logs appear in Execution Logs)

### Debug Tips

Add these lines to the script for debugging:

```deluge
info "Payment ID received: " + paymentId;
info "Organization ID: " + ORGANIZATION_ID;
info "Payment Response: " + paymentResponse;
```

## Customization

### Adding Conditions

To trigger only for specific payments, modify the workflow rule conditions:
- Filter by payment amount (e.g., amount > 1000)
- Filter by customer
- Filter by payment mode
- Use field comparison (2025 feature)

### Modifying Payload

Edit the `trigger_fawri_api.ds` script to customize the payload:

```deluge
// Add custom fields
payload.put("custom_field", "value");

// Add invoice details if needed
payload.put("invoice_id", payment.get("invoice_id"));

// Remove fields
// Comment out or delete unwanted payload.put() lines
```

## Security Notes

- Store OAuth credentials securely in the Zoho connection (never hardcode in scripts)
- Use the connection-based script (`trigger_fawri_api.ds`) for production
- Use HTTPS for all API communications
- Review and restrict OAuth scopes to minimum required permissions
- Regularly rotate OAuth credentials

## Recent Zoho Books Updates (2025)

- **Field Comparison**: You can now create workflow rules with field-to-field comparison criteria
- **Sub-entity Workflows**: Workflow rules can be set up for sub-entities within modules
- **Background Processing**: Long-running custom functions (>10 seconds) can run in background with in-app notifications

## References

- [Zoho Books Automation Help](https://www.zoho.com/us/books/help/settings/automation.html)
- [Creating Custom Functions](https://www.zoho.com/us/books/kb/automation/custom-function.html)
- [Zoho Books API Documentation](https://www.zoho.com/books/api/v3/introduction/)
- [Zoho OAuth 2.0 Guide](https://www.zoho.com/accounts/protocol/oauth.html)

## Support

For issues related to:
- **Zoho Books**: Contact [Zoho Support](https://www.zoho.com/books/support.html)
- **FAWRI API**: Contact your API provider
- **This Plugin**: Review the configuration files and execution logs
