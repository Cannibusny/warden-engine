# ManyChat Webhook Template — CNY Lead Capture

## Setup Instructions

### Step 1: Create Custom Fields in ManyChat
Go to **Settings > Custom Fields** and create these 6 fields (if they don't already exist):

| Field Name   | Type  | Description                              |
|-------------|-------|------------------------------------------|
| `first_name` | Text  | Lead's first name (auto-populated by IG) |
| `last_name`  | Text  | Lead's last name (auto-populated by IG)  |
| `phone`      | Phone | Lead's phone number                      |
| `email`      | Email | Lead's email address                     |
| `interest`   | Text  | Product/service they're interested in    |
| `source`     | Text  | Which IG post/story triggered the lead   |

### Step 2: Add an External Request Action in Your Flow
In ManyChat's Flow Builder, add an **External Request** (Action) step with these settings:

```
Request Type:   POST
URL:            https://YOUR-N8N-DOMAIN/webhook/manychat/lead
```

**Headers:**
```
Content-Type: application/json
```

**Body (JSON):**
```json
{
  "first_name": "{{first_name}}",
  "last_name": "{{last_name}}",
  "phone": "{{phone}}",
  "email": "{{email}}",
  "interest": "{{interest}}",
  "source": "{{source}}"
}
```

Replace `{{field_name}}` with the actual ManyChat custom field variables from the dropdown.

### Step 3: Set the Source Field
Before the External Request step, add a **Set Custom Field** action:
- Set `source` to the name of the post/story/trigger  
  Example: `"Instagram Story — June 12 Launch"`

This lets you track which content drove each lead.

### Step 4: Set the Interest Field
Use one of these approaches:
- **Static:** Set `interest` to a fixed value per flow (e.g., "VIP Table Package")
- **Dynamic:** Use a Quick Reply or User Input step to ask the subscriber what they want, then save their response to the `interest` field

---

## What Happens After the Webhook Fires

1. ManyChat sends the lead data to the n8n webhook
2. The **ManyChat Lead Capture Agent** workflow transforms the data and forwards it to the **Warden Engine** intake endpoint
3. Warden creates a PENDING transaction and emails Sheridan an approval request
4. Sheridan clicks **Approve** or **Deny** in the email
5. On Approve: Warden routes to the Google_Sheets service branch and appends the lead to the **CNY Pre-Launch List** sheet

### Columns Written to Google Sheet
| Column       | Value                                      |
|-------------|---------------------------------------------|
| First Name  | From ManyChat `first_name`                  |
| Last Name   | From ManyChat `last_name`                   |
| Phone       | From ManyChat `phone`                       |
| Email       | From ManyChat `email`                       |
| Interest    | From ManyChat `interest`                    |
| Source      | From ManyChat `source`                      |
| Date Added  | Auto-generated ISO timestamp                |
| Status      | "New" (default)                             |

---

## Current Webhook URL (Tunnel — Temporary)

```
POST https://622fa465b063-tunnel-m04ggedz.devinapps.com/webhook/manychat/lead
```

Basic Auth: `user` / `803919d74c8d4a4b4344e368efb82390`

> **Note:** This URL is temporary (Devin tunnel). Once Railway deployment is live, the URL will change to the permanent Railway domain. Update the ManyChat External Request URL at that time.

---

## Testing Without ManyChat

You can test the webhook manually with curl:

```bash
curl -X POST https://user:803919d74c8d4a4b4344e368efb82390@622fa465b063-tunnel-m04ggedz.devinapps.com/webhook/manychat/lead \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Test",
    "last_name": "Lead",
    "phone": "+18455550000",
    "email": "test@example.com",
    "interest": "VIP Table Package",
    "source": "Manual Test"
  }'
```

Expected response:
```json
{
  "success": true,
  "message": "Lead submitted to Warden for approval",
  "transaction_id": "uuid-here",
  "status": "PENDING",
  "lead": "Test Lead",
  "interest": "VIP Table Package"
}
```
