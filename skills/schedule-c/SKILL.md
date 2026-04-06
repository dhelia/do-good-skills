# /schedule-c -- Generate Schedule C from Bank Statements

Generate an IRS Schedule C (Profit or Loss from Business) from categorized bank and credit card statement CSVs. Produces line-by-line totals ready for TurboTax or paper filing, with supporting transaction lists for each line item.

## Invocation

```
/schedule-c <path to directory containing bank/CC CSVs>
```

Or if you've already run `/tax-prep`:
```
/schedule-c <path to 2025_taxes directory>
```

## Prerequisites

Bank and credit card statements exported as CSV for the full tax year. Run `/tax-prep` first to categorize transactions, or provide pre-categorized CSVs with a `Category` column.

## Configuration

Create a `config.local.md` in this skill's directory:

```markdown
## Business Info
- **Entity name**: Your Business Name
- **Entity type**: Single-member LLC / Sole Prop / S-Corp
- **Business code**: NAICS code (e.g., 454110 for e-commerce)
- **Tax year**: 2025
- **Accounting method**: Cash / Accrual
- **Business started**: Before [tax year] / During [tax year]

## Home Office
- **Method**: Simplified / Regular
- **Square footage (office)**: 250
- **Square footage (home)**: 2200
- **Simplified rate**: $5/sq ft (max 300 sq ft)

## Interest
- **Shopify Capital**: List loan files with interest amounts
- **Line of Credit**: Total interest paid in tax year
- **Credit card interest**: Will be auto-detected from statements

## COGS
- **Method**: Cash basis (expensed when purchased) / Accrual (beginning + purchases - ending)
- **Beginning inventory**: $0 (if cash basis)
- **Ending inventory**: $0 (if cash basis)
```

## Workflow

### Step 1: Load Categorized Transactions

Read all CSV files in the provided directory. Each transaction should have: Date, Account, Description, Amount, Category.

If running after `/tax-prep`, read the `qb_import_*.csv` files which have the Category column.

If no Category column exists, run categorization using the `/tax-prep` skill first.

### Step 2: Map Categories to Schedule C Lines

Map each category to its Schedule C line:

| Schedule C Line | Categories |
|----------------|------------|
| Line 1 - Gross receipts | Gross Revenue, Sales |
| Line 2 - Returns | Returns & Refunds |
| Line 4 - COGS | Cost of Goods Sold, Inventory Purchases, Customs, Freight |
| Line 6 - Other income | Other Income, Asset Sales |
| Line 8 - Advertising | Advertising, Google Ads, Meta Ads, SEMRush |
| Line 9 - Car & truck | Gas, Tolls, Parking, U-Haul, Car Wash |
| Line 10 - Commissions | Commissions & Fees, Affirm fees, Etsy fees |
| Line 11 - Contract labor | Contract Labor, Instawork, freelancers |
| Line 15 - Insurance | Insurance, Hiscox |
| Line 16b - Interest | Interest (CC, Shopify Capital, LOC) |
| Line 17 - Legal & professional | Legal, CPA, attorney, filing fees |
| Line 18 - Office expense | Office supplies, equipment, Amazon supplies |
| Line 20b - Rent | Warehouse rent, storage |
| Line 21 - Repairs | Repairs & Maintenance, hardware stores |
| Line 23 - Taxes & licenses | State taxes, B&O, business licenses |
| Line 24a - Travel | Flights, hotels, rideshare, CLEAR |
| Line 24b - Meals | Restaurant meals (enter full; 50% deducted) |
| Line 27 - Other | Shipping, Fulfillment, Software, Bank fees, Education |
| Line 30 - Home office | Simplified or regular method |

### Step 3: Generate Supporting Files

For each Schedule C line with a non-zero amount:
1. Create a CSV file named `L[XX]_[Category].csv`
2. Include: Date, Account, Description, Amount
3. Include a TOTAL row at the bottom
4. Sort by date ascending

Also generate:
- `MASTER_LEDGER_[YEAR].csv` — all transactions in one file with categories
- `EXCLUDED_*.csv` — files for transfers, CC payments, loan proceeds, personal expenses
- `PERSONAL.csv` — personal expenses identified and excluded

### Step 4: Generate Schedule C Document

Create `SCHEDULE_C_[YEAR].md` with:
- Part I (Income): Lines 1-7
- Part II (Expenses): Lines 8-28, plus Lines 29-31
- Part III (COGS): Lines 33-42
- Part V (Other Expenses): Line 27 detail breakdown
- Interest detail (Line 16b breakdown by source)
- Home office calculation
- Excluded transactions summary
- Links to supporting CSV files for each line

### Step 5: Generate P&L

Create `PROFIT_AND_LOSS_[YEAR].md` with:
- Income breakdown by channel
- COGS detail
- Gross profit and margin
- Operating expenses by category (sorted by amount descending)
- Net profit/loss
- Key metrics (margins, expense ratios)

## Output

```
[output_directory]/
├── SCHEDULE_C_2025.md          — Complete Schedule C with line references
├── PROFIT_AND_LOSS_2025.md     — P&L statement with metrics
├── MASTER_LEDGER_2025.csv      — Every transaction, categorized
├── L1_Gross_Revenue.csv        — Revenue transactions
├── L2_Returns.csv              — Returns & refunds
├── L4_COGS_Inventory.csv       — Inventory purchases
├── L4_COGS_Customs.csv         — Customs & duties
├── L8_Advertising.csv          — Ad spend detail
├── L9_Car_Truck.csv            — Vehicle expenses
├── L10_Commissions_Fees.csv    — Platform fees
├── L11_Contract_Labor.csv      — Contractor payments
├── ...                         — (one file per line item)
├── L27_Shipping.csv            — Shipping costs
├── L27_Software.csv            — SaaS subscriptions
├── EXCLUDED_CC_Payments.csv    — Credit card payments (not expenses)
├── EXCLUDED_Transfers.csv      — Internal transfers
├── EXCLUDED_Loan_Proceeds.csv  — Loan advances (not income)
├── PERSONAL.csv                — Personal expenses removed
```

## Notes

- All amounts in USD
- For single-member LLCs, Schedule C is filed with your personal 1040
- Net loss offsets other income (W-2, etc.) on your 1040
- No self-employment tax when there is a net loss
- Keep all supporting files for 7 years (IRS statute of limitations is 3 years, but 7 for significant understatement)
- This is not tax advice — have a CPA review before filing
