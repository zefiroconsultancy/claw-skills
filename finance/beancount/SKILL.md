---
name: beancount
description: >
  Double-entry bookkeeping with Beancount 3.x. Use for any ledger operations:
  creating/editing transactions, opening accounts, running queries, validating
  ledgers, and generating text/CSV reports via bean-query. This is the generic
  tooling skill — domain-specific skills (accounting, wealth management, trading)
  layer on top and handle categorization, parsing, and workflows.
triggers:
  - beancount
  - ledger
  - double-entry
  - transaction
  - accounting entry
  - bean-query
  - balance sheet
  - income statement
  - journal entry
version: 2
---

# Beancount 3.x — Generic Skill

## Environment

- **Beancount**: 3.2.0 (`bean-check`, `bean-query`, `bean-format`, `bean-doctor`, `bean-example`)
- **Ledger path**: `/workspace/ledger/` (persistent volume, survives sessions)
- **Python**: beancount importable as library (`import beancount`)

## Prerequisites

Beancount must be pre-installed. Verify with:
```bash
bean-check --help && bean-query --help
```

## File Conventions

### Main ledger file
`/workspace/ledger/main.beancount` — entry point. Uses `include` directives to split by concern.

### Recommended structure
```
/workspace/ledger/
  main.beancount          # options, plugins, includes
  accounts.beancount      # open/close directives
  commodities.beancount   # commodity declarations
  prices.beancount        # price directives
  YYYY/                   # per-year transaction files
    YYYY-MM.beancount     # per-month or per-quarter
  importers/              # beangulp importer configs
```

### main.beancount template
```beancount
option "title" "Ledger"
option "operating_currency" "EUR"

plugin "beancount.plugins.auto_accounts"

include "commodities.beancount"
include "accounts.beancount"
include "prices.beancount"
include "2026/2026-03.beancount"
```

## Beancount 3.x Syntax Reference

### Account hierarchy (five root types, mandatory)
```
Assets:*        — things you own
Liabilities:*   — things you owe
Income:*        — money coming in (signs are negative by convention)
Expenses:*      — money going out
Equity:*        — opening balances, retained earnings
```

Subaccounts use colons: `Assets:Bank:Checking`, `Expenses:Food:Groceries`

### Directives

**Open account:**
```beancount
2024-01-01 open Assets:Bank:Checking  EUR
```

**Close account:**
```beancount
2026-01-01 close Assets:Bank:OldAccount
```

**Commodity declaration:**
```beancount
1900-01-01 commodity EUR
  name: "Euro"
```

**Transaction:**
```beancount
2026-03-16 * "Payee" "Narration"
  Assets:Bank:Checking       -42.00 EUR
  Expenses:Food:Groceries     42.00 EUR
```

- `*` = cleared, `!` = pending/flagged
- At least two postings; amounts must sum to zero
- One posting amount can be omitted (auto-balanced)
- Payee is optional: `2026-03-16 * "Just a narration"`

**Transaction with metadata:**
```beancount
2026-03-16 * "Amazon" "Electronics"
  ref: "order-123-456"
  Assets:Bank:Checking       -199.99 EUR
  Expenses:Electronics        199.99 EUR
```

**Balance assertion:**
```beancount
2026-03-16 balance Assets:Bank:Checking  1234.56 EUR
```

**Pad (auto-balance to match next balance assertion):**
```beancount
2026-03-01 pad Assets:Bank:Checking Equity:Opening-Balances
2026-03-02 balance Assets:Bank:Checking  5000.00 EUR
```

**Price:**
```beancount
2026-03-16 price BTC  65000.00 EUR
```

**Note:**
```beancount
2026-03-16 note Assets:Bank:Checking "Switched to new account number"
```

**Document:**
```beancount
2026-03-16 document Assets:Bank:Checking "/path/to/statement.pdf"
```

**Event:**
```beancount
2026-03-16 event "location" "Berlin, Germany"
```

### Cost and price syntax
```beancount
; Buying at cost
Assets:Broker:Stocks    10 AAPL {150.00 USD}
Assets:Bank:Checking    -1500.00 USD

; Selling at price (different from cost)
Assets:Bank:Checking     1600.00 USD
Assets:Broker:Stocks    -10 AAPL {150.00 USD} @ 160.00 USD
Income:CapitalGains      -100.00 USD
```

### Tags and links
```beancount
2026-03-16 * "Payee" "Narration" #trip-rome ^invoice-2026-001
  Assets:Bank:Checking    -100.00 EUR
  Expenses:Travel          100.00 EUR
```

## CLI Tools

### Validation (always run after mutations)
```bash
bean-check /workspace/ledger/main.beancount
```
Exit code 0 = valid. Non-zero = errors printed to stderr.

**CRITICAL**: Always run `bean-check` after adding or modifying any entry. Never leave the ledger in an invalid state.

