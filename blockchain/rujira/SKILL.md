---
name: rujira
description: "Interact with Rujira DeFi contracts on THORChain's App Layer: trade, lend, borrow, stake, and manage liquidity."
---

# Rujira

Rujira is a DeFi protocol suite deployed as CosmWasm contracts on THORChain's App Layer.
It provides an orderbook DEX (FIN), AMM pools (BOW), lending vaults and credit accounts (Ghost),
staking (Staking), liquid node bonding (bRUNE), token launches (Pilot), and revenue distribution.
All contracts share THORChain's oracle-secured asset system.
Contracts can be interacted with via `thornode` CLI, REST/gRPC endpoints, or programmatic SDKs.

## Secured Assets

THORChain represents cross-chain assets as **secured denoms** on the App Layer:

| Asset | L1 denom (dot, upper) | Secured denom (dash, lower)                           |
| ----- | --------------------- | ----------------------------------------------------- |
| RUNE  | `THOR.RUNE`           | `rune`                                                |
| BTC   | `BTC.BTC`             | `btc-btc`                                             |
| ETH   | `ETH.ETH`             | `eth-eth`                                             |
| USDC  | `ETH.USDC-0XA0B8...`  | `eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` |
| USDT  | `ETH.USDT-0XDAC1...`  | `eth-usdt-0xdac17f958d2ee523a2206206994597c13d831ec7` |

L1 denoms (dot separator, uppercase) are used in THORChain memos (S+/S-). Secured denoms (dash separator, lowercase) are used in all contract interactions on the App Layer.

All amounts use **8 decimal places** (THORChain standard).

**Cross-chain users must follow the S+/S- flow:**

1. **S+ (deposit):** Send an L1 transaction to a THORChain inbound vault with memo `S+:<contract>:<base64_msg>`. Funds arrive on the App Layer as secured denoms.
2. **Interact:** Use contracts normally on the App Layer — all secured denoms work as standard Cosmos coins.
3. **S- (withdraw):** Send secured denoms back to L1 with memo `S-:<contract>:<base64_msg>`.

This is a **required prerequisite** for any user bridging assets from an external chain (BTC, ETH, etc.) and a **required postscript** to return assets to L1. Users who already have assets on the App Layer can skip S+ and interact directly.

See `/skill/references/secured-assets.md` for the full denom list and cross-chain flow details.

## Interacting with Contracts

There are three ways to query and execute contracts. Agents should pick whichever fits their runtime.

### 1. CLI (`thornode` binary)

```bash
# Query (free, no signing)
thornode query wasm contract-state smart <contract_addr> '<json_query>'

# Execute
thornode tx wasm execute <contract_addr> '<json_msg>' \
  --from <key> --gas auto --gas-adjustment 1.5 --fees 2000000rune

# Execute with funds
thornode tx wasm execute <contract_addr> '{"deposit":{}}' \
  --amount 100000000btc/btc --from <key> --gas auto --gas-adjustment 1.5
```

See `/skill/references/signing.md` for keyring and signing details.

### 2. REST / gRPC (no binary required)

Query contract state via HTTP — no CLI needed:

```
GET {rest}/cosmwasm/wasm/v1/contract/{address}/smart/{query_data_base64}
```

`query_data_base64` is the base64-encoded JSON query message.
Example — query FIN config:

```bash
curl "https://api-thorchain.rorcual.xyz/cosmwasm/wasm/v1/contract/<fin_addr>/smart/$(echo -n '{"config":{}}' | base64)"
```

Broadcast a signed transaction:

```
POST {rest}/cosmos/tx/v1beta1/txs
Body: {"tx_bytes": "<base64-protobuf-encoded-signed-tx>", "mode": "BROADCAST_MODE_SYNC"}
```

Simulate (estimate gas):

```
POST {rest}/cosmos/tx/v1beta1/simulate
```

REST base URLs (use any):

- `https://api-thorchain.rorcual.xyz` (mainnet)
- `https://thorchain.ibs.team/api/` (mainnet)

See `/skill/references/networks.md` for all endpoints including Midgard, GraphQL, and stagenet.

### 3. Programmatic SDKs

**CosmJS** (TypeScript) — standard Cosmos SDK client:

```typescript
import { SigningCosmWasmClient } from "@cosmjs/cosmwasm-stargate";
const client = await SigningCosmWasmClient.connectWithSigner(rpcUrl, signer);
await client.execute(sender, contractAddr, msg, "auto", "", funds);
const result = await client.queryContractSmart(contractAddr, queryMsg);
```

**Vultisig SDK** (TypeScript) — MPC-based agentic wallet with no seed phrases:

```bash
npm install @vultisig/sdk
```

