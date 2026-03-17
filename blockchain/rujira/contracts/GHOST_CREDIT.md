# Ghost Credit (Ghost Credit Accounts)

Manages leveraged credit accounts that can hold collateral, borrow from Ghost vaults/mints, and be liquidated when undercollateralized.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/ghost/credit/](https://gitlab.com/thorchain/rujira/-/tree/main/packages/rujira-rs/src/interfaces/ghost/credit)

## Execute Messages

### create

Create a new credit account with a deterministic address.

```json
{
  "create": {
    "salt": "base64encodeddata",
    "label": "my-account",
    "tag": "trading"
  }
}
```

| Field | Type            | Required | Description                               |
| ----- | --------------- | -------- | ----------------------------------------- |
| salt  | String (Binary) | Yes      | Salt for deterministic address generation |
| label | String          | Yes      | Label appended to the account contract    |
| tag   | String          | Yes      | Tag for filtering accounts in queries     |

Funds: None

### account

Execute a batch of operations on a credit account. The account must remain sufficiently collateralized (LTV below `adjustment_threshold`) after all messages execute.

```json
{
  "account": {
    "addr": "thor1account...",
    "msgs": [
      {
        "borrow": {
          "amount": "1000000",
          "denom": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
        }
      },
      {
        "send": {
          "to_address": "thor1...",
          "funds": [
            {
              "amount": "500000",
              "denom": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
            }
          ]
        }
      }
    ]
  }
}
```

| Field | Type         | Required | Description                        |
| ----- | ------------ | -------- | ---------------------------------- |
| addr  | String       | Yes      | Credit account address             |
| msgs  | [AccountMsg] | Yes      | Ordered list of account operations |

Funds: None (funds are managed within the account)

#### AccountMsg variants

**borrow** - Borrow assets from a Ghost vault/mint into the account.

```json
{
  "borrow": {
    "amount": "1000000",
    "denom": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
  }
}
```

**repay** - Repay debt using account funds.

```json
{
  "repay": {
    "amount": "500000",
    "denom": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
  }
}
```

**repay_all** - Repay all outstanding debt to the maximum possible from account balances.

```json
{ "repay_all": {} }
```

**execute** - Execute an arbitrary contract call from the account.

```json
{
  "execute": {
    "contract_addr": "thor1...",
    "msg": "base64encodedmsg",
    "funds": [
      {
        "amount": "1000000",
        "denom": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
      }
    ]
  }
}
```

**send** - Send funds from the account to an address.

```json
{
  "send": {
    "to_address": "thor1...",
    "funds": [
      {
        "amount": "1000000",
        "denom": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
      }
    ]
  }
}
```

**close** - Close the account and withdraw all collateral to the owner. Fails if outstanding debts exist.

```json
{ "close": {} }
```

**transfer** - Transfer account ownership to a new address.

```json
{ "transfer": "thor1newowner..." }
```

**set_preference_msgs** - Set liquidation preference messages that run before any liquidation.

```json
{
  "set_preference_msgs": [
    { "repay": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48" },
    {
      "execute": {
        "contract_addr": "thor1...",
        "msg": "base64...",
        "funds": []
      }
    }
  ]
}
```

**set_preference_order** - Set liquidation order constraint: `denom` cannot be liquidated while `after` denom is still held.

```json
{ "set_preference_order": { "denom": "rune", "after": "btc-btc" } }
```

Pass `after: null` to remove the constraint for that denom.

### do_account

Internal message for sequential AccountMsg processing. Not user-callable.

### check_account

Internal health check against `adjustment_threshold`. Not user-callable.

### liquidate

Liquidate an undercollateralized credit account. Can only succeed when account LTV exceeds `liquidation_threshold`.

```json
{
  "liquidate": {
    "addr": "thor1account...",
    "msgs": [
      { "repay": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48" },
      {
        "execute": {
          "contract_addr": "thor1dex...",
          "msg": "base64swap...",
          "funds": [{ "amount": "1000000", "denom": "btc-btc" }]
        }
      }
    ]
  }
}
```

| Field | Type           | Required | Description                         |
| ----- | -------------- | -------- | ----------------------------------- |
| addr  | String         | Yes      | Credit account address to liquidate |
| msgs  | [LiquidateMsg] | Yes      | Liquidation route operations        |

Funds: None

#### LiquidateMsg variants

**repay** - Repay debt using the account's balance of the specified denom.

```json
{ "repay": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48" }
```

**execute** - Execute a contract call (e.g. swap collateral for debt token) from the account.

```json
{
  "execute": {
    "contract_addr": "thor1...",
    "msg": "base64...",
    "funds": [{ "amount": "1000000", "denom": "btc-btc" }]
  }
}
```

### do_liquidate

Internal message for sequential LiquidateMsg processing. Not user-callable.

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{code_id: u64, collateral_ratios: {denom: Decimal}, fee_liquidation: Decimal, fee_liquidator: Decimal, fee_address: Addr, liquidation_max_slip: Decimal, liquidation_threshold: Decimal, adjustment_threshold: Decimal, full_liquidation_threshold: Uint128}`

### borrows

```json
{ "borrows": {} }
```

Response: `{borrowers: [BorrowerResponse]}`

### account

```json
{ "account": "thor1account..." }
```

Response: `{owner: Addr, account: Addr, tag: String, collaterals: [CollateralResponse], debts: [DebtResponse], ltv: Decimal, liquidation_preferences: LiquidationPreferences}`

### accounts

```json
{ "accounts": { "owner": "thor1...", "tag": null } }
```

Response: `{accounts: [AccountResponse]}`

### all_accounts

```json
{ "all_accounts": { "cursor": null, "limit": 100 } }
```

Response: `{accounts: [AccountResponse]}`

### predict

Returns the predicted address for a new account before creation.

```json
{ "predict": { "owner": "thor1...", "salt": "base64..." } }
```

Response: `Addr`

## Sudo Messages (Governance)

- **SetVault** - Register a Ghost vault/mint for borrowing: `{set_vault: {address: "thor1..."}}`
- **SetCollateral** - Set collateralization ratio for a denom: `{set_collateral: {denom: "btc-btc", collateralization_ratio: "0.8"}}`
- **UpdateConfig** - Update liquidation/fee parameters: `{update_config: {fee_liquidation, fee_liquidator, ...}}`

## Key Types

### CollateralResponse

```json
{
  "collateral": { "coin": { "amount": "100000000", "denom": "btc-btc" } },
  "value_full": "50000.0",
  "value_adjusted": "40000.0"
}
```

`value_adjusted` = `value_full` \* collateralization_ratio for that denom.

### DebtResponse

```json
{"debt": {"borrower": {...}, "addr": "...", "current": "1000000", "shares": "950000"}, "value": "1000.0"}
```

### LiquidationPreferences

```json
{
  "messages": [
    { "repay": "eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48" }
  ],
  "order": { "map": { "rune": "btc-btc" }, "limit": 100 }
}
```

`messages`: LiquidateMsg list injected at liquidation start (errors ignored to prevent blocking).
`order`: Constraints on which collateral can be liquidated first. `{"A": "B"}` means A cannot be liquidated while B is still held.

### Collateral

Enum with one variant: `{"coin": {"amount": "100000000", "denom": "btc-btc"}}`. Only secured assets (L1 chains) are accepted.

## Examples

Create a credit account:

```bash
thornode tx wasm execute <ghost_credit> \
  '{"create":{"salt":"bXlzYWx0","label":"my-loan","tag":"trading"}}' \
  --from mykey --gas auto --gas-adjustment 1.5
```

Deposit collateral to the account (via bank send):

```bash
thornode tx bank send mykey <account_addr> 100000000btc-btc \
  --gas auto --gas-adjustment 1.5
```

Borrow USDC against collateral:

```bash
thornode tx wasm execute <ghost_credit> \
  '{"account":{"addr":"<account_addr>","msgs":[{"borrow":{"amount":"50000000000","denom":"eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"}}]}}' \
  --from mykey --gas auto --gas-adjustment 1.5
```

Repay all debt and close account:

```bash
thornode tx wasm execute <ghost_credit> \
  '{"account":{"addr":"<account_addr>","msgs":[{"repay_all":{}},{"close":{}}]}}' \
  --from mykey --gas auto --gas-adjustment 1.5
```
