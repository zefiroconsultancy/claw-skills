---
name: beancount-reports
description: >
  Visualize beancount ledger data as chart images (.png). Takes CSV output
  from bean-query and produces matplotlib charts for delivery via Telegram
  or other channels. Depends on the beancount skill for data extraction.
triggers:
  - beancount chart
  - beancount report
  - ledger chart
  - expense chart
  - financial report chart
  - visualize expenses
  - plot expenses
  - spending chart
version: 1
---

# Beancount Report Visualization

## Dependencies

- **beancount skill** — for data extraction via bean-query
- **telegram-delivery skill** — for file delivery path mapping
- **matplotlib** — for chart generation
- **bean-query CLI** — for CSV export

## Pipeline

```
bean-query → CSV → matplotlib → PNG → MEDIA: (host path)
```

## Setup

Ensure beancount, beanquery, and matplotlib are installed:
```bash
bean-query --help && python3 -c "import matplotlib; print(matplotlib.__version__)"
```

## Chart Style

Dark theme matching Telegram dark mode:

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

plt.rcParams.update({
    'figure.facecolor': '#1a1a2e',
    'axes.facecolor': '#16213e',
    'axes.edgecolor': '#e0e0e0',
    'axes.labelcolor': '#e0e0e0',
    'text.color': '#e0e0e0',
    'xtick.color': '#e0e0e0',
    'ytick.color': '#e0e0e0',
    'grid.color': '#2a2a4a',
    'grid.alpha': 0.5,
    'font.size': 11,
    'figure.dpi': 150,
})

COLORS = ['#e94560','#0f3460','#533483','#48c9b0','#f39c12',
          '#3498db','#e74c3c','#2ecc71','#9b59b6','#1abc9c']
```

## Parsing bean-query CSV

bean-query CSV embeds the currency in the value column (e.g. `7593.84 EUR`).
Strip it before plotting:

```python
import re

def parse_amount(s):
    """Extract numeric value from bean-query CSV field like ' 7593.84 EUR'"""
    return float(re.sub(r'[^\d.\-]', '', s.strip().split()[0]))

def short(acct):
    """Shorten account names for chart labels"""
    for prefix in ['Expenses:', 'Income:', 'Assets:', 'Liabilities:']:
        acct = acct.replace(prefix, '')
    return acct
```

## Standard Report Suite

All reports save to `/workspace/ledger/reports/` and deliver via host path
`/var/lib/claw/hermes/test/.hermes/ledger/reports/`.

### 1. Balance Sheet (horizontal bar)
- **Data**: `SELECT account, sum(cost(position)) as balance WHERE account ~ 'Assets|Liabilities' GROUP BY account ORDER BY account`
- **Chart**: horizontal bar, green for positive, red for negative
- **Title**: includes net worth total

### 2. Expense Categories (donut)
- **Data**: `SELECT root(account, 2) as category, sum(cost(position)) as total WHERE account ~ 'Expenses' GROUP BY category ORDER BY sum(cost(position)) DESC`
- **Chart**: donut with center total, legend on right

### 3. Monthly Expenses (bar)
- **Data**: `SELECT month, sum(cost(position)) as total WHERE account ~ 'Expenses' GROUP BY month ORDER BY month`
- **Chart**: vertical bars with average line

### 4. Monthly Net / Savings (bar)
- **Data**: `SELECT month, sum(cost(position)) as net WHERE account ~ 'Income|Expenses' GROUP BY month ORDER BY month`
- **Chart**: vertical bars, green for saved (negative), red for overspent (positive)
- **Note**: in beancount, income is negative, so negative net = saved money

### 5. Top Payees (horizontal bar)
- **Data**: `SELECT payee, count(position) as txns, sum(cost(position)) as total WHERE account ~ 'Expenses' AND payee != '' GROUP BY payee ORDER BY sum(cost(position)) DESC LIMIT 10`
- **Chart**: horizontal bar with value labels

### 6. Income Sources (donut)
- **Data**: `SELECT account, sum(cost(position)) as total WHERE account ~ 'Income' GROUP BY account ORDER BY sum(cost(position))`
- **Chart**: donut, filter to only rows where total < 0 (actual income)
- **Note**: use abs() for display values

## Procedure: Full Report Generation

1. Export CSVs from bean-query (all 6+ queries above)
2. Run matplotlib script to generate PNGs
3. Deliver each PNG using host path:
   ```
   MEDIA:/var/lib/claw/hermes/test/.hermes/ledger/reports/01_balance_sheet.png
   ```

## Output Directory

```
/workspace/ledger/reports/        ← sandbox path (for generation)
  01_balance_sheet.png
  02_expense_categories.png
  03_monthly_expenses.png
  04_monthly_net.png
  05_top_payees.png
  06_income_sources.png
  *.csv                           ← raw data files
```

## Pitfalls

- **Always use `matplotlib.use('Agg')`** — no display server in sandbox
- **bean-query CSV values include currency** — must parse with `parse_amount()`
- **Month column is numeric** — map to names (1→Jan, 2→Feb, etc.)
- **Income is negative in beancount** — flip sign for display, green for negative net
- **Deliver via host path** — see telegram-delivery skill for mapping
- **`figure.dpi: 150`** — good balance of quality vs file size for Telegram
- **`plt.tight_layout()` + `bbox_inches='tight'`** — prevents label clipping
