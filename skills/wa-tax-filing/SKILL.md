# /wa-tax-filing -- WA State Sales Tax Filing from Shopify Export

Prepare a Washington State Department of Revenue (DOR) sales tax filing from a Shopify sales export CSV. Computes B&O tax, sales tax by city, foreign (out-of-state) sales, and flags anomalies.

## Invocation

```
/wa-tax-filing <path to Shopify sales CSV>
```

## Prerequisites

Export your Shopify sales data:

### Option 1: ShopifyQL (recommended)

Go to Shopify Admin > Analytics > Reports > **Total Sales over time** > click **Query** and paste:

```sql
FROM sales
  SHOW gross_sales, discounts, returns, net_sales, shipping_charges, return_fees,
    taxes, total_sales, shipping_city, shipping_postal_code, shipping_region,
    sales_channel
  GROUP BY day, sale_id, order_name, product_title_at_time_of_sale,
    shipping_city, shipping_postal_code, shipping_region, sales_channel WITH TOTALS
  SINCE startOfQuarter(-1q) UNTIL endOfQuarter(-1q)
  ORDER BY day ASC
  LIMIT 1000
VISUALIZE total_sales TYPE table
```

Adjust the `-1q` values to target the quarter you need:
- Last quarter: `startOfQuarter(-1q)` / `endOfQuarter(-1q)`
- Two quarters ago: `startOfQuarter(-2q)` / `endOfQuarter(-2q)`

Export the results as CSV.

### Option 2: Manual export

1. Go to Shopify Admin > Analytics > Reports > **Sales by order**
2. Set the date range to the quarter you're filing (e.g., Jan 1 - Mar 31)
3. Add columns: **Shipping city**, **Shipping postal code**, **Shipping region**
4. Export as CSV

### Required columns

The CSV must contain these columns:
- `Day`, `Sale ID`, `Order name`, `Product title at time of sale`
- `Gross sales`, `Discounts`, `Returns`, `Net sales`, `Shipping charges`, `Return fees`, `Taxes`, `Total sales`
- `Shipping city`, `Shipping postal code`, `Shipping region`
- `Sales channel` (critical for identifying marketplace vs direct orders)

## Configuration

Before first use, create a `config.local.md` file in this skill's directory with your business details:

```markdown
- **Entity**: Your Business Name / Your LLC Name (WA)
- **Nexus states**: Washington (list all states where you have nexus)
- **Shopify order prefix**: #YOURPREFIX (used to identify direct orders vs marketplace orders)
- **Marketplace channels**: Etsy, Amazon, etc. (platforms that handle their own tax remittance)
```

If no config file is found, the skill will prompt you for this information.

## Context

