# Shiplab Plugin for Claude

Access and analyze your Shiplab shipping invoices and charges directly from Claude.

## What it does

- Export shipping invoices and charges as CSV
- Filter by carrier, date range, company
- Get analytical insights on shipping costs
- Works with UPS, FedEx, DHL, and any carrier connected to Shiplab

## Setup

### 1. Set your API key

Before using the plugin, set your Shiplab API key as an environment variable:

```bash
export SHIPLAB_API_KEY=your_api_key_here
```

Or add it to your shell profile (`~/.zshrc` or `~/.bashrc`) to make it permanent.

### 2. Install the plugin

```bash
claude plugin install https://github.com/your-org/shiplab-plugin
```

### 3. Start using it

Just ask Claude naturally:

- "Show me my invoices for last month"
- "How much did we spend on FedEx this quarter?"
- "Download all UPS charges for January 2026"
- "Which carriers are connected to my account?"

## Example prompts

```
دانلود فاکتورهای ماه گذشته
هزینه ارسال UPS در سه ماه اول سال چقدر بوده؟
مقایسه هزینه FedEx و DHL در سال جاری
لیست کانکشن‌های فعال من
```

## Available data

**Invoices:** invoice totals, amounts, package counts, due dates, exceptions

**Charges:** per-shipment charges, tracking numbers, service types, charge descriptions, origin/destination addresses

## Requirements

- Claude Code or Claude Cowork
- Shiplab account with API access
- `SHIPLAB_API_KEY` environment variable set
