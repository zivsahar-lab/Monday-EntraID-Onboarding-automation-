# n8n Monday.com to Microsoft Entra ID Onboarding Automation

This project contains an n8n workflow template for a simple employee onboarding lab:

1. Watch for a user creation request.
2. Create a Monday.com ticket.
3. Poll Monday.com for approved tickets.
4. Create a Microsoft Entra ID user through Microsoft Graph.
5. Mark the Monday.com ticket as done.

The workflow files are sanitized for GitHub. Secrets, tenant values, board IDs, column IDs, emails, and n8n credential references were replaced with placeholders.

## Workflow Files

| File | Purpose |
| --- | --- |
| `n8n-monday-onboarding.json` | Creates a Monday.com ticket from an approved Outlook email request. |
| `n8n-entra-user-creation.json` | Finds approved Monday.com tickets and creates Microsoft Entra ID users. |

## Architecture

```text
Outlook Email
    |
    v
n8n Workflow 1
    |
    v
Monday.com Ticket
    |
    v
Approval Status Updated in Monday.com
    |
    v
n8n Workflow 2
    |
    v
Microsoft Graph API
    |
    v
Microsoft Entra ID User Created
```

## Requirements

- n8n instance
- Microsoft Outlook account connected to n8n
- Monday.com account and board
- Microsoft Entra ID tenant
- Azure app registration with Microsoft Graph permissions
- Monday.com API token or Monday.com n8n credential

## Required Microsoft Graph Permissions

The Azure app registration needs application permissions that allow user creation.

Typical lab permission:

```text
User.ReadWrite.All
```

Admin consent is required for application permissions.

## Placeholders To Replace

Before importing or running the workflows, replace the placeholder values below.

| Placeholder | Description |
| --- | --- |
| `REPLACE_WITH_YOUR_N8N_OUTLOOK_CREDENTIAL_ID` | Your n8n Outlook credential ID. |
| `REPLACE_WITH_YOUR_OUTLOOK_CREDENTIAL_NAME` | Your n8n Outlook credential name. |
| `REPLACE_WITH_ALLOWED_REQUESTER_EMAIL@example.com` | Email address allowed to submit onboarding requests. |
| `REPLACE_WITH_APPROVER_EMAIL@example.com` | Email address that receives approval notifications. |
| `REPLACE_OR_REGENERATE_WITH_N8N_OUTLOOK_WEBHOOK_ID` | n8n-generated Outlook webhook ID, if needed. |
| `REPLACE_WITH_YOUR_MONDAY_API_TOKEN` | Monday.com API token. Do not commit a real token. |
| `REPLACE_WITH_YOUR_MONDAY_BOARD_ID` | Monday.com board ID used for onboarding tickets. |
| `REPLACE_WITH_YOUR_MONDAY_APPROVAL_STATUS_COLUMN_ID` | Monday.com column ID for the approval status. |
| `REPLACE_WITH_YOUR_MONDAY_REQUEST_STATUS_COLUMN_ID` | Monday.com column ID for the request completion status. |
| `REPLACE_WITH_APPROVED_STATUS_VALUE` | Monday.com status index/value that means approved. |
| `REPLACE_WITH_NOT_DONE_STATUS_VALUE` | Monday.com status index/value that means not completed. |
| `REPLACE_WITH_YOUR_N8N_MONDAY_CREDENTIAL_ID` | Your n8n Monday.com credential ID. |
| `REPLACE_WITH_YOUR_MONDAY_CREDENTIAL_NAME` | Your n8n Monday.com credential name. |
| `REPLACE_WITH_YOUR_AZURE_TENANT_ID` | Microsoft Entra tenant ID. |
| `REPLACE_WITH_YOUR_AZURE_APP_CLIENT_ID` | Azure app registration client ID. |
| `REPLACE_WITH_YOUR_AZURE_APP_CLIENT_SECRET` | Azure app registration client secret. Do not commit a real secret. |
| `REPLACE_WITH_YOUR_ENTRA_TENANT_DOMAIN.onmicrosoft.com` | Your Entra tenant domain. |
| `REPLACE_WITH_TEMP_PASSWORD_OR_GENERATE_SECURELY` | Temporary password used for the created account. Prefer generating this dynamically. |
| `REMOVED_N8N_INSTANCE_ID` | Removed n8n instance metadata. No action required. |

## Monday.com Board Setup

Create a Monday.com board with at least these columns:

| Column | Purpose |
| --- | --- |
| Item name | Stores the requested user's name or request title. |
| Approval status | Used to decide whether the user should be created. |
| Request status | Used to prevent the same request from being processed again. |

Example status labels:

```text
Approval Status: Pending, Approved, Rejected
Request Status: Not Done, Done
```

Update the workflow placeholders with the actual board ID, column IDs, and status values from your Monday.com board.

## Import Steps

1. Import `n8n-monday-onboarding.json` into n8n.
2. Import `n8n-entra-user-creation.json` into n8n.
3. Create or reconnect the Outlook credential in n8n.
4. Create or reconnect the Monday.com credential in n8n.
5. Replace all placeholder values.
6. Configure your Azure app registration and client secret.
7. Test both workflows with fake lab users before enabling them.

## Security Notes

Do not commit real API tokens, client secrets, tenant-specific IDs, credential IDs, personal emails, or temporary passwords.

For a real environment, avoid hardcoding secrets in HTTP Request nodes. Prefer n8n credentials, environment variables, or a secret manager.

For temporary passwords, prefer generating a unique random password per user instead of using a static value.

## Known Implementation Notes

The second workflow's Monday.com API request returns the GraphQL response as a nested object. The `Extract Monday Items` node converts `data.boards[0].items_page.items` into individual n8n items before the `If` node checks the request status.

The GraphQL query returns `id`, `name`, and `column_values` so the downstream `If` node can safely check whether the request is already marked `Done`.

## Disclaimer

This is a lab automation template. Review permissions, logging, error handling, password handling, and approval controls before adapting it for production.
