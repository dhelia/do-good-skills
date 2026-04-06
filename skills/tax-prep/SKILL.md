# /tax-prep -- Small Business Tax Prep from Bank Statements

Categorize business expenses from bank and credit card statements, generate QuickBooks-importable .qbo files, and produce a summary P&L. Built for single-member LLCs and sole proprietors who run e-commerce businesses.

## Invocation

```
/tax-prep <path to bank/credit card statement CSV(s)>
```

## Prerequisites

Download CSV exports of all business account activity for the tax year:

- **Chase Business Checking**: chase.com > Activity > Download > CSV
- **Chase Credit Cards**: chase.com > Activity > Download > CSV
- **Amex Credit Cards**: americanexpress.com > Statements & Activity > Download > CSV
- **Other banks**: Download CSV or OFX from your bank's website

Each CSV should cover the full tax year (Jan 1 - Dec 31).

## Configuration

Before first use, create a `config.local.md` file in this skill's directory:

```markdown
## Business Info
- **Entity name**: Your Business Name
- **Entity type**: Single-member LLC / Sole Prop / S-Corp / Partnership
- **Tax year**: 2025
- **State**: WA (or your state)
- **Industry**: E-commerce / Retail / Services / etc.

## Accounts
List each account with a label:
- **Chase Checking (primary)**: Main operating account — all Shopify income deposits here
- **Chase Credit Card 1**: Business expenses
- **Chase Credit Card 2**: Business expenses
- **Amex Credit Card**: Business expenses
- **Line of Credit**: Business line of credit (interest is deductible)
- **Shopify Capital**: Merchant cash advance / loan (interest portion is deductible)

## Income Sources
- **Shopify**: Direct deposits to checking — this is gross revenue minus Shopify fees
- **Etsy**: If applicable — deposits to checking
- **Other**: Any other income sources

## Expense Categories (Chart of Accounts)
If you have an existing QuickBooks chart of accounts, paste it here.
Otherwise, the skill will use the standard Schedule C categories below.

## Inventory Method
- **Method**: FIFO / LIFO / Average Cost
- **Track COGS**: Yes / No
- **Inventory purchases identified by**: Wire transfers to suppliers, vendor names, etc.

## Notes
- **Personal expenses on business cards**: Rare / Frequent (will be flagged for exclusion)
- **Home office**: Yes / No
- **Contractors**: List any regular contractors paid via Zelle/Venmo/check
```

## Workflow

### Step 1: Load Configuration and Statements

1. Read `config.local.md` for business context.
2. Read each CSV file provided. Auto-detect the format:
   - **Chase checking/credit CSV columns**: `Transaction Date`, `Post Date`, `Description`, `Category`, `Type`, `Amount`, `Memo`
   - **Amex CSV columns**: `Date`, `Description`, `Amount`, `Extended Details`, `Category`
   - **Generic CSV**: Look for date, description, and amount columns
3. Normalize all transactions into a common format: `Date`, `Description`, `Amount`, `Account`, `Raw Category`

### Step 2: Classify Transactions

Categorize every transaction into Schedule C categories using vendor name pattern matching. The standard categories are:

**Income:**
| Category | Schedule C Line | Common Vendors/Patterns |
|----------|----------------|------------------------|
| Gross Revenue | Line 1 | Shopify, Etsy, Amazon deposits, Stripe, PayPal |
| Returns & Refunds | Line 2 | Shopify refund, chargeback |

**Cost of Goods Sold (COGS):**
| Category | Schedule C Line | Common Vendors/Patterns |
|----------|----------------|------------------------|
| Inventory Purchases | Line 36 | Wire transfers to manufacturers/suppliers, Alibaba, trade suppliers |
| Freight & Shipping Inbound | Line 36 | Freight forwarder, DHL, container shipping, customs, duty |
| Packaging & Materials | Line 36 | Uline, packaging suppliers |

