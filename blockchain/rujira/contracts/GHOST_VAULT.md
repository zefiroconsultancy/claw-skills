# Ghost Vault (Ghost Lending)

Lending vault that accepts deposits of a single asset and lends to whitelisted market contracts, earning interest for depositors.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/ghost/vault/](https://gitlab.com/thorchain/rujira/-/tree/main/packages/rujira-rs/src/interfaces/ghost/vault)

## Execute Messages

### deposit

Deposit the borrowable asset into the lending pool and receive receipt tokens representing pool share.

```json
{ "deposit": { "callback": null } }
```

| Field    | Type         | Required | Description                                  |
| -------- | ------------ | -------- | -------------------------------------------- |
| callback | Binary\|null | No       | Opaque callback data forwarded after deposit |

Funds: Send the vault's deposit denom (e.g. `usdc`)

### withdraw

Withdraw the borrowable asset from the lending pool by returning receipt tokens.

```json
{ "withdraw": { "callback": null } }
```

| Field    | Type         | Required | Description                                     |
| -------- | ------------ | -------- | ----------------------------------------------- |
| callback | Binary\|null | No       | Opaque callback data forwarded after withdrawal |

Funds: Send receipt tokens (pool share tokens)

### market (whitelisted contracts only)

Borrow or repay assets. Only callable by contracts registered via `SetBorrower` sudo.

#### market.borrow

```json
{
  "market": {
    "borrow": { "amount": "1000000", "callback": null, "delegate": null }
  }
}
```

| Field    | Type             | Required | Description                                 |
| -------- | ---------------- | -------- | ------------------------------------------- |
| amount   | String (Uint128) | Yes      | Amount to borrow                            |
| callback | Binary\|null     | No       | Callback data forwarded with borrowed funds |
| delegate | String\|null     | No       | Address to allocate the debt obligation to  |

Funds: None

#### market.repay

```json
{ "market": { "repay": { "delegate": null } } }
```

| Field    | Type         | Required | Description                                 |
| -------- | ------------ | -------- | ------------------------------------------- |
| delegate | String\|null | No       | Repay a delegate's debt instead of caller's |

Funds: Send the vault's denom to repay

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{denom: String, interest: Interest, fee_address: String, fee: Decimal}`

### status

```json
{ "status": {} }
```

Response: `{last_updated: Timestamp, utilization_ratio: Decimal, debt_rate: Decimal, lend_rate: Decimal, debt_pool: PoolResponse, deposit_pool: PoolResponse}`

### borrower

```json
{ "borrower": { "addr": "thor1..." } }
```

Response: `{addr: String, denom: String, limit: Uint128, current: Uint128, shares: Uint128, available: Uint128}`

### delegate

```json
{ "delegate": { "borrower": "thor1...", "addr": "thor1..." } }
```

Response: `{borrower: BorrowerResponse, addr: String, current: Uint128, shares: Uint128}`

### borrowers

```json
{ "borrowers": { "limit": 10, "start_after": null } }
```

Response: `{borrowers: [BorrowerResponse]}`

## Sudo Messages (Governance)

- **SetBorrower** - Register a market contract with a borrow limit: `{set_borrower: {contract, limit}}`
- **SetInterest** - Update the interest rate curve: `{set_interest: {target_utilization, base_rate, step1, step2}}`
- **SetFee** - Update fee rate and destination: `{set_fee: [Decimal, String]}`

## Key Types

### Interest

```json
{ "target_utilization": "0.8", "base_rate": "0", "step1": "1", "step2": "2" }
```

Two-slope interest rate curve. Below `target_utilization`, rate scales by `step1`. Above, rate accelerates by `step2`.

### PoolResponse

```json
{ "size": "1000000", "shares": "950000", "ratio": "0.95" }
```

Share-based pool tracking deposits or debt. `ratio = shares / size`.

## Examples

Deposit 100 USDC into a Ghost lending vault:

```bash
thornode tx wasm execute <ghost_vault_usdc> \
  '{"deposit":{"callback":null}}' \
  --amount 10000000000eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 \
  --from mykey --gas auto --gas-adjustment 1.5
```

Check vault status (utilization, rates):

```bash
thornode query wasm contract-state smart <ghost_vault_usdc> '{"status":{}}'
```

Withdraw by returning receipt tokens:

```bash
thornode tx wasm execute <ghost_vault_usdc> \
  '{"withdraw":{"callback":null}}' \
  --amount 10000000000factory/thor1vaultaddr.../receipt \
  --from mykey --gas auto --gas-adjustment 1.5
```
