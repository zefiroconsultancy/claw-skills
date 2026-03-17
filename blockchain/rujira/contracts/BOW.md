# BOW (AMM Liquidity Pools)

Automated market maker that provides liquidity to FIN order books via XY=K pools with LP token issuance.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/bow/](https://gitlab.com/thorchain/rujira/-/tree/main/packages/rujira-rs/src/interfaces/bow)

## Execute Messages

### swap

Market swap through the AMM pool.

```json
{
  "swap": {
    "min_return": { "denom": "btc-btc", "amount": "950000" },
    "to": null,
    "callback": null
  }
}
```

| Field      | Type         | Required | Description                                        |
| ---------- | ------------ | -------- | -------------------------------------------------- |
| min_return | Coin         | Yes      | Minimum return coin (denom + amount) or tx reverts |
| to         | string       | No       | Recipient address (defaults to sender)             |
| callback   | CallbackData | No       | Binary callback data for composability             |

Funds: Exactly one coin of either pool denom

### deposit

Deposit liquidity and receive LP tokens. Supports single-sided and dual-sided deposits.

```json
{ "deposit": { "min_return": "1000000", "callback": null } }
```

| Field      | Type             | Required | Description                  |
| ---------- | ---------------- | -------- | ---------------------------- |
| min_return | Uint128 (string) | No       | Minimum LP shares to receive |
| callback   | CallbackData     | No       | Binary callback data         |

Single-sided deposits incur a swap fee on the imbalanced portion.

Funds: One or both pool denoms

### withdraw

Burn LP tokens to withdraw underlying assets proportionally.

```json
{ "withdraw": { "callback": null } }
```

| Field    | Type         | Required | Description          |
| -------- | ------------ | -------- | -------------------- |
| callback | CallbackData | No       | Binary callback data |

Funds: LP token (pool receipt denom)

## Query Messages

### strategy

Query the pool's strategy configuration and current state.

```json
{ "strategy": {} }
```

Response: `{"xyk": [{x, y, step, min_quote, fee}, {x, y, k, shares}]}`

### quote

Request a price quote from the AMM (used internally by FIN).

```json
{
  "quote": {
    "min_price": null,
    "offer_denom": "btc-btc",
    "ask_denom": "rune",
    "data": null
  }
}
```

Response: `{price: Decimal, size: Uint128, data: Binary|null}`

## Sudo Messages (Governance)

- `SetStrategy(Strategies)` -- replace the pool strategy (currently only Xyk)

## Key Types

- **Strategies**: `{"xyk": {x, y, step, min_quote, fee}}` -- XY=K pool config
  - `x`, `y`: pool denom strings
  - `step`: Decimal -- percentage of pool deployed per quote iteration (1-1000 bps)
  - `min_quote`: Uint128 -- minimum balance required to return a quote
  - `fee`: Decimal -- swap fee percentage (0-1000 bps)
- **XykState**: `{x, y, k, shares}` -- current pool balances and invariant
- **Coin**: `{"denom": "btc-btc", "amount": "1000000"}` -- standard Cosmos coin
- **CallbackData**: Binary (base64-encoded)

## Examples

Deposit dual-sided liquidity (1 BTC + 450 RUNE):

```bash
thornode tx wasm execute <bow_btc_rune> \
  '{"deposit":{"min_return":"1000000"}}' \
  --amount 100000000btc-btc,45000000000rune --from mykey --gas auto --gas-adjustment 1.5
```

Withdraw LP tokens:

```bash
thornode tx wasm execute <bow_btc_rune> \
  '{"withdraw":{}}' \
  --amount 5000000factory/thor1bowaddr.../lp --from mykey --gas auto --gas-adjustment 1.5
```
