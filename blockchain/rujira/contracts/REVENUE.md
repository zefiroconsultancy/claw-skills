# Revenue (Fee Distributor)

Collects protocol revenue in various token denoms, swaps them to target denoms, and distributes to configured addresses.

Contract address: See /skill/references/networks.md
Source: [packages/rujira-rs/src/interfaces/revenue.rs](https://gitlab.com/thorchain/rujira/-/blob/main/packages/rujira-rs/src/interfaces/revenue.rs)

## Execute Messages

### run

Execute all configured swap actions to convert accumulated revenue tokens to target denoms and distribute. Can be called by anyone but typically called by the designated executor.

```json
{ "run": {} }
```

Funds: None

## Query Messages

### config

```json
{ "config": {} }
```

Response: `{owner: String, executor: String, target_denoms: [String], target_addresses: [(String, u8)]}`

The `target_addresses` field is a list of `(address, weight)` tuples where weight is a u8 percentage.

### actions

List all configured swap actions.

```json
{ "actions": {} }
```

Response: `{actions: [{denom: String, contract: String, limit: Uint128, msg: Binary}]}`

### status

```json
{ "status": {} }
```

Response: `{last: String | null}` -- last execution info

## Sudo Messages (Governance)

- `set_owner` -- Change the owner address
- `set_executor` -- Change the executor address
- `set_action` -- Configure a swap action for a denom (contract, limit, msg)
- `unset_action` -- Remove a swap action for a denom
- `add_target_denom` -- Add a new target denom for distribution

## Examples

Run revenue distribution:

```bash
thornode tx wasm execute <revenue> \
  '{"run":{}}' \
  --from mykey --gas auto --gas-adjustment 1.5
```
