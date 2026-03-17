# Secured Assets

THORChain's App Layer uses **secured denoms** to represent cross-chain assets natively.
All secured assets use 8 decimal places.

## Cross-Chain Flow

Any cross-chain interaction with Rujira contracts follows a 3-step pattern:

```
L1 Chain ──S+──▶ App Layer ──(interact)──▶ App Layer ──S-──▶ L1 Chain
```

1. **Deposit (S+):** Send an L1 transaction (e.g. BTC on Bitcoin) to a THORChain inbound vault with a `S+:<contract>:<msg>` memo. Funds arrive on the App Layer as secured denoms (e.g. `btc-btc`).
2. **Interact:** Once on the App Layer, use contracts normally — swap on FIN, deposit into Ghost, stake RUJI, etc. All operations use secured denoms as standard Cosmos coins.
3. **Withdraw (S-):** Send secured denoms back to an L1 chain using a `S-:<contract>:<msg>` memo. THORChain's vaults release native assets on the destination chain.

Users who already have assets on the App Layer (e.g. from a previous deposit or on-chain transfer) skip step 1 and interact directly. The S+/S- pattern is only needed to bridge between L1 chains and the App Layer.

## Denom Formats

There are two denom formats. **L1 denoms** identify assets on their native chain (used in THORChain memos). **Secured denoms** are the x/bank tokens on the App Layer (used in contract interactions).

| Asset | L1 denom (dot, uppercase) | Secured denom (dash, lowercase)                       |
| ----- | ------------------------- | ----------------------------------------------------- |
| BTC   | `BTC.BTC`                 | `btc-btc`                                             |
| ETH   | `ETH.ETH`                 | `eth-eth`                                             |
| USDC  | `ETH.USDC-0XA0B8...`      | `eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` |
| RUNE  | `THOR.RUNE`               | `rune`                                                |
| BCH   | `BCH.BCH`                 | `bch-bch`                                             |
| LTC   | `LTC.LTC`                 | `ltc-ltc`                                             |
| DOGE  | `DOGE.DOGE`               | `doge-doge`                                           |
| XRP   | `XRP.XRP`                 | `xrp-xrp`                                             |
| ATOM  | `GAIA.ATOM`               | `gaia-atom`                                           |
| AVAX  | `AVAX.AVAX`               | `avax-avax`                                           |
| BNB   | `BSC.BNB`                 | `bsc-bnb`                                             |

**Pattern:** L1 = `CHAIN.SYMBOL` (dot, uppercase). Secured = `chain-symbol` (dash, lowercase).
For ERC-20 tokens: `eth-<symbol>-<contract_address>` (all lowercase).

## Common Token Denoms

| Token         | Denom                                                 |
| ------------- | ----------------------------------------------------- |
| BTC           | `btc-btc`                                             |
| ETH           | `eth-eth`                                             |
| USDC (ERC-20) | `eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48` |
| USDT (ERC-20) | `eth-usdt-0xdac17f958d2ee523a2206206994597c13d831ec7` |
| RUNE          | `rune`                                                |

## Deposit (S+): L1 -> App Layer

To bring assets onto the App Layer, send an L1 transaction to a THORChain inbound vault with a `S+` memo:

```
S+:<contract_address>:<base64_execute_msg>
```

The base64 payload is the CosmWasm execute message the contract will receive along with the deposited funds.

### Example: Deposit 1 BTC into Ghost Vault

```bash
# 1. Query inbound vault address for Bitcoin
curl "https://api-thorchain.rorcual.xyz/thorchain/inbound_addresses" | jq '.[] | select(.chain=="BTC")'
# Returns: { "chain": "BTC", "address": "bc1q...", ... }

# 2. Base64-encode the contract execute message
echo -n '{"deposit":{}}' | base64
# Output: eyJkZXBvc2l0Ijp7fX0=

# 3. Send BTC on the Bitcoin network to the inbound vault address with memo:
#    S+:<ghost_vault_contract>:eyJkZXBvc2l0Ijp7fX0=
#
#    Use any Bitcoin wallet/tool (e.g., bitcoin-cli, Vultisig, etc.)
#    The memo goes in the OP_RETURN output.
```

THORChain observes the L1 transaction, mints `btc-btc` secured tokens, and delivers them to the contract with the decoded execute message.

## Withdraw (S-): App Layer -> L1

To withdraw secured assets back to an L1 chain, send a THORChain `MsgDeposit` with a `S-` memo:

```
S-:<contract_address>:<base64_execute_msg>
```

### Example: Withdraw BTC from Ghost Vault back to Bitcoin L1

```bash
# 1. Base64-encode the withdraw message
echo -n '{"withdraw":{}}' | base64
# Output: eyJ3aXRoZHJhdyI6e319

# 2. Send the secured tokens via MsgDeposit with the S- memo
thornode tx deposit 100000000btc-btc \
  "S-:<ghost_vault_contract>:eyJ3aXRoZHJhdyI6e319" \
  --from mykey --gas auto --gas-adjustment 1.5
```

THORChain burns the secured tokens and sends native BTC from the vault to the sender's L1 address.

## Inbound Addresses

Query current inbound vault addresses via the THORNode REST API:

```bash
curl "https://api-thorchain.rorcual.xyz/thorchain/inbound_addresses"
```

Each L1 chain has a dedicated vault address. **Addresses rotate periodically — always query fresh before sending. Do NOT cache inbound addresses.**

## Decimal Handling

All amounts use 8 decimal places regardless of the native token's decimals:

- 1 BTC = `100000000` (8 decimals, same as native)
- 1 ETH = `100000000` (8 decimals, not 18)
- 1 USDC = `100000000` (8 decimals, not 6)

In JSON messages, amounts are always strings: `"100000000"`.
