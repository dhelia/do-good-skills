# /financial-report -- Generate Interactive HTML Financial Report

Generate a self-contained HTML financial report from categorized bank and credit card statements. Replaces QuickBooks with a local, interactive, drillable report — no subscription, no login, no vendor lock-in.

## Invocation

```
/financial-report <path to directory containing categorized CSVs or bank statement CSVs>
```

Best used after running `/tax-prep` to categorize transactions.

## What It Produces

A single self-contained HTML file (no external dependencies, no server needed) with 7 interactive tabs:

| Tab | What it shows |
|-----|---------------|
| **P&L** | Full Profit & Loss statement — click any line to drill into transactions |
| **By Category** | Every expense/income category with count and total — click to see transactions |
| **By Vendor** | All vendors ranked by spend with visual bars — click to see their transactions |
| **By Account** | Each bank/credit card account with credits, debits, and net |
| **All Transactions** | Every transaction, searchable and filterable by account and type |
| **Schedule C** | IRS Schedule C line-by-line totals |
| **TurboTax** | TurboTax-matching categories with 2024 vs 2025 comparison and breakdowns |

### Features
- Click any P&L line item to see every transaction in that category
- Click any vendor to see all payments to them
- Search and filter across all transactions
- Color-coded badges: Income (green), COGS (yellow), Expense (red), Excluded (gray), Personal (pink)
- Self-contained — works offline, no internet needed, just open the HTML file in a browser
- All data embedded in the file — no external JSON or API calls
- Print-friendly for audit documentation

## Prerequisites

Run `/tax-prep` first to categorize all transactions. The output of `/tax-prep` provides:
- Categorized transaction CSVs with columns: Date, Account, Description, Amount, Category
- Or a master ledger CSV with all transactions

Alternatively, provide raw bank/CC statement CSVs and this skill will run categorization first.

## Configuration

Create a `config.local.md` in this skill's directory:

```markdown
## Business Info
- **Entity name**: Your Business Name
- **DBA**: Your DBA Name (if different)
- **Tax year**: 2025
- **Accounting method**: Cash

## Prior Year (for TurboTax comparison)
Provide prior year TurboTax numbers for side-by-side comparison:
- Vehicle: $X,XXX
- Home office: $X,XXX
- Communications: $X,XXX
- Advertising: $X,XXX
- Meals: $XXX
- Legal: $XXX
- Travel: $XXX
- Office expenses: $X,XXX
- Interest: $X,XXX
- Taxes & licenses: $X,XXX
- Insurance: $XXX
- Repairs: $XXX
- Equipment rental: $X,XXX
- Building rental: $X,XXX
- Inventory/COGS: $XXX,XXX
- Contract labor: $X,XXX
- Other misc: $XX,XXX
- Income: $XXX,XXX

## Home Office
- **Method**: Simplified
- **Square footage**: 250
- **Rate**: $5/sq ft

## Accounts
List each account label as it should appear in the report:
- Chase Checking XXXX
- Chase CC XXXX
- Amex XXXX
- etc.
```

## Workflow

### Step 1: Load and Validate Data

1. Read all categorized transaction CSVs from the provided directory
2. Each transaction needs: Date, Account, Description, Amount, Category, Group (Income/COGS/Expenses/Excluded/Personal)
3. If transactions aren't categorized, prompt the user to run `/tax-prep` first
4. Extract vendor names from descriptions for the vendor view

### Step 2: Compute Summary Numbers

Calculate and verify:
- **Income**: Sum of all Income-group transactions
- **COGS**: Sum of all COGS-group transactions
- **Gross Profit**: Income + COGS (COGS is negative)
- **Operating Expenses**: Sum of all Expense-group transactions
- **Other Income/Expenses**: Sum of Other Income/Expense groups
- **Home Office**: From config (simplified method: sq ft × $5, max $1,500)
- **Net Profit/Loss**: Gross Profit + Expenses + Other + Home Office

Cross-check: Total of all non-excluded transactions should equal Net Profit/Loss + Home Office.

### Step 3: Build Category-to-TurboTax Mapping

Map internal categories to TurboTax line items. The standard mapping:

| TurboTax Category | Internal Categories |
|---|---|
| Other self-employed income | Sales, Gross Revenue |
| Vehicle | Vehicle gas & fuel, Parking & tolls |
| Home office | (calculated, not from transactions) |
| Communications | Telecommunication, Phone, Slack, Google Workspace |
| Advertising | Advertising, Google Ads, Meta Ads, SEMRush |
| Meals (50% limit) | Meals |
| Legal and professional fees | Legal, CPA, Attorney, Filing fees |
| Business travel | Travel, Flights, Hotels, Rideshare |
| Office expenses | Software & SaaS, Office supplies |
| Credit card, loan, and other interest | Interest, Shopify Capital interest, LOC interest |
| Taxes and licenses | State taxes, B&O, Business licenses |
| Business insurance | Insurance |
| Repairs and maintenance | Repairs, Hardware stores |
| Equipment rental | U-Haul, Equipment leases |
| Building or land rental | Warehouse rent, Storage |
| Inventory | COGS (all sub-categories) |
| Contract labor | Contractors, Temp labor |
| Commissions & fees | Platform fees, Payment processing |
| Other miscellaneous expenses | Shipping, Bank fees, Education, Fulfillment |

