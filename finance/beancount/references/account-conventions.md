# Account Naming Conventions

## Rules (enforced by beancount)
- Five root types only: Assets, Liabilities, Income, Expenses, Equity
- Each component starts with uppercase letter
- Components separated by colons
- Allowed chars: letters, digits, dashes
- No spaces, underscores, or special chars in components

## Recommended Hierarchy

### Assets
```
Assets:Bank:<Institution>:<Type>        Assets:Bank:Revolut:Checking
Assets:Cash:<Currency>                  Assets:Cash:EUR
Assets:Broker:<Institution>             Assets:Broker:IBKR
Assets:Broker:<Institution>:<Asset>     Assets:Broker:IBKR:AAPL
Assets:Crypto:<Exchange>                Assets:Crypto:Kraken
Assets:Crypto:<Exchange>:<Coin>         Assets:Crypto:Kraken:BTC
Assets:Receivable:<Entity>              Assets:Receivable:CompanyX
Assets:Property:<Name>                  Assets:Property:Apartment-Berlin
Assets:Prepaid:<Type>                   Assets:Prepaid:Insurance
```

### Liabilities
```
Liabilities:CreditCard:<Name>           Liabilities:CreditCard:Amex
Liabilities:Loan:<Type>                 Liabilities:Loan:Mortgage
Liabilities:Payable:<Entity>            Liabilities:Payable:Landlord
Liabilities:Tax:<Type>                  Liabilities:Tax:IncomeTax-2025
```

### Income
```
Income:Salary:<Employer>                Income:Salary:CompanyX
Income:Freelance:<Client>               Income:Freelance:ClientY
Income:Interest:<Source>                 Income:Interest:BankA
Income:Dividends:<Source>               Income:Dividends:IBKR
Income:CapitalGains                     Income:CapitalGains
Income:Rental:<Property>                Income:Rental:Apartment-Berlin
Income:Gifts                            Income:Gifts
Income:Refunds                          Income:Refunds
```

### Expenses
```
Expenses:Food:Groceries
Expenses:Food:Restaurants
Expenses:Food:Coffee
Expenses:Housing:Rent
Expenses:Housing:Utilities
Expenses:Housing:Electricity
Expenses:Housing:Internet
Expenses:Housing:Maintenance
Expenses:Transport:Public
Expenses:Transport:Fuel
Expenses:Transport:Taxi
Expenses:Transport:CarInsurance
Expenses:Health:Insurance
Expenses:Health:Doctor
Expenses:Health:Pharmacy
Expenses:Health:Gym
Expenses:Entertainment:Subscriptions
Expenses:Entertainment:Movies
Expenses:Entertainment:Books
Expenses:Clothing
Expenses:Education
Expenses:Travel:Flights
Expenses:Travel:Hotels
Expenses:Travel:Activities
Expenses:Financial:Fees
Expenses:Financial:Interest
Expenses:Financial:FX-Loss
Expenses:Insurance:Life
Expenses:Insurance:Home
Expenses:Taxes:Income
Expenses:Taxes:Social
Expenses:Taxes:Property
Expenses:Gifts
Expenses:Misc
```

### Equity
```
Equity:Opening-Balances
Equity:Retained-Earnings
Equity:Conversions                      # for currency conversions
```

## Tips
- Keep hierarchy depth 2-4 levels (deeper gets unwieldy)
- Use consistent naming: pick a convention and stick with it
- Prefix accounts per country/region if multi-country: `Assets:DE:Bank:DKB`
- Close accounts you no longer use (keeps reports clean)
- Use metadata for details that don't belong in the account name