**Operating Expenses:**
| Category | Schedule C Line | Common Vendors/Patterns |
|----------|----------------|------------------------|
| Advertising | Line 8 | Meta/Facebook Ads, Google Ads, TikTok, Amazon Ads, influencer payments |
| Car & Truck | Line 9 | Gas, tolls, parking (business use %) |
| Commissions & Fees | Line 10 | Shopify fees, Etsy fees, payment processing, marketplace fees |
| Contract Labor | Line 11 | Zelle to contractors, Fiverr, Upwork |
| Insurance | Line 15 | Business insurance, product liability, general liability |
| Interest - Mortgage | Line 16a | Home office mortgage interest (% of home used) |
| Interest - Other | Line 16b | Line of credit interest, Shopify Capital interest, credit card interest |
| Legal & Professional | Line 17 | Attorney, CPA, accountant, legal services, tax prep |
| Office Expense | Line 18 | Office supplies, printer ink, paper, desk supplies |
| Rent/Lease - Equipment | Line 20a | Equipment leases |
| Rent/Lease - Property | Line 20b | Warehouse rent, storage unit, co-working |
| Repairs & Maintenance | Line 21 | Equipment repair, website maintenance |
| Shipping & Postage (Outbound) | Line 22 | USPS, UPS, FedEx, DHL outbound, postage |
| Supplies | Line 22 | Business supplies not fitting other categories |
| Taxes & Licenses | Line 23 | Business license, state B&O tax, sales tax remitted, permits |
| Travel | Line 24a | Flights, hotels, Airbnb (business travel) |
| Meals | Line 24b | Restaurants, DoorDash, UberEats (50% deductible, business purpose) |
| Utilities | Line 25 | Internet, phone (business % only), electricity (home office %) |
| Software & SaaS | Line 27a (Other) | Shopify subscription, apps, Klaviyo, QuickBooks, Canva, Adobe |
| Fulfillment / 3PL | Line 27a (Other) | 3PL fees, fulfillment center charges, ShipBob, ShipStation |
| Bank & Merchant Fees | Line 27a (Other) | Monthly bank fees, wire fees, ATM fees |
| Education & Training | Line 27a (Other) | Courses, books, conferences (business-related) |
| Miscellaneous | Line 27a (Other) | Anything that doesn't fit above |

**Non-Deductible / Excluded:**
| Category | Reason |
|----------|--------|
| Personal Expense | Not business-related — flag and exclude |
| Transfer Between Accounts | Not income or expense — internal movement |
| Credit Card Payment | Already captured as individual charges on the card |
| Loan Principal | Principal repayment is not deductible (only interest is) |
| Owner Draw / Distribution | Not an expense for single-member LLC |

### Step 3: Vendor Pattern Matching

Build a vendor-to-category mapping from transaction descriptions. Common patterns:

```
# Income
"SHOPIFY" → Gross Revenue
"ETSY" → Gross Revenue
"STRIPE" → Gross Revenue

# COGS
"WIRE TRANSFER" + large amount + known supplier → Inventory Purchases
"DHL" / "FREIGHT" / "CUSTOMS" / "DUTY" → Freight & Shipping Inbound
"ULINE" → Packaging & Materials

# Advertising
"FACEBK" / "FB ADS" / "META" → Advertising
"GOOGLE ADS" / "GOOGLE *ADS" → Advertising
"TIKTOK" → Advertising

# Fees
"SHOPIFY *" (small recurring) → Software & SaaS
"SQ *" / "SQUARE" → Commissions & Fees
"PAYPAL" → Commissions & Fees

# Fulfillment
"SHIPBOB" / "SHIPSTATION" / "DELIVERR" → Fulfillment / 3PL

# Software
"KLAVIYO" / "CANVA" / "ADOBE" / "QUICKBOOKS" / "INTUIT" → Software & SaaS
"AMAZON WEB SERVICES" / "AWS" → Software & SaaS
"OPENAI" / "ANTHROPIC" → Software & SaaS

# Shipping outbound
"USPS" / "UPS" / "FEDEX" → Shipping & Postage

# Interest
"INTEREST CHARGE" → Interest - Other
"SHOPIFY CAPITAL" + interest → Interest - Other

# Transfers & non-deductible
"TRANSFER" / "PAYMENT THANK YOU" → Transfer Between Accounts
"ATM WITHDRAWAL" → Flag for review (personal?)
"ZELLE" → Flag for review (contractor or personal?)
```

For any transaction that can't be confidently classified:
1. Flag it as "NEEDS REVIEW"
2. Show the description and amount
3. Ask the user to classify it

### Step 4: Handle Special Items

**Shopify Capital:**
- Separate principal from interest. Only interest is deductible.
- If the user provides a Shopify Capital statement, parse it. Otherwise, ask for the total interest paid in the tax year.

**Line of Credit:**
- Interest charges are deductible (Interest - Other).
- Principal payments are not deductible (exclude).