### Queries (bean-query)

bean-query is the primary reporting tool. SQL-like syntax, outputs text tables or CSV.

```bash
bean-query /workspace/ledger/main.beancount "QUERY" [--format text|csv] [-o output.csv]
```

#### Standard Reports

**Net worth:**
```sql
SELECT sum(cost(position)) WHERE account ~ 'Assets|Liabilities'
```

**Balance sheet (all account balances):**
```sql
SELECT account, sum(position) GROUP BY account ORDER BY account
```

**Income statement:**
```sql
SELECT account, sum(cost(position)) WHERE account ~ 'Income|Expenses' GROUP BY account ORDER BY account
```

**Monthly expenses:**
```sql
SELECT month, sum(cost(position)) as total WHERE account ~ 'Expenses' GROUP BY month ORDER BY month
```

**Monthly expenses by category:**
```sql
SELECT month, root(account, 2) as category, sum(cost(position)) as total
WHERE account ~ 'Expenses'
GROUP BY month, category ORDER BY month, category
```

**Monthly income vs expenses (net savings):**
```sql
SELECT month, sum(cost(position)) as net
WHERE account ~ 'Income|Expenses'
GROUP BY month ORDER BY month
```

**Top payees by spend:**
```sql
SELECT payee, count(position) as txns, sum(cost(position)) as total
WHERE account ~ 'Expenses' AND payee != ''
GROUP BY payee ORDER BY sum(cost(position)) DESC LIMIT 15
```

**Expense categories breakdown:**
```sql
SELECT root(account, 2) as category, sum(cost(position)) as total
WHERE account ~ 'Expenses'
GROUP BY category ORDER BY sum(cost(position)) DESC
```

**Transactions for a specific account:**
```sql
SELECT date, payee, narration, position, balance
WHERE account = 'Assets:Bank:Checking'
ORDER BY date
```

**Pending/flagged transactions:**
```sql
SELECT date, payee, narration, position WHERE flag = '!' ORDER BY date
```

**Tagged transactions:**
```sql
SELECT date, payee, narration, position WHERE 'vacation' IN tags ORDER BY date
```

**Year-over-year comparison:**
```sql
SELECT year, root(account, 2) as category, sum(cost(position)) as total
WHERE account ~ 'Expenses'
GROUP BY year, category ORDER BY year, category
```

#### Output formats
- `--format text` — aligned table (default, good for reading)
- `--format csv` — CSV (good for piping to other tools or plotting skills)
- `-o filename.csv` — write directly to file

### Formatting
```bash
bean-format /workspace/ledger/main.beancount
```
Reformats in-place with aligned numbers. **Caution: overwrites the file.**

### Doctor (diagnostics)
```bash
bean-doctor context /workspace/ledger/main.beancount 42
# Shows context around line 42
```

### Example ledger (reference)
```bash
bean-example > /tmp/example.beancount
```

## Procedures

### Adding a transaction
1. Determine the correct accounts and amounts
2. Append the transaction to the appropriate file (current month)
3. Run `bean-check` to validate
4. If errors, fix and re-validate

### Opening a new account
1. Add `open` directive with date and optional currency constraint
2. Place in `accounts.beancount` (or relevant section)
3. Validate with `bean-check`

### Monthly reconciliation
1. Get statement balance from bank
2. Add `balance` assertion for the statement date
3. Run `bean-check` — any discrepancy shows as error
4. Investigate and fix differences

### Importing transactions
1. Use beangulp importers or parse CSV/data manually
2. Format as valid beancount entries
3. Append to appropriate month file
4. Validate with `bean-check`
5. Review for duplicate detection

### Generating a report
1. Choose the appropriate query from the standard reports above
2. Run via `bean-query` with `--format text` for display or `--format csv` for data export
3. For visualization, export CSV and hand off to a plotting skill

## Pitfalls

- **Amounts must balance to zero** — beancount rejects unbalanced transactions
- **Dates are YYYY-MM-DD** — no other format accepted
- **Account names**: ASCII letters, digits, dashes; each component capitalized (`Assets:My-Bank:Checking`)
- **Indentation**: postings must be indented (2+ spaces), directives start at column 0
- **Currency codes**: uppercase, 2-24 chars, must start with a letter
- **One currency per posting** — use cost `{}` or price `@` for conversions
- **Beancount 3.x changes from 2.x**: different plugin API, beangulp replaces old importers, beanquery is separate package
- **bean-format overwrites** — it modifies the file in place
- **Encoding**: UTF-8 only
- **cost() takes position, not number** — `sum(cost(position))` works; `sum(cost(number))` fails in beanquery 0.2.0
- **ORDER BY with aliases** — `ORDER BY total DESC` may fail; use `ORDER BY sum(cost(position)) DESC` instead
- **beanquery is a separate package** — beancount 3.x does not include bean-query; install `beanquery` separately
