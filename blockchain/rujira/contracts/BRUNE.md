# FIN (Orderbook DEX)

Central limit order book exchange with limit orders, market swaps, and concentrated liquidity ranges.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/fin/](https://gitlab.com/thorchain/rujira/-/tree/main/packages/rujira-rs/src/interfaces/fin)

## Execute Messages

### swap

Market swap against the order book. Send one of the pair's denoms as funds.

```json
{ "swap": { "min_return": "1000000", "to": null, "callback": null } }
```

| Field        | Type             | Required | Description                                           |
| ------------ | ---------------- | -------- | ----------------------------------------------------- |
| min_return   | Uint128 (string) | No\*     | Minimum return amount or tx reverts                   |
| exact_return | Uint128 (string) | No\*     | Exact return amount or tx reverts                     |
| price        | Decimal (string) | No\*     | Limit price; fills at-or-better, returns unused offer |
| to           | string           | No       | Recipient address (defaults to sender)                |
| callback     | CallbackData     | No       | Binary callback data for composability                |

\*SwapRequest is **untagged** -- the variant is inferred by which fields are present:

- No amount fields = Yolo (swap for whatever is available)
- `min_return` present = Min
- `exact_return` present = Exact
- `price` present = Limit

Funds: Exactly one coin of either pair denom

### order

Manage limit orders: creates, adjusts, and withdraws filled orders in a single atomic action.

```json
{ "order": [[[["base", { "fixed": "95000.5" }, "500000"]], null]] }
```

The payload is a tuple: `(Vec<OrderTarget>, Option<CallbackData>)`.

Each `OrderTarget` is a tuple: `(Side, Price, Option<Uint128>)`.

- Side: `"base"` or `"quote"`
- Price: `{"fixed": "95000.5"}` or `{"oracle": -200}` (basis points offset)
- Uint128: Target offer amount; `null` to withdraw completely

Behavior per target:

1. All filled amounts at matching prices are withdrawn first
2. If no order exists at that price, one is created
3. If an order exists, it is adjusted to match the target amount

Funds: Net change in balances (withdrawals offset new placements)

### range (create)

Create a concentrated liquidity range position.

```json
{
  "range": {
    "create": {
      "config": {
        "high": "96000.0",
        "low": "94000.0",
        "skew": "0.0",
        "spread": "0.005",
        "fee": "0.003"
      },
      "slippage": null
    }
  }
}
```

| Field         | Type                   | Required | Description                               |
| ------------- | ---------------------- | -------- | ----------------------------------------- |
| config.high   | Decimal (string)       | Yes      | Upper price bound                         |
| config.low    | Decimal (string)       | Yes      | Lower price bound                         |
| config.skew   | SignedDecimal (string) | Yes      | Order distribution slope (0 = uniform)    |
| config.spread | Decimal (string)       | Yes      | Bid/ask deviation from XY=K price         |
| config.fee    | Decimal (string)       | Yes      | Fee on returned tokens (must be < spread) |
| slippage      | [Decimal, Decimal]     | No       | (price, tolerance) for orderly creation   |

Funds: Both pair denoms to seed the range

### range (claim)

Claim accrued fees from a range position.

```json
{ "range": { "claim": { "idx": "1" } } }
```

Funds: None

### range (deposit)

Add liquidity to an existing range position.

```json
{ "range": { "deposit": { "idx": "1" } } }
```

Funds: Both pair denoms

### range (withdraw)

Withdraw a fraction of a range position.

```json
{ "range": { "withdraw": { "idx": "1", "amount": "0.5" } } }
```

| Field  | Type             | Required | Description                    |
| ------ | ---------------- | -------- | ------------------------------ |
| idx    | Uint128 (string) | Yes      | Range index                    |
| amount | Decimal (string) | Yes      | Fraction to withdraw (0.0-1.0) |

Funds: None

### range (close)

Close a range position entirely and return all assets.

```json
{ "range": { "close": { "idx": "1" } } }
```

Funds: None

### range (transfer)

Transfer range ownership to another address.

```json
{ "range": { "transfer": { "idx": "1", "to": "thor1..." } } }
```

Funds: None

### arb

Trigger a market-maker arbitrage before the next operation. (whitelisted contracts only)

### do_swap / do_order / do_range

Internal callback actions -- not user-callable.

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{denoms: [string, string], oracles: OraclePair|null, market_makers: [string], tick: u8, range_delta: Decimal, fee_taker: Decimal, fee_maker: Decimal, fee_amm: Decimal, fee_address: string}`

### simulate

Simulate a swap. Pass a Coin to see the expected return.

```json
{ "simulate": { "denom": "btc/btc", "amount": "1000000" } }
```

Response: `{returned: Uint128, fee: Uint128}`

### order

Query a specific order by owner, side, and price.

```json
{ "order": ["thor1...", "base", { "fixed": "95000.5" }] }
```

Response: `{owner, side, price, rate, updated_at, offer, remaining, filled}`

### orders

Paginate a user's orders.

```json
{ "orders": { "owner": "thor1...", "side": "base", "offset": 0, "limit": 30 } }
```

Response: `{orders: [OrderResponse]}`

### range

Query a specific range by index.

```json
{ "range": "1" }
```

Response: `{idx, owner, high, low, skew, spread, fee, base, quote, price, ask, bid, fees}`

### ranges

Paginate range positions, optionally filtered by owner.

```json
{ "ranges": { "owner": "thor1...", "cursor": null, "limit": 10 } }
```

Response: `{ranges: [RangeResponse]}`

### book

Query the aggregated order book.

```json
{ "book": { "limit": 20, "offset": 0 } }
```

Response: `{base: [{price, total}], quote: [{price, total}]}`

## Sudo Messages (Governance)

- `StartSchedule(u64)` -- begin a timed schedule
- `EndSchedule {}` -- end the active schedule
- `UpdateConfig(ConfigUpdate)` -- update fees, market makers, oracles, range_delta

## Key Types

- **Price**: `{"fixed": "42000.5"}` (absolute) or `{"oracle": -500}` (basis points offset from oracle). Oracle bps must be |bps| < 10000.
- **Side**: `"base"` or `"quote"`
- **Tick**: `u8` -- significant figures for price truncation
- **Denoms**: `[string, string]` -- `[base_denom, quote_denom]`
- **RangeConfig**: `{high, low, skew, spread, fee}` -- all Decimal/SignedDecimal strings
- **CallbackData**: Binary (base64-encoded)

## Examples

Market swap 0.1 BTC for RUNE:

```bash
thornode tx wasm execute <fin_btc_rune> \
  '{"swap":{"min_return":"45000000"}}' \
  --amount 10000000btc/btc --from mykey --gas auto --gas-adjustment 1.5
```

Place a limit order at 200bps below oracle:

```bash
thornode tx wasm execute <fin_btc_rune> \
  '{"order":[[[["quote",{"oracle":-200},"100000000"]],null]]}' \
  --amount 100000000rune --from mykey --gas auto --gas-adjustment 1.5
```

Query your orders:

```bash
thornode query wasm contract-state smart <fin_btc_rune> \
  '{"orders":{"owner":"thor1...","side":"quote","offset":0,"limit":30}}'
```
