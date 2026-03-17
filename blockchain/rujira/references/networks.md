# Networks and Contract Discovery

## THORChain Mainnet

### Endpoints

| Type    | URL                                  |
| ------- | ------------------------------------ |
| RPC     | `https://rpc-thorchain.rorcual.xyz`  |
| RPC     | `https://thorchain.ibs.team/rpc/`    |
| gRPC    | `https://grpc-thorchain.rorcual.xyz` |
| gRPC    | `https://thorchain.ibs.team:443`     |
| REST    | `https://api-thorchain.rorcual.xyz`  |
| REST    | `https://thorchain.ibs.team/api/`    |
| Midgard | `https://midgard.ninerealms.com`     |

### Rujira APIs (Mainnet)

| Type       | URL                                                                                          |
| ---------- | -------------------------------------------------------------------------------------------- |
| GraphQL    | `https://api.rujira.network/api/graphiql` (requires API key — request at api@rujira.network) |
| Trade REST | `https://api.rujira.network/api/trade/`                                                      |
| RUJI REST  | `https://api.rujira.network/api/ruji/`                                                       |

### Contract Discovery

Contract addresses change per deployment and migration — never hardcode them. Use the methods below to look up current addresses at runtime.

Query deployed contracts:

```bash
thornode query wasm list-contract-by-code <code_id>
```

Or via REST (no CLI needed):

```bash
curl "https://api-thorchain.rorcual.xyz/cosmwasm/wasm/v1/code/<code_id>/contracts"
```

Contract addresses are deterministic per deployment. Use the Rujira GraphQL API to look up current addresses:

```graphql
{
  contracts {
    name
    address
    codeId
  }
}
```

## THORChain Stagenet

### Endpoints

| Type    | URL                                       |
| ------- | ----------------------------------------- |
| Midgard | `https://stagenet-midgard.ninerealms.com` |

> Stagenet RPC/gRPC: check [docs.rujira.network](https://docs.rujira.network) for current stagenet endpoints.

### Rujira APIs (Stagenet)

| Type       | URL                                                                                                  |
| ---------- | ---------------------------------------------------------------------------------------------------- |
| GraphQL    | `https://api-preview.rujira.network/api/graphiql` (requires API key — request at api@rujira.network) |
| Trade REST | `https://preview-api.rujira.network/api/trade/`                                                      |
| RUJI REST  | `https://preview-api.rujira.network/api/ruji/`                                                       |

### Notes

- Stagenet is the primary test environment.
- Contract code IDs differ from mainnet.
- Check [docs.rujira.network](https://docs.rujira.network) for current stagenet RPC endpoints to use with `thornode` CLI.

## Chain ID

| Network  | Chain ID                |
| -------- | ----------------------- |
| Mainnet  | `thorchain-1`           |
| Stagenet | `thorchain-stagenet-v2` |

## Endpoint Reference

Full endpoint list: [docs.rujira.network/developers/developer-endpoints](https://docs.rujira.network/developers/developer-endpoints)
