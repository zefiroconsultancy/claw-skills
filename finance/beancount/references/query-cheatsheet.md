# Beancount Query Language (BQL) Cheatsheet

BQL is SQL-like. Runs via `bean-query LEDGER "QUERY"` or interactively.

## Core columns
- `date` — transaction date
- `account` — full account name
- `payee` — payee string
- `narration` — narration string
- `position` — amount + currency
- `balance` — running balance
- `number` — numeric amount only
- `currency` — currency string
- `cost_number` — cost basis amount
- `cost_currency` — cost basis currency
- `month` — YYYY-MM
- `year` — YYYY
- `flag` — * or !
- `tags` — set of tags
- `links` — set of links

## Functions
- `sum(position)` — aggregate positions
- `cost(position)` — convert to cost basis
- `value(position)` — convert at market price
- `convert(position, "EUR")` — convert to target currency
- `root(account, N)` — first N components of account
- `leaf(account)` — last component
- `parent(account)` — all but last component
- `length(account)` — number of components
- `open_date(account)` — when account was opened
- `close_date(account)` — when account was closed
- `currency_meta(currency)` — commodity metadata

## Filtering
- `WHERE account ~ 'Assets:Bank'` — regex match on account
- `WHERE account = 'Assets:Bank:Checking'` — exact match
- `WHERE payee ~ 'Amazon'` — regex on payee
- `WHERE date >= 2026-01-01` — date range
- `WHERE date >= 2026-01-01 AND date < 2026-04-01` — Q1
- `WHERE flag = '!'` — pending transactions
- `WHERE 'trip-rome' IN tags` — by tag
- `WHERE number > 100` — by amount
- `WHERE currency = 'EUR'` — by currency

## Common Queries

### Net worth
```sql
SELECT sum(cost(position)) WHERE account ~ 'Assets|Liabilities'
```

### Account balances
```sql
SELECT account, sum(position) GROUP BY account ORDER BY account
```

### Monthly expenses
```sql
SELECT month, sum(cost(position)) as total WHERE account ~ 'Expenses' GROUP BY month ORDER BY month
```

### Expenses by category
```sql
SELECT root(account, 2) as category, sum(cost(position)) as total
WHERE account ~ 'Expenses'
GROUP BY category ORDER BY sum(cost(position)) DESC
```

### Top payees by spend
```sql
SELECT payee, count(position) as txns, sum(cost(position)) as total
WHERE account ~ 'Expenses' AND payee != ''
GROUP BY payee ORDER BY sum(cost(position)) DESC
LIMIT 20
```

### Income sources
```sql
SELECT account, sum(cost(position))
WHERE account ~ 'Income'
GROUP BY account ORDER BY account
```

### Transactions for a specific account
```sql
SELECT date, payee, narration, position, balance
WHERE account = 'Assets:Bank:Checking'
ORDER BY date
```

### Uncleared/pending transactions
```sql
SELECT date, payee, narration, position
WHERE flag = '!'
ORDER BY date
```

### Tagged transactions
```sql
SELECT date, payee, narration, position
WHERE 'vacation' IN tags
ORDER BY date
```

### Year-over-year comparison
```sql
SELECT year, root(account, 2) as category, sum(cost(position)) as total
WHERE account ~ 'Expenses'
GROUP BY year, category
ORDER BY year, category
```

## Pitfalls
- `cost()` takes `position`, NOT `number` — `cost(number)` fails in beanquery 0.2.0
- `ORDER BY alias DESC` can fail — use `ORDER BY sum(cost(position)) DESC` with the full expression
- beanquery is a separate package from beancount 3.x — must be installed independently

## Output
- `--format text` — aligned table (default)
- `--format csv` — CSV for processing
- `--format beancount` — beancount directives
- `-o filename.csv` — write to file