Vultisig provides threshold-signature vaults (Fast Vault for 2-of-2, Secure Vault for N-of-M)
with support for THORChain and 40+ chains. Designed for autonomous agents and bots that need
self-custodial signing without managing private keys directly.
See [Vultisig SDK docs](https://github.com/vultisig/docs/blob/main/developer-docs/vultisig-sdk/README.md)
and [@vultisig/sdk on npm](https://www.npmjs.com/package/@vultisig/sdk).

See [Integration Guides](https://gitlab.com/thorchain/rujira/-/tree/main/docs) for full CosmJS examples.

## Contract Index

| Contract            | Product              | Agent Docs                                          |
| ------------------- | -------------------- | --------------------------------------------------- |
| rujira-fin          | Orderbook DEX (FIN)  | [FIN.md](/skill/contracts/FIN.md)                   |
| rujira-bow          | AMM Pools (BOW)      | [BOW.md](/skill/contracts/BOW.md)                   |
| rujira-ghost-vault  | Lending Vaults       | [GHOST_VAULT.md](/skill/contracts/GHOST_VAULT.md)   |
| rujira-ghost-mint   | Mint Credit Pool     | [GHOST_MINT.md](/skill/contracts/GHOST_MINT.md)     |
| rujira-ghost-credit | Credit Accounts      | [GHOST_CREDIT.md](/skill/contracts/GHOST_CREDIT.md) |
| rujira-staking      | RUJI Staking         | [STAKING.md](/skill/contracts/STAKING.md)           |
| rujira-brune        | BRUNE Liquid Bonding | [BRUNE.md](/skill/contracts/BRUNE.md)               |
| rujira-merge        | Token Merge          | [MERGE.md](/skill/contracts/MERGE.md)               |
| rujira-revenue      | Revenue Router       | [REVENUE.md](/skill/contracts/REVENUE.md)           |

## Common Workflows

### Trade on FIN (market swap)

1. Query simulate to preview: `{"simulate": {"denom": "btc-btc", "amount": "10000000"}}`
2. Execute swap with funds: `{"swap": {"min_return": "950000000"}}` + attach `btc-btc`

### Trade on FIN (limit order)

1. Place an oracle-tracked order: `{"order": [[[["quote", {"oracle": -200}, "100000000000"]], null]]}` + attach quote denom
2. Query orders: `{"orders": {"owner": "<addr>"}}`
3. Withdraw filled amounts: `{"order": [[[["quote", {"oracle": -200}, null]], null]]}`

### Provide BOW liquidity

1. Deposit both denoms: `{"deposit": {"min_return": "1000000"}}` + attach both tokens
2. Withdraw LP tokens: `{"withdraw": {}}` + attach LP receipt token

### Lend via Ghost Vault

1. Deposit asset: `{"deposit": {"callback": null}}` + attach the vault's deposit denom
2. Check status: `{"status": {}}`
3. Withdraw: `{"withdraw": {"callback": null}}` + attach receipt tokens

### Open a credit account and borrow

1. Create account: `{"create": {"salt": "<base64>", "label": "my-loan", "tag": "app"}}`
2. Send collateral to the account address via `bank send`
3. Borrow: `{"account": {"addr": "<account>", "msgs": [{"borrow": {"amount": "1000000", "denom": "eth-usdc-..."}}]}}`

### Repay and close a credit account

1. Send repayment tokens to the account address via `bank send`
2. Repay and withdraw: `{"account": {"addr": "<account>", "msgs": [{"repay": {"amount": "1050000", "denom": "eth-usdc-..."}}, {"send": {"to_address": "<owner>", "funds": [{"amount": "50000000", "denom": "btc-btc"}]}}]}}`
3. Or close entirely: `{"account": {"addr": "<account>", "msgs": [{"repay_all": {}}, {"close": {}}]}}`

### Liquidate an undercollateralized position

1. Query all accounts: `{"all_accounts": {"cursor": null, "limit": 100}}`
2. Filter for `ltv >= "1.0"`
3. Build swap route and execute: `{"liquidate": {"addr": "<account>", "msgs": [{"execute": {"contract_addr": "<fin_pool>", "msg": "<base64_swap>", "funds": [{"amount": "50000000", "denom": "btc-btc"}]}}, {"repay": "eth-usdc-..."}]}}`

### Stake RUJI

**Account staking** (earn revenue, manual claim):

1. Bond: `{"account": {"bond": {}}}` + attach bond_denom
2. Claim: `{"account": {"claim": {}}}`
3. Withdraw: `{"account": {"withdraw": {"amount": null}}}`

**Liquid staking** (receive transferable share token):

1. Bond: `{"liquid": {"bond": {}}}` + attach bond_denom
2. Unbond: `{"liquid": {"unbond": {}}}` + attach share token

## Execution Checklist

Before broadcasting any transaction, follow these steps in order:

1. **Resolve contract address** — Query the Rujira GraphQL API or REST endpoint to get the current contract address. Never hardcode addresses.
2. **Verify denom** — Confirm the secured denom format (`chain-symbol`, lowercase, dash separator). All amounts are 8-decimal integer strings.
3. **Simulate** — Run `--dry-run` (CLI), `POST /cosmos/tx/v1beta1/simulate` (REST), or `client.simulate()` (CosmJS). Review gas estimate and expected state changes.
4. **Inspect funds & fees** — Confirm `--amount` matches intent. Gas fees are paid in RUNE (`--gas auto --gas-adjustment 1.5`).
5. **Broadcast** — Send the transaction.
6. **Verify result** — Query the tx hash to confirm success. For swaps, verify returned amount meets `min_return`.

### Safety Notes

- Set `min_return` on swaps to prevent excessive slippage.
- Monitor credit account LTV -- positions with `ltv >= 1.0` can be liquidated by anyone.
- Claim filled FIN orders regularly; unclaimed fills remain locked.
- All amounts are 8-decimal integers as strings (e.g., `"100000000"` = 1.0 BTC).
- Gas fees are paid in RUNE. Use `--gas auto --gas-adjustment 1.5` for safe estimation.
- Never send funds to a contract address directly via `bank send` unless it is a credit account collateral deposit.
- Verify contract addresses against `/skill/references/networks.md` before transacting.

## References

- [Secured Assets](/skill/references/secured-assets.md) -- full denom list and deposit/withdraw memos
- [Networks](/skill/references/networks.md) -- contract addresses for mainnet and stagenet
- [Signing](/skill/references/signing.md) -- keyring setup, signing options, and CLI examples
- [Integration Guides](https://gitlab.com/thorchain/rujira/-/tree/main/docs) -- detailed TypeScript/CosmJS examples
- [Source Code](https://gitlab.com/thorchain/rujira) -- Rust contract source on GitLab
