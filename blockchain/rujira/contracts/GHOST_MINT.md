# Ghost Mint (Ghost Mint Credit Pool)

Credit pool that mints synthetic assets on demand for whitelisted market contracts, tracking debt with interest.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/ghost/mint/](https://gitlab.com/thorchain/rujira/-/tree/main/packages/rujira-rs/src/interfaces/ghost/mint)

## Execute Messages

### market (whitelisted contracts only)

Mint or burn synthetic assets. Only callable by contracts registered via `SetBorrower` sudo.

#### market.borrow

Mint new tokens for the borrower.

```json
{
  "market": {
    "borrow": { "amount": "1000000", "callback": null, "delegate": null }
  }
}
```

| Field    | Type             | Required | Description                                |
| -------- | ---------------- | -------- | ------------------------------------------ |
| amount   | String (Uint128) | Yes      | Amount to mint                             |
| callback | Binary\|null     | No       | Callback data forwarded with minted funds  |
| delegate | String\|null     | No       | Address to allocate the debt obligation to |

Funds: None

#### market.repay

Burn tokens to repay debt.

```json
{ "market": { "repay": { "delegate": null } } }
```

| Field    | Type         | Required | Description                                 |
| -------- | ------------ | -------- | ------------------------------------------- |
| delegate | String\|null | No       | Repay a delegate's debt instead of caller's |

Funds: Send the minted denom to burn/repay

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{denom: String, interest: Interest, fee_address: String, fee: Decimal, mint_cap: Uint128}`

### status

```json
{ "status": {} }
```

Response: `{last_updated: Timestamp, utilization_ratio: Decimal, debt_rate: Decimal, debt_pool: PoolResponse}`

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
- **SetMintCap** - Update the maximum mintable supply: `{set_mint_cap: "1000000000"}`

## Key Types

### Interest

```json
{ "target_utilization": "0.8", "base_rate": "0", "step1": "1", "step2": "2" }
```

Two-slope interest rate curve. `utilization_ratio = total_minted / mint_cap`.

### PoolResponse

```json
{ "size": "1000000", "shares": "950000", "ratio": "0.95" }
```

Share-based pool tracking debt obligations.
