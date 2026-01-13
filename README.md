# Zoho Books FAWRI API Integration Plugin

This plugin integrates Zoho Books with the FAWRI API, automatically triggering the API endpoint when a new payment is created.

## Overview

- **Trigger**: Payment Creation in Zoho Books
- **Action**: POST request to FAWRI API endpoint
- **Authentication**: OAuth 2.0
- **API Endpoint**: `https://oic.asbblabs.com/ic/api/integration/v1/flows/rest/CHANNELS_FAWRI_FT/1.0/fawri_execute`

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

### Step 1: Create OAuth 2.0 Connection

1. Log in to your Zoho Books account
2. Go to **Settings** > **Integrations** > **Connections**
3. Click **+ Add Connection**
4. Select **Custom Service**
5. Configure the connection:
   - **Connection Name**: `fawri_oauth_connection`
   - **Authentication Type**: OAuth 2.0
   - **Grant Type**: Client Credentials
   - **Token URL**: `https://oic.asbblabs.com/oauth2/token` (update with actual URL)
   - **Client ID**: Your OAuth client ID
   - **Client Secret**: Your OAuth client secret
   - **Scope**: (leave empty or add required scope)
6. Click **Create** and authorize the connection

### Step 2: Create Custom Function

1. Go to **Settings** > **Automation** > **Custom Functions**
2. Click **+ New Custom Function**
3. Set the following:
   - **Function Name**: `triggerFawriApi`
   - **Module**: Payments
4. Copy the code from `scripts/trigger_fawri_api.ds` into the editor
5. Click **Save**

### Step 3: Create Workflow Rule

1. Go to **Settings** > **Automation** > **Workflow Rules**
2. Click **+ New Workflow Rule**
3. Configure the workflow:
   - **Module**: Payments
   - **Workflow Rule Name**: Trigger FAWRI API on Payment Creation
   - **When**: A payment is **Created**
4. Under **Conditions** (optional):
   - Select "All Payments" or add specific criteria
5. Under **Actions**:
   - Click **+ Add Action** > **Custom Function**
   - Select `triggerFawriApi`
   - Map parameters:
     - `paymentId`: `${payment.payment_id}`
     - `organizationId`: `${organization.organization_id}`
6. Click **Save** and ensure the workflow is **Active**

## File Structure

```
zoho-books-fawri-plugin/
├── README.md                          # This file
├── config/
│   ├── oauth_connection.json          # OAuth connection configuration reference
│   └── workflow_rule.json             # Workflow rule configuration reference
└── scripts/
    └── trigger_fawri_api.ds           # Deluge custom function script
```

## Testing

1. Create a test payment in Zoho Books
2. Check the workflow execution logs:
   - Go to **Settings** > **Automation** > **Workflow Rules**
   - Click on your workflow rule
   - View the **Execution History**
3. Verify the API was called successfully

## Troubleshooting

### Common Issues

1. **Connection Error**: Verify OAuth credentials and token URL
2. **API Call Failed**: Check API endpoint URL and payload format
3. **Workflow Not Triggering**: Ensure workflow rule is active and conditions match

### Viewing Logs

- Go to **Settings** > **Automation** > **Custom Functions**
- Select the function and view **Execution Logs**
- Use `info` statements in the script for debugging

## Customization

### Adding Conditions

To trigger only for specific payments, modify the workflow rule conditions:
- Filter by payment amount
- Filter by customer
- Filter by payment mode

### Modifying Payload

Edit the `trigger_fawri_api.ds` script to customize the payload:

```deluge
// Add custom fields
payload.put("custom_field", "value");

// Remove fields
// Comment out or delete unwanted payload.put() lines
```

## Security Notes

- Store OAuth credentials securely in the Zoho connection
- Do not hardcode credentials in the script
- Use HTTPS for all API communications
- Review Zoho's security best practices

## Support

For issues related to:
- **Zoho Books**: Contact Zoho Support
- **FAWRI API**: Contact your API provider
- **This Plugin**: Review the configuration files and logs
