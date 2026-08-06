---
name: shiplab
description: >
  Expert skill for retrieving and analyzing shipping data from the Shiplab platform.
  Use this skill whenever the user asks about their shipping invoices, charges, costs,
  carriers, or wants to export or analyze postal data from Shiplab. Trigger even when
  the user phrases it casually — e.g. "show me my invoices", "how much did we spend on
  shipping last month", "download my charges", "get UPS data", "compare carrier costs",
  or any question about shipments, parcel costs, freight bills, or postal analytics.
  Always use this skill for Shiplab-related data requests, even if the user doesn't
  mention Shiplab by name.
---

# Shiplab Data Skill

Shiplab is a platform that aggregates postal and shipping invoices from carriers (like UPS, FedEx, DHL, etc.) for companies and freight brokers. This skill guides you through retrieving, exporting, and analyzing that data via the Shiplab MCP tools.

## Understanding the Data Model

Two core data types:

- **Invoices** — one row per invoice. Contains totals: `invoice_gross_amount`, `invoice_net_amount`, `invoice_package_count`, `invoice_due_date`, etc.
- **Charges** — one row per charge line on an invoice. Contains shipment-level detail: `tracking_number`, `charge_description`, `charge_amount`, `service_type`, etc.

Use **invoices** for high-level financial summaries. Use **charges** for shipment-level analysis (e.g., which packages cost the most, breakdown by service type).

## Workflow: Exporting Data

Exports are async — the file is built in the background and downloaded from S3. Always follow these steps:

### Step 1 — Understand what the user needs

Before exporting, clarify:
- **Date range**: start and end date (format: YYYY-MM-DD)
- **Carrier filter** (optional): e.g., "UPS", "FedEx" — must match an active carrier in the account
- **Company/connector** (optional): if the user has multiple companies, ask which one. Use `get_connector_groups` to list them.
- **What fields**: only request what's needed — unnecessary fields make exports slower and larger

If the user hasn't specified a date range, ask for it. Don't assume.

### Step 2 — Start the export

Call `export_invoices` or `export_charges` with:
- `start_date` and `end_date` (YYYY-MM-DD)
- `fields`: only the columns needed (see field lists below)
- `export_format`: always `"csv"` unless the user explicitly asks for JSONL
- `carrier` (optional filter)
- `company_id` or `credential_id` (optional filters)

**Invoice fields available:**
`credential_id`, `company_id`, `carrier`, `file_format`, `invoice_number`, `invoice_date`, `file_format_id`, `invoice_file_format`, `parent_account_number`, `account_number`, `invoice_currency`, `invoice_gross_amount`, `invoice_net_amount`, `invoice_due_date`, `invoice_package_count`, `exception`, `created_at`, `updated_at`

**Charge fields available (core):**
`id`, `credential_id`, `company_id`, `carrier`, `invoice_number`, `invoice_date`, `file_format_id`, `invoice_file_format`, `tracking_number`, `charge_description`, `charge_amount`, `charge_amount_currency`, `service_type`, `created_at`, `updated_at`

**Charge fields (address/zone — only include if user needs them):**
`zone`, `shipper_city`, `shipper_state`, `shipper_country`, `shipper_zip`, `recipient_city`, `recipient_state`, `recipient_country`, `recipient_zip`

### Step 3 — Poll until complete

Call `get_export_status(report_id)` every few seconds. Status values:
- `PENDING` / `RUNNING` → keep polling
- `COMPLETED` → proceed to step 4
- `FAILED` → tell the user and show `error_message`

Large exports can take several minutes. Let the user know you're waiting.

### Step 4 — Give the user the download URL

Call `get_export_download_url(report_id)` and give the user the `download_url` field (NOT `direct_download_url` — it's a huge S3 URL). The download link streams the CSV file directly and requires no authentication.

## Workflow: Account Discovery

Before exporting, you may need to identify what's in the account:

- `get_profile` — who is the authenticated user
- `get_connector_groups` — list companies/groups
- `get_connectors` — list all carrier connectors
- `list_exports` — view recent export history

## Analytical Guidance

When the user asks for analysis (not just raw data), help them think about:

- **Cost breakdown by carrier**: export charges filtered by `carrier`
- **Month-over-month trends**: export invoices for multiple months, compare `invoice_gross_amount` totals
- **Service type mix**: use charges with `service_type` to see ground vs. express split
- **Exception tracking**: use invoices with `exception` field to flag problematic invoices
- **Per-shipment cost**: charges give you `charge_amount` per `tracking_number`

## Common Requests

| User says | What to do |
|---|---|
| "Show me my invoices for last month" | `export_invoices` with last month's dates |
| "How much did I spend on UPS?" | `export_charges` filtered by `carrier="UPS"` |
| "Download my charges" | `export_charges` with core fields, give CSV link |
| "What carriers are connected?" | `get_connectors` |
| "Compare FedEx and UPS costs" | Two `export_charges` calls, one per carrier |
| "Show recent exports" | `list_exports` |

## Output Format

- Always give the user a **CSV download link**
- Don't fetch or read CSV contents into the conversation — exports exist so large data stays out of model context
- For analytical summaries, suggest opening the CSV in Excel for pivot tables
