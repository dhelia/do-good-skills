# /audit-prep -- Prepare Audit-Ready Tax Documentation

Generate comprehensive audit-proof documentation from your tax records. Organizes every deduction with supporting transaction lists, creates summary schedules, and flags areas that may attract IRS scrutiny.

## Invocation

```
/audit-prep <path to tax year directory>
```

## Prerequisites

Run `/tax-prep` and `/schedule-c` first. This skill reads the categorized CSVs and Schedule C output to generate audit documentation.

## Workflow

### Step 1: Verify Completeness

Check that all required files exist:
- [ ] Bank statement CSVs for all accounts
- [ ] Credit card statement CSVs for all cards
- [ ] Master ledger with all categorized transactions
- [ ] Schedule C line-item CSVs
- [ ] Shopify Capital / loan documentation
- [ ] Line of credit interest documentation
- [ ] Home office calculation

Flag any missing documentation.

### Step 2: Generate Audit Binder

Create an `audit_binder/` directory with:

```
audit_binder/
├── 00_INDEX.md                    — Table of contents
├── 01_SCHEDULE_C_SUMMARY.md       — Schedule C with line totals
├── 02_INCOME/
│   ├── gross_revenue.csv          — All income transactions
│   ├── returns.csv                — All returns/refunds
│   ├── other_income.csv           — Other income
│   └── revenue_by_channel.md      — Revenue breakdown by source
├── 03_COGS/
│   ├── inventory_purchases.csv    — Supplier payments
│   ├── customs_duties.csv         — Import duties
│   ├── freight.csv                — Shipping inbound
│   ├── replacements.csv           — Customer replacement parts
│   └── cogs_summary.md            — COGS methodology explanation
├── 04_EXPENSES/
│   ├── L08_advertising.csv
│   ├── L09_car_truck.csv
│   ├── L10_commissions.csv
│   ├── L11_contract_labor.csv
│   ├── L15_insurance.csv
│   ├── L16b_interest.csv
│   ├── L17_legal.csv
│   ├── L18_office.csv
│   ├── L20b_rent.csv
│   ├── L21_repairs.csv
│   ├── L23_taxes.csv
│   ├── L24a_travel.csv
│   ├── L24b_meals.csv
│   ├── L27_shipping.csv
│   ├── L27_fulfillment.csv
│   ├── L27_software.csv
│   ├── L27_bank_fees.csv
│   └── L27_other.csv
├── 05_EXCLUDED/
│   ├── cc_payments.csv            — Credit card payments (not double-counted)
│   ├── transfers.csv              — Internal transfers
│   ├── loan_proceeds.csv          — Loan advances
│   ├── shopify_capital.csv        — SC payments (interest in L16b)
│   └── personal.csv               — Personal expenses removed
├── 06_HOME_OFFICE/
│   └── home_office_calc.md        — Method, square footage, calculation
├── 07_INTEREST_DETAIL/
│   ├── shopify_capital_loans.md   — Loan-by-loan interest breakdown
│   ├── loc_interest.md            — Line of credit interest
│   └── cc_interest.md             — Credit card interest charges
├── 08_1099_REVIEW/
│   └── contractor_payments.md     — All payments >$600 to individuals
└── 09_RECONCILIATION/
    └── bank_reconciliation.md     — Total debits + credits vs. Schedule C
```

### Step 3: Audit Risk Assessment

Flag areas that may attract IRS scrutiny:

**High Scrutiny Areas for E-commerce:**
- **Home office deduction**: Ensure exclusive use, document the space
- **Car & truck expenses**: Must have business purpose documentation; consider mileage log
- **Travel**: Must have clear business purpose; document meetings/conferences
- **Meals**: Must have business purpose and who was present; 50% limitation
- **Contract labor**: Verify W-9/W-8BEN on file; 1099 filing compliance
- **Large COGS**: International wire transfers — keep supplier invoices, customs declarations
- **Net loss**: Losses in multiple years may trigger hobby loss rules (Section 183)

For each flagged area, recommend:
1. What documentation to keep
2. What the IRS would ask for
3. How to strengthen the position

### Step 4: Reconciliation Check

Verify that:
1. Total income on Schedule C matches total deposits minus excluded items
2. Total expenses match total debits minus excluded items
3. No double-counting between bank account and credit card (CC payments excluded from checking, individual charges on CC)
4. COGS + operating expenses + excluded = total debits
5. Cross-check major vendor totals (top 10 vendors by spend)

### Step 5: Generate 1099 Review

List all payments to individuals/vendors exceeding $600:
- Name, total amount, payment method
- Whether 1099-NEC was filed
- Whether W-9 is on file
- Foreign contractors: W-8BEN status

### Step 6: Retention Schedule

Recommend what to keep and for how long:
| Document | Retention |
|----------|-----------|
| Tax return | Permanent |
| Schedule C and supporting schedules | 7 years |
| Bank/CC statements | 7 years |
| Receipts for deductions >$75 | 7 years |
| Receipts for deductions <$75 | 3 years |
| Contractor W-9/W-8BEN forms | 4 years after last payment |
| Home office photos/measurements | Duration of deduction |
| Vehicle mileage logs | 7 years |
| Supplier invoices (international) | 7 years |
| Customs/import documentation | 7 years |

## Output Format

```
## Audit Readiness Report — [Tax Year]

### Completeness Check
[✓/✗ for each required document]

### Risk Assessment
[High/Medium/Low risk areas with recommendations]

### Reconciliation
[Bank totals vs Schedule C totals — must balance]

### 1099 Review
[Contractor payment summary and filing status]

### Missing Documentation
[List of items to gather/create]

### Recommendations
[Specific actions to strengthen audit position]
```

## Notes

- The IRS audit rate for Schedule C filers is higher than W-2 employees
- Net losses, high deductions relative to income, and large cash transactions increase audit risk
- The best audit defense is organized, contemporaneous records
- This skill organizes records but does not provide legal or tax advice
- Keep digital copies in addition to physical records
- Consider scanning receipts for expenses >$75 and storing with the transaction files