**Credit Card Interest:**
- If the business credit card carries a balance, interest charges are deductible.

**Transfers Between Accounts:**
- Identify and exclude: checking → credit card payments, checking → savings, internal transfers.
- These are not income or expenses.

**Owner Draws:**
- Transfers from business checking to personal accounts are owner distributions, not expenses.
- Identify and exclude.

**Inventory / COGS:**
- For proper COGS tracking: Beginning Inventory + Purchases - Ending Inventory = COGS
- Ask the user for beginning and ending inventory values if tracking COGS properly.
- Inventory purchases (wire transfers to suppliers) go to COGS, not operating expenses.

**Home Office Deduction:**
- Two methods: Simplified ($5/sq ft, max 300 sq ft = $1,500) or Regular (% of home expenses)
- Ask the user which method they prefer and the square footage
- If regular method: need mortgage interest, property tax, insurance, utilities, repairs — proportioned by office %

### Step 5: Generate Outputs

#### 5a: Transaction Ledger (CSV)

Export a CSV with all categorized transactions:
```
Date, Description, Amount, Account, Category, Schedule C Line, Notes, Flagged
```

#### 5b: Summary P&L

Generate a Profit & Loss statement:
```
INCOME
  Gross Revenue                    $XXX,XXX
  Returns & Refunds                ($X,XXX)
  ─────────────────────────────────────────
  Net Revenue                      $XXX,XXX

COST OF GOODS SOLD
  Inventory Purchases              $XX,XXX
  Freight & Shipping Inbound       $X,XXX
  Packaging & Materials            $X,XXX
  ─────────────────────────────────────────
  Total COGS                       $XX,XXX

GROSS PROFIT                       $XX,XXX

OPERATING EXPENSES
  Advertising                      $X,XXX
  Commissions & Fees               $X,XXX
  Contract Labor                   $X,XXX
  Fulfillment / 3PL                $X,XXX
  Insurance                        $XXX
  Interest                         $X,XXX
  Legal & Professional             $XXX
  Office Expense                   $XXX
  Shipping & Postage (Outbound)    $X,XXX
  Software & SaaS                  $X,XXX
  Taxes & Licenses                 $X,XXX
  Utilities                        $XXX
  Other Expenses                   $XXX
  ─────────────────────────────────────────
  Total Operating Expenses         $XX,XXX

NET PROFIT (LOSS)                  $XX,XXX
```

#### 5c: QuickBooks .qbo File

Generate a .qbo file (OFX format) for each account. The .qbo format is XML-based:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?OFX OFXHEADER="200" VERSION="220" SECURITY="NONE" OLDFILEUID="NONE" NEWFILEUID="NONE"?>
<OFX>
  <SIGNONMSGSRSV1>
    <SONRS>
      <STATUS>
        <CODE>0</CODE>
        <SEVERITY>INFO</SEVERITY>
      </STATUS>
      <DTSERVER>[YYYYMMDDHHMMSS]</DTSERVER>
      <LANGUAGE>ENG</LANGUAGE>
    </SONRS>
  </SIGNONMSGSRSV1>
  <BANKMSGSRSV1>
    <STMTTRNRS>
      <TRNUID>[unique-id]</TRNUID>
      <STATUS>
        <CODE>0</CODE>
        <SEVERITY>INFO</SEVERITY>
      </STATUS>
      <STMTRS>
        <CURDEF>USD</CURDEF>
        <BANKACCTFROM>
          <BANKID>[routing-number-or-placeholder]</BANKID>
          <ACCTID>[account-label]</ACCTID>
          <ACCTTYPE>CHECKING</ACCTTYPE>
        </BANKACCTFROM>
        <BANKTRANLIST>
          <DTSTART>[start-date]</DTSTART>
          <DTEND>[end-date]</DTEND>
          <!-- One STMTTRN per transaction -->
          <STMTTRN>
            <TRNTYPE>DEBIT</TRNTYPE>
            <DTPOSTED>[YYYYMMDD]</DTPOSTED>
            <TRNAMT>-[amount]</TRNAMT>
            <FITID>[unique-transaction-id]</FITID>
            <NAME>[description]</NAME>
            <MEMO>[category]</MEMO>
          </STMTTRN>
        </BANKTRANLIST>
        <LEDGERBAL>
          <BALAMT>0</BALAMT>
          <DTASOF>[end-date]</DTASOF>
        </LEDGERBAL>
      </STMTRS>
    </STMTTRNRS>
  </BANKMSGSRSV1>