For each TurboTax line, also prepare a breakdown showing what's inside (e.g., "Other miscellaneous" breaks into Shipping $89K + Software $8K + Bank fees $1K).

### Step 4: Generate HTML Report

Generate a single self-contained HTML file with embedded data and JavaScript. The file must:

1. **Embed all transaction data as JSON** directly in a `<script>` tag — do NOT use `fetch()` or external files (browsers block local file access via fetch)
2. **Include all CSS inline** — no external stylesheets
3. **Include all JavaScript inline** — no external libraries or CDN
4. **Be fully functional when opened via `file://`** — test this

#### HTML Structure

```
<!DOCTYPE html>
<html>
<head>
  <style>/* All CSS inline */</style>
</head>
<body>
  <header>Business Name — Tax Year Financial Report</header>
  <nav>Tabs: P&L | By Category | By Vendor | By Account | All Transactions | Schedule C | TurboTax</nav>
  <main>
    <div id="stats">Summary cards: Revenue, COGS, Gross Profit, Net Income</div>
    <div id="main-content">Active tab content</div>
    <div id="detail-panel">Drill-down transaction list (hidden until clicked)</div>
  </main>
  <script>
    const DATA = [/* ALL transactions embedded here as JSON array */];
    // Tab rendering functions
    // Click-to-drill-down handlers
    // Search and filter functions
  </script>
</body>
</html>
```

#### Required Tab Implementations

**P&L Tab:**
- Show Income section with clickable line items
- Show COGS section with clickable line items
- Show Gross Profit subtotal
- Show Expenses section with clickable line items (sorted by amount)
- Show Net Income/Loss total
- Clicking any line opens a detail panel showing all transactions in that category

**By Category Tab:**
- Table: Category | Group (badge) | Transaction Count | Total
- Sorted by total amount
- Searchable
- Each row clickable to show transactions

**By Vendor Tab:**
- Table: Vendor | Category | Transaction Count | Total
- Visual bar showing relative spend
- Sorted by total amount
- Searchable
- Each row clickable to show transactions

**By Account Tab:**
- Table: Account | Transaction Count | Credits | Debits | Net
- Each row clickable to show all transactions for that account

**All Transactions Tab:**
- Table: Date | Account | Description | Category | Type (badge) | Amount
- Searchable text input
- Dropdown filter by account
- Dropdown filter by type (Income/COGS/Expenses/Excluded/Personal)
- Sorted by date descending

**Schedule C Tab:**
- Table: Line | Description | Amount
- Matching IRS Schedule C line numbers
- Bold totals for Lines 5, 7, 31

**TurboTax Tab:**
- Table: TurboTax Category | Prior Year | Current Year
- Each row clickable to expand breakdown OR drill into transactions
- Breakdowns show sub-categories with amounts
- Net Profit/Loss total at bottom
- Numbers must match the verified Schedule C totals exactly

#### Detail Panel (Drill-Down)

When any line item, category, or vendor is clicked:
- Hide the main content
- Show a "← Back" button
- Show a table of all matching transactions: Date | Account | Description | Vendor | Amount
- Show the total at the top
- Sorted by date ascending

#### Styling Guidelines

- Clean, professional look (no bright colors, no gradients)
- Monospace font for all amounts
- Green for positive/income, red for negative/expenses
- Hover effects on clickable rows
- Responsive — works on laptop screens
- Print-friendly (consider @media print)

### Step 5: Verify Numbers

Before saving the file, verify:
1. P&L net income matches Schedule C Line 31
2. TurboTax total matches Schedule C
3. Sum of all category totals equals the net
4. No transactions are double-counted
5. Excluded and Personal transactions are excluded from all totals

### Step 6: Output

Save the HTML file to the tax year directory:
```
[tax_year_directory]/audit_backup/report.html
```

Tell the user to open it with:
```
open [path]/report.html
```

## Notes

- The HTML file is typically 300-500KB depending on transaction count (data is embedded)
- Works in all modern browsers (Chrome, Safari, Firefox, Edge)
- No server needed — opens directly from the filesystem
- All data stays local — nothing is sent to any server
- This replaces QuickBooks for small businesses that just need P&L visibility and tax filing support
- The report is read-only — it's a snapshot, not a live accounting system
- Generate a new report each time you update categorizations
- This is not accounting software and not tax advice — have a CPA review before filing
