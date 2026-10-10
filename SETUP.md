# Setup Guide

This guide explains how to configure the Lead Management Automation workflow in n8n.

## Requirements

Before starting, you need:

- An n8n account or self-hosted n8n instance
- A Google account
- Google Sheets access
- Gmail access
- The workflow JSON file from this repository

## 1. Import the Workflow

1. Open n8n.
2. Create a new workflow.
3. Choose **Import from File**.
4. Select:

   `lead-management-public-demo.json`

5. Open the imported workflow.

The workflow contains the complete automation structure, but external credentials must be connected manually.

## 2. Create the Google Sheet

Create a Google Spreadsheet for lead management.

Create two sheets:

- `Leads`
- `Invalid Leads`

### Leads columns

Create the following columns in the first row:

```text
name
email
phone
source
location
message
created_at
updated_at
assigned_to
assigned_email
status

Invalid Leads columns
Create the following columns:
name
email
phone
source
location
message
created_at
updated_at
status
invalid_reason

3. Connect Google Sheets
In n8n:
1. Open each Google Sheets node.
2. Create or select your Google Sheets credential.
3. Select your spreadsheet.
4. Select the correct sheet:
   - Leads
   - Invalid Leads
5. Verify that the column mappings match your spreadsheet.
The public workflow intentionally does not contain private Google Sheet IDs or credentials.
4. Connect Gmail
Open the node:
Send Assignment Email
Then:
1. Create or select your Gmail credential.
2. Authorize your Google account.
3. Verify the recipient expression uses:
{{ $json.assigned_email }}

The email notification will be sent to the sales representative assigned by the workflow.
5. Configure Sales Representatives
The demo routing configuration is:
USA -> Alex
Europe -> Maria
Other locations -> General Sales

The example email addresses inside the public workflow use safe placeholder addresses such as:
alex@example.com
maria@example.com
sales@example.com

Replace them with the real email addresses used by your organization.
6. Configure the Webhook
Open the node:
Receive Lead
The webhook path is:
lead-form

After publishing the workflow, n8n will generate a Production Webhook URL.
Use that URL from your website, form, CRM, application, or another automation.
Example request:
curl -i -X POST "YOUR_PRODUCTION_WEBHOOK_URL" \
-H "Content-Type: application/json" \
-d '{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+15550100001",
  "source": "Website",
  "location": "USA",
  "message": "Interested in your service"
}'

A valid lead should return approximately:
{
  "success": true,
  "status": "processed",
  "message": "Lead processed successfully"
}

with HTTP status:
200

7. Test Validation
The workflow requires at least an email address or phone number.
Example invalid request:
curl -i -X POST "YOUR_PRODUCTION_WEBHOOK_URL" \
-H "Content-Type: application/json" \
-d '{
  "name": "Invalid Lead",
  "source": "Website",
  "location": "USA",
  "message": "Validation test"
}'

Expected result:
{
  "success": false,
  "status": "invalid",
  "message": "Lead validation failed",
  "reason": "Missing email and phone"
}

with HTTP status:
400

The invalid lead will also be stored in the Invalid Leads sheet.
8. Publish the Workflow
After all credentials and spreadsheet settings are configured:
1. Save the workflow.
2. Publish or activate the workflow.
3. Copy the Production Webhook URL.
4. Send a valid test lead.
5. Send an invalid test lead.
6. Check:
   - n8n Executions
   - Google Sheets
   - Gmail notifications
Expected Workflow
A valid new lead follows approximately:
Webhook
-> Normalize
-> Validate
-> Find Existing Lead
-> Create or Update Lead
-> Route by Location
-> Assign Sales Representative
-> Update Google Sheets
-> Send Assignment Email
-> Return HTTP 200

An invalid lead follows:
Webhook
-> Normalize
-> Validate
-> Mark as Invalid
-> Save Invalid Lead
-> Return HTTP 400

Security Notes
Do not commit real credentials, API keys, private spreadsheet IDs, production webhook URLs, or customer information to a public repository.
The workflow included in this repository is a sanitized public demo intended for portfolio and demonstration purposes.

## Optional Error Handling

For production use, configure your own n8n Error Workflow for error notifications and monitoring.