- **B&O Tax Rate**: Retailing = 0.471% (applied to ALL net sales, all states — it's a gross receipts tax)
- **WA DOR filing site**: https://dor.wa.gov (My DOR)
- **Filing deadlines**: Q1 = Apr 30, Q2 = Jul 31, Q3 = Oct 31, Q4 = Jan 31
- **Late penalties**: 5% (≤1 month late), 15% (2+ months late), 25% (3+ months late)

## Workflow

### Step 1: Load Configuration

Check for `config.local.md` in this skill's directory. If not found, ask the user:
1. What is your business entity name?
2. Which states do you have sales tax nexus in?
3. What is your Shopify order name prefix (e.g., `#MYSTORE`)?
4. Do you sell on marketplaces (Etsy, Amazon, Walmart) that remit their own taxes?

### Step 2: Read and Parse the CSV

Read the CSV file. Aggregate line items by `Order name`. For each order, compute totals for:
- Gross sales, Discounts, Returns, Net sales, Shipping charges, Taxes, Total sales
- Shipping city, zip, and region (state)

### Step 3: Identify and Exclude Marketplace Orders

Use the `Sales channel` column to identify marketplace orders:

1. List all unique sales channels found in the data with order counts and net sales totals.
2. Exclude orders from known marketplace channels (e.g., "QuickSync for Etsy", "Amazon", "Walmart Marketplace"). Marketplaces collect and remit their own sales tax — the merchant never receives that tax money.
3. If an unfamiliar channel name appears, ask the user whether it is a marketplace or a direct channel before proceeding.
4. Keep orders from direct channels: "Online Store", "Draft Orders", "Shop", "POS", and similar.

If the CSV does not contain a `Sales channel` column, ask the user to re-export using the ShopifyQL query in Prerequisites.

### Step 4: Anomaly Check — Tax Collected Outside Nexus States

After excluding marketplace orders, check remaining orders for tax collected on non-nexus state shipments. These are errors (employee mistakes, Shopify misconfiguration, etc.). Flag them for the user but do NOT exclude them from the net sales / B&O calculation — only the tax amount is anomalous.

### Step 5: Compute Filing Numbers

Calculate and report:

**Summary Table:**
| Category | Amount |
|----------|--------|
| Total Net Sales (all states) | sum of all order net sales |
| Foreign (out-of-WA) Sales | sum of net sales where region ≠ Washington |
| WA Taxable Sales | sum of net sales where region = Washington |
| WA Sales Tax Collected | sum of taxes where region = Washington |

Verify: Foreign Sales + WA Taxable Sales = Total Net Sales.

**WA City-by-City Breakdown:**

Group all Washington orders by city. For each city report:
- City name, Zip code, Taxable Sales (net sales), Tax Collected, Effective Rate

Sort alphabetically by city. Show totals row. Confirm city total matches WA Taxable Sales from summary.

**Sales by State (reference):**

Group all orders by state. Show: Orders, Net Sales, Tax Collected. Sort by net sales descending.

**International vs Domestic:** If there are international (non-US) orders, break them out separately from domestic out-of-state ("foreign") sales.

### Step 6: Compute Taxes Due

| Line | Calculation |
|------|-------------|
| Retailing B&O Tax | 0.471% x Total Net Sales (all states) |
| WA Sales Tax | = WA Sales Tax Collected (pass-through) |
| Subtotal | B&O + Sales Tax |

If the filing is late (current date is past the deadline for the quarter), estimate:
- Penalty tier based on how many months late
- Interest at ~1% per month from the due date

Provide DOR location codes for each WA city/zip where known. Common codes:

| Zip(s) | City | Location Code |
|--------|------|---------------|
| 98004-98008 | Bellevue | 1703 |
| 98011-98012, 98021 (King Co) | Bothell | 1702 |
| 98021 (Snohomish Co) | Bothell | 2907 |
| 98028 | Kenmore | 1718 |
| 98033-98034 | Kirkland | 1719 |
| 98052-98053 | Redmond | 1731 |
| 98074 | Sammamish | 1735 |
| 98101-98199 | Seattle | 1726 |
| 98294 | Sultan | 3119 |

Note: Always verify location codes at https://dor.wa.gov/tax-rate-lookup as they can change.

### Step 7: Output

Present the filing in this format:

```
## Q[X] [YEAR] — WA DOR Filing

### Summary
[Summary table: Total, Foreign, WA Taxable, WA Tax Collected]

### WA Sales — City Breakdown
[City-by-city table with taxable sales, tax collected, effective rate]

Check: Foreign + WA = Total ✓

### Taxes Due
[B&O + Sales Tax + penalties/interest if late]

### Anomalies
[Tax collected outside nexus states]
[Unusual order patterns]

### Action Items
- File at dor.wa.gov
- Fix Shopify tax settings if anomalies found
- Next quarter deadline: [date]
```

## Notes

- All amounts in USD
- B&O tax applies to ALL net sales regardless of customer location — it is a gross receipts tax on the business, not a consumer sales tax
- WA Sales Tax is reported by location (city/zip) of the customer's shipping address
- Orders with $0 net sales (fully discounted or fully returned) can be excluded from the breakdown
- Returns reduce net sales in the period they occur, even if the original sale was in a prior quarter
- This skill prepares the filing numbers — you still need to log into My DOR and enter them manually
- This is not tax advice. Consult a tax professional for questions about nexus, exemptions, or complex situations
