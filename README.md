# Zoho Books FAWRI API Integration Plugin

This plugin integrates Zoho Books with the FAWRI API, automatically triggering the API endpoint when you record a payment to a vendor (pay a bill).

## Overview

- **Trigger**: When you pay a vendor/supplier (bill payment recorded)
- **Action**: POST request to FAWRI API endpoint
- **Authentication**: OAuth 2.0
- **API Endpoint**: `https://oic.asbblabs.com/ic/api/integration/v1/flows/rest/CHANNELS_FAWRI_FT/1.0/fawri_execute`

## When Does This Trigger?

The function triggers when:
1. You have a **Bill** from a vendor
2. You **record a payment** against that bill
3. The bill status changes to **Paid**

**Example flow:**
```
Vendor sends invoice → You create Bill in Zoho → You record payment → FAWRI API triggered
```

## Quick Start Checklist

- [ ] Created OAuth connection (`fawri_oauth_connection`) in Zoho Books
- [ ] Found your Organization ID from Settings > Organization Profile
- [ ] Created custom function with Module: **Bills** and `ORGANIZATION_ID` configured
- [ ] Created workflow rule to trigger when bill is **Paid**

## Payload Structure

The following bill/payment information is sent to the API:

```json
{
  "bill_id": "123456789",
  "bill_number": "BILL-001",
  "amount": 1000.00,
  "amount_paid": 1000.00,
  "balance": 0.00,
  "currency_code": "SAR",
  "date": "2024-01-15",
  "due_date": "2024-02-15",
  "reference_number": "REF-001",
  "status": "paid",
  "vendor_id": "987654321",
  "vendor_name": "Vendor Name",
  "triggered_at": "2024-01-15T10:30:00Z"
}
```

## Setup Instructions

### Step 1: Create OAuth 2.0 Connection

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
   - **Authorize URL**: `https://oic.asbblabs.com/oauth2/authorize` (update with actual)
   - **Access Token URL**: `https://oic.asbblabs.com/oauth2/token` (update with actual)
   - **Refresh Token URL**: `https://oic.asbblabs.com/oauth2/token` (update with actual)
   - **Client ID**: Your OAuth client ID
   - **Client Secret**: Your OAuth client secret
9. Click **Create**
10. Create the connection:
    - **Connection Name**: `fawri_oauth_connection`
11. Click **Create and Connect**
12. Authorize when prompted

### Step 2: Find Your Organization ID

1. Click the **Settings** gear icon (top-right corner)
2. Click **Organization Profile**
3. Copy your **Organization ID** (numeric value like `60005123456`)

### Step 3: Create Custom Function

1. Click **Settings** gear icon (top-right corner)
2. Select **Automation** from the left sidebar
3. Click **Workflow Actions**
4. Select **Custom Functions**
5. Click **+ New Custom Function**
6. Configure:
   - **Function Name**: `triggerFawriApi`
   - **Module**: **Bills**
7. Copy the code from `scripts/trigger_fawri_api.ds` into the editor
8. **Update line 24** with your Organization ID:
   ```deluge
   ORGANIZATION_ID = "60005123456";
   ```
9. Click **Save**

### Step 4: Create Workflow Rule

1. Click **Settings** gear icon (top-right corner)
2. Select **Automation** from the left sidebar
3. Click **Workflow Rules**
4. Click **+ New Workflow Rule**
5. Configure:
   - **Workflow Rule Name**: Trigger FAWRI API on Bill Payment
   - **Module**: **Bills**
6. Under **Workflow Rule Execution Condition**:
   - **Workflow Type**: Event Based
   - **When a Bill is**: **Edited**
7. Under **Conditions**:
   - **Payment Status** is **Paid**
8. Under **Actions**:
   - Click **+ Add New Action**
   - Select **Immediate Actions**
   - Choose **Custom Functions**
   - Select `triggerFawriApi`
9. Click **Save**
10. Ensure workflow is **Active**

## File Structure

```
zoho-books-fawri-plugin/
├── README.md
├── config/
│   ├── oauth_connection.json
│   └── workflow_rule.json
└── scripts/
    ├── trigger_fawri_api.ds              # Main script (uses Zoho connection)
    └── trigger_fawri_api_standalone.ds   # Alternative (manual OAuth)
```

## Testing

1. Create a Bill in Zoho Books (Purchases > Bills > + New)
2. Record a payment against the bill (Pay Bill)
3. Check workflow logs: **Settings** > **Automation** > **Workflow Rules** > Your rule > **Execution History**
4. Check function logs: **Settings** > **Automation** > **Workflow Actions** > **Custom Functions** > `triggerFawriApi` > **Execution Logs**

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `Variable 'bill' is not defined` | Ensure custom function Module is set to **Bills** |
| `Variable 'ORGANIZATION_ID' is not defined` | Update line 24 with your Organization ID |
| Workflow not triggering | Check condition is "Payment Status is Paid" |
| Connection error | Verify OAuth credentials and token URLs |

## Debug Tips

Add these lines to the script:
```deluge
info "Bill map: " + bill;
info "Bill ID: " + billId;
info "Organization ID: " + ORGANIZATION_ID;
```

## References

- [Zoho Books Automation Help](https://www.zoho.com/us/books/help/settings/automation.html)
- [Creating Custom Functions](https://www.zoho.com/us/books/kb/automation/custom-function.html)
