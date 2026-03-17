# Merge (Token Merger)

Merges a legacy token into RUJI with a time-decaying exchange rate -- deposit the merge token during the grace/decay period to receive RUJI shares.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/merge.rs](https://gitlab.com/thorchain/rujira/-/blob/main/packages/rujira-rs/src/interfaces/merge.rs)

## Execute Messages

### deposit

Deposit the merge token to receive RUJI shares. The exchange rate decays linearly from `decay_starts_at` to `decay_ends_at`.

```json
{ "deposit": {} }
```

Funds: Send the `merge_denom` token.

### withdraw

Withdraw RUJI by burning shares.

```json
{ "withdraw": { "share_amount": "1000000" } }
```

| Field        | Type    | Required | Description                         |
| ------------ | ------- | -------- | ----------------------------------- |
| share_amount | Uint128 | Yes      | Number of shares to redeem for RUJI |

Funds: None

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{merge_denom: String, merge_supply: Uint128, ruji_denom: String, ruji_allocation: Uint128, decay_starts_at: Timestamp, decay_ends_at: Timestamp}`

### status

Query total merge progress and remaining allocation.

```json
{ "status": {} }
```

Response: `{merged: Uint128, shares: Uint128, size: Uint128}`

### account

Query a specific account's merge position.

```json
{ "account": { "addr": "thor1..." } }
```

Response: `{addr: String, merged: Uint128, shares: Uint128, size: Uint128}`

## Examples

Deposit merge token to receive RUJI shares:

```bash
thornode tx wasm execute <merge> \
  '{"deposit":{}}' \
  --amount 100000000factory/thor1mergeaddr.../token --from mykey --gas auto --gas-adjustment 1.5
```
