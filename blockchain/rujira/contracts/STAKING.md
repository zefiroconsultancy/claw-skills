# Staking (RUJI Staking)

Dual-mode staking contract supporting account-based yield (claim revenue) and liquid staking (mint/burn share tokens).

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/staking.rs](https://gitlab.com/thorchain/rujira/-/blob/main/packages/rujira-rs/src/interfaces/staking.rs)

## Execute Messages

### account > bond

Bond tokens to an account to earn revenue share.

```json
{ "account": { "bond": {} } }
```

Funds: bond_denom (see config)

### account > claim

Claim all pending revenue for bonded tokens.

```json
{ "account": { "claim": {} } }
```

Funds: None

### account > withdraw

Withdraw bonded tokens and claim pending revenue. Omit `amount` to withdraw all.

```json
{ "account": { "withdraw": { "amount": "500000" } } }
```

| Field  | Type             | Required | Description                      |
| ------ | ---------------- | -------- | -------------------------------- |
| amount | Uint128 (string) | No       | Amount to withdraw; null for all |

Funds: None

### liquid > bond

Bond tokens and receive liquid staking share tokens.

```json
{ "liquid": { "bond": {} } }
```

Funds: bond_denom (see config)

### liquid > unbond

Burn liquid staking share tokens to redeem underlying bonded assets.

```json
{ "liquid": { "unbond": {} } }
```

Funds: Share token (liquid staking receipt denom)

### settle

Internal: settles returned funds from revenue conversion swaps. Not user-callable.

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{bond_denom, revenue_denom, revenue_converter: [contract, msg, limit], fee: [Decimal, string]|null}`

### status

```json
{ "status": {} }
```

Response: `{account_bond, assigned_revenue, liquid_bond_shares, liquid_bond_size, undistributed_revenue}` (all Uint128 strings)

### account

Query a specific account's bonded amount and pending revenue.

```json
{ "account": { "addr": "thor1..." } }
```

Response: `{addr, bonded, pending_revenue}` (amounts as Uint128 strings)

## Sudo Messages (Governance)

- `SetRevenueConverter {contract, msg, limit}` -- update the revenue conversion config

## Key Types

- **AccountMsg**: Nested under `"account"` key -- `bond`, `claim`, `withdraw`
- **LiquidMsg**: Nested under `"liquid"` key -- `bond`, `unbond`
- **revenue_converter**: Tuple `(contract_address, execute_msg_binary, threshold_limit)` -- config for converting revenue_denom into bond_denom
- **fee**: Optional `(Decimal, string)` -- fee rate and destination address

## Examples

Bond RUJI to account staking:

```bash
thornode tx wasm execute <staking> \
  '{"account":{"bond":{}}}' \
  --amount 100000000factory/thor1stakingaddr.../ruji --from mykey --gas auto --gas-adjustment 1.5
```

Claim pending revenue:

```bash
thornode tx wasm execute <staking> \
  '{"account":{"claim":{}}}' \
  --from mykey --gas auto --gas-adjustment 1.5
```

Liquid bond (receive transferable share token):

```bash
thornode tx wasm execute <staking> \
  '{"liquid":{"bond":{}}}' \
  --amount 100000000factory/thor1stakingaddr.../ruji --from mykey --gas auto --gas-adjustment 1.5
```