</OFX>
```

For credit cards, use `CREDITCARDMSGSRSV1` and `CCSTMTTRNRS` instead of `BANKMSGSRSV1`.

The MEMO field contains the expense category, which QuickBooks can use for auto-categorization rules.

### Step 6: Review and Finalize

1. Present the summary P&L to the user.
2. Show all flagged/ambiguous transactions and ask for classification.
3. After user confirms, generate final .qbo files.
4. Provide Schedule C line-by-line totals ready for tax filing.

## Output Format

```
## [Tax Year] Tax Prep — [Business Name]

### Accounts Processed
[List of accounts with transaction counts]

### Flagged Items (Needs Review)
[Transactions that couldn't be auto-classified]

### Summary P&L
[Profit & Loss statement]

### Schedule C Summary
[Line-by-line totals matching IRS Schedule C]

### Files Generated
- [account]_categorized.csv — Full transaction ledger
- [account].qbo — QuickBooks import file
```

## Notes

- All amounts in USD
- This skill categorizes expenses but does not provide tax advice
- Always have a CPA or tax professional review before filing
- The .qbo file imports into QuickBooks Desktop and QuickBooks Online
- QuickBooks may prompt you to map categories on first import — match MEMO fields to your chart of accounts
- For single-member LLCs, business income is reported on Schedule C of your personal 1040
- Keep all bank statements as backup documentation
- Contractor payments over $600/year to any single person require a 1099-NEC filing

### Categorization Learnings

- **Credit card payments to suppliers**: When the business uses credit cards to pay suppliers, those charges appear on the CC statement as vendor-specific prefixes. Don't miscategorize supplier payments as office expenses just because they're on a credit card.
- **Amazon purchases on Amex/CC**: For a product business, Amazon purchases on business credit cards are likely COGS (customer replacement parts), not office supplies. ASK the user before categorizing.
- **PayPal payments**: May be rent (warehouse/storage services) — don't assume all PayPal transactions are supplier payments or fees. Check the recipient.
- **Zelle payments**: Need careful review — could be contractors, suppliers, truckers, CPA, repairs, or personal. Always flag for user review with context.
- **Temp labor platforms** (e.g., Instawork): These platforms handle their own employment taxes (W-2s, withholding). Classify as Contract Labor (Line 11), not wages.
- **Shopify Capital interest**: Interest is a lump-sum factor fee charged at funding, not accrued over time. On cash basis, it's deductible when paid (i.e., when the loan is funded). For loans funded in prior years, the interest was already deductible in that year — do not double count. Request loan-by-loan statements from Shopify, not just annual summaries.
- **Line of Credit**: Only interest is deductible, principal repayment is not. LOC advances/draws are NOT income — exclude them from revenue entirely.

### Vendor Classification CSV

- When generating the vendor classification CSV for user review, include a **"Section"** column to separate "Business or Personal?" items (ambiguous) from "Likely Personal" items (high confidence personal).

### TurboTax Category Mapping

- Use TurboTax-specific category names when generating TurboTax-ready output: Vehicle, Home office, Communications, Advertising, Meals, Legal and professional fees, Business travel, Office expenses, Credit card/loan/other interest, Taxes and licenses, Business insurance, Repairs and maintenance, Equipment rental, Building or land rental, Inventory/COGS, Contract labor, Commissions & fees, Other miscellaneous expenses.
- **Keep categories consistent with prior year filing** to avoid IRS red flags from big year-over-year swings.
- **Communications** should be broken out from Software — includes phone services, VoIP platforms, messaging/collaboration tools (e.g., Slack, Google Workspace), and virtual phone numbers.
- **Equipment rental** should be broken out from Car & Truck — truck/equipment rentals (e.g., U-Haul) are equipment rental, not vehicle expenses.

### Tax Rules to Remember

- **Passive activity loss rules**: Rental losses are passive and suspended if AGI > $150K. They release (become deductible) on sale/disposition of the rental property.
- **Home office deduction**: Cannot increase a net loss — it can only bring you to $0. Any excess carries forward.
- **Vehicle expenses**: Require a mileage log to claim. Do not claim vehicle deduction without one.
- **Meals**: Enter the full amount paid. TurboTax auto-applies the 50% limitation — do not pre-halve the amount.
