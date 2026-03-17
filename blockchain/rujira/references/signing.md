# Signing Transactions

## Keyring Setup

Create or import a key:

```bash
thornode keys add <name>
thornode keys add <name> --recover   # from mnemonic
```

List keys:

```bash
thornode keys list
```

## Executing Contracts

### Basic Execute

```bash
thornode tx wasm execute <contract_addr> '<json_msg>' \
  --from <key_name> \
  --gas auto --gas-adjustment 1.5 \
  --fees 2000000rune \
  --chain-id thorchain-1 \
  --node https://rpc-thorchain.rorcual.xyz
```

### Execute with Funds

```bash
thornode tx wasm execute <contract_addr> '{"deposit":{}}' \
  --amount 100000000btc-btc \
  --from <key_name> \
  --gas auto --gas-adjustment 1.5 \
  --fees 2000000rune
```

### Execute with Multiple Funds

```bash
thornode tx wasm execute <contract_addr> '{"deposit":{"min_return":"1000000"}}' \
  --amount 100000000btc-btc,50000000000eth-usdc-0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48 \
  --from <key_name> \
  --gas auto --gas-adjustment 1.5
```

## Querying Contracts

Queries are free and require no signing:

```bash
thornode query wasm contract-state smart <contract_addr> '<json_query>'
```

Example:

```bash
thornode query wasm contract-state smart <fin_addr> '{"config":{}}'
```

## Signing Options

| Flag                     | Purpose                                |
| ------------------------ | -------------------------------------- |
| `--from <key>`           | Key name or address to sign with       |
| `--gas auto`             | Automatically estimate gas             |
| `--gas-adjustment 1.5`   | Buffer multiplier for gas estimate     |
| `--fees <amount>`        | Explicit fee (e.g., `2000000rune`)     |
| `--gas-prices 0.02rune`  | Alternative to --fees; auto-calculates |
| `--chain-id thorchain-1` | Chain identifier                       |
| `--node <rpc_url>`       | RPC endpoint                           |
| `--broadcast-mode sync`  | Wait for CheckTx only (default)        |
| `--broadcast-mode block` | Wait for block inclusion               |
| `-y`                     | Skip confirmation prompt               |

## Stagenet

For stagenet transactions, override:

```bash
--chain-id thorchain-stagenet-v2
--node https://stagenet-rpc.ninerealms.com  # check docs.rujira.network for current stagenet endpoints
```

## Programmatic Signing

Use CosmJS for TypeScript/JavaScript:

```typescript
import { SigningCosmWasmClient } from "@cosmjs/cosmwasm-stargate";
import { DirectSecp256k1HdWallet } from "@cosmjs/proto-signing";

const wallet = await DirectSecp256k1HdWallet.fromMnemonic(mnemonic, {
  prefix: "thor",
});
const client = await SigningCosmWasmClient.connectWithSigner(
  "https://rpc-thorchain.rorcual.xyz",
  wallet,
  { gasPrice: { amount: "0.02", denom: "rune" } }
);

await client.execute(sender, contractAddr, msg, "auto", "", funds);
```

## Vultisig SDK

[Vultisig](https://github.com/vultisig/docs/blob/main/developer-docs/vultisig-sdk/README.md) provides MPC threshold-signature vaults — no seed phrases or private keys to manage. Designed for autonomous agents and bots.

```bash
npm install @vultisig/sdk
```

```typescript
import { FastVault, THORChainService } from "@vultisig/sdk";

// Create or load a 2-of-2 Fast Vault
const vault = await FastVault.create({ name: "my-agent-vault" });

// Sign and broadcast a CosmWasm execute message
const thorchain = vault.getService(THORChainService);
const txHash = await thorchain.executeContract({
  contractAddr: "<contract_addr>",
  msg: { swap: { min_return: "950000000" } },
  funds: [{ denom: "btc-btc", amount: "10000000" }],
  gasAdjustment: 1.5,
});
```

For Secure Vaults (N-of-M), multi-chain support, and advanced configuration see the [Vultisig SDK docs](https://github.com/vultisig/docs/blob/main/developer-docs/vultisig-sdk/README.md) and [@vultisig/sdk on npm](https://www.npmjs.com/package/@vultisig/sdk).

See [Integration Guides](https://gitlab.com/thorchain/rujira/-/tree/main/docs) for complete examples.
