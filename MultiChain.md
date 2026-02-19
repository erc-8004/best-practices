# Multi-Chain Deployment Guide

Best practices for deploying ERC-8004 registries on non-EVM chains, based on the TRC-8004 implementation on TRON.

## Why Multi-Chain?

ERC-8004 is designed as a per-chain singleton. While the reference implementation targets EVM L1s and L2s, the standard's architecture is chain-agnostic. Agents on different chains can still reference each other using CAIP-10 identifiers.

Deploying on non-EVM chains expands the agent discovery network and brings ERC-8004's trust layer to ecosystems with different user bases and cost structures.

## TRON (TRC-8004)

TRON is EVM-compatible at the smart contract level (Solidity), but differs in tooling, address formats, and RPC layer.

### Key Differences from Ethereum

| Aspect | Ethereum | TRON |
|--------|----------|------|
| Address Format | `0x` hex (20 bytes) | Base58Check (starts with `T`) |
| Block Numbers | uint64-safe | Can exceed uint64 — use BigInteger |
| Tooling | Hardhat / Foundry | TronBox / TronIDE |
| RPC | JSON-RPC (eth_*) | TronGrid / TronWeb API |
| Token Standard | ERC-721 | TRC-721 (same interface) |
| Gas Model | Gas + EIP-1559 | Energy + Bandwidth |
| Event Indexing | The Graph / custom | TronGrid events API |

### Deployment Checklist

1. **Solidity compatibility** — TRON supports Solidity 0.8.x. No contract changes needed for basic ERC-8004 registries.

2. **Address handling** — Convert between Base58 and hex internally. Store hex on-chain, display Base58 to users.

3. **Block number columns** — Use `BigInteger` (not `int64`) for block numbers in your database schema.

4. **Event indexing** — TronGrid provides an events API. Poll at 30-second intervals with block range tracking.

5. **Transaction confirmation** — TRON blocks finalize in ~3 seconds. Use 20-block confirmation for safety.

6. **Registration URIs** — Same IPFS-based metadata format. No changes to the registration file schema.

### Deployed Addresses (TRON Mainnet)

| Contract | Address |
|----------|---------|
| IdentityRegistry | `THmfi8uJuUpTfUmYLDX7UD1KaE4P6HKgqA` |
| ValidationRegistry | `TCoJA4BYXWZhp5eanCchMw67VA83tQ83n1` |
| ReputationRegistry | `TV8KWmp8qcj55sjs1NSjVxmRmZP7CYzNxH` |
| IncidentRegistry | `TJ26Pu24ar7Qdh9Bm6tbBVdtzCJkbxS5eR` |

### Deployed Addresses (TRON Shasta Testnet)

| Contract | Address |
|----------|---------|
| IdentityRegistry | `TFKNqk9bjwWp5uRiiGimqfLhVQB8jSxYi7` |
| ValidationRegistry | `TPgGWWyUdxNryUCN49TdT4b3F4WB3Edr16` |
| ReputationRegistry | `TRaYogyr2qc7WgsmuVF5Js39aCmoG7vZrA` |
| IncidentRegistry | `TPB59NFdypBpkJtWH7yE8XenKrdT1Q4g4s` |

## Cross-Chain Agent References

Use CAIP-10 format for cross-chain agent identification:

```
# Ethereum agent
eip155:1:0x8004A169FB4a3325136EB29fA0ceB6D2e539a432

# TRON agent (use hex address, not Base58)
tron:728126428:<hex-address>
```

In the registration file, agents can list endpoints on multiple chains:

```json
{
  "endpoints": [
    {
      "name": "agentWallet",
      "endpoint": "eip155:1:0xYourEthAddress"
    },
    {
      "name": "agentWallet",
      "endpoint": "tron:728126428:0xYourTronHexAddress"
    }
  ],
  "registrations": [
    {
      "agentId": 1,
      "agentRegistry": "eip155:1:0x8004A169FB4a3325136EB29fA0ceB6D2e539a432"
    },
    {
      "agentId": 1,
      "agentRegistry": "tron:728126428:THmfi8uJuUpTfUmYLDX7UD1KaE4P6HKgqA"
    }
  ]
}
```

## General Multi-Chain Best Practices

1. **Same Solidity source** — Keep contract code identical across chains. Only change deployment tooling.

2. **Consistent metadata schema** — Use the same ERC-8004 registration-v1 JSON format regardless of chain.

3. **CAIP-10 everywhere** — Always use chain-qualified identifiers in cross-references.

4. **Chain-specific SDKs** — Wrap chain-specific RPC calls (ethers.js, tronweb, etc.) behind a common interface.

5. **Event indexer per chain** — Each chain needs its own indexer. Normalize events to a common database schema.

6. **Test on testnets first** — Every chain has a testnet. Deploy and run integration tests before mainnet.

## Resources

- [M2M TRC-8004 Registry](https://m2mregistry.io) — Live TRON implementation
- [ERC-8004 Contracts](https://github.com/erc-8004/erc-8004-contracts) — Reference Ethereum implementation
- [CAIP-10 Specification](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-10.md) — Chain-agnostic account IDs
- [TronBox Documentation](https://developers.tron.network/docs/tronbox) — TRON smart contract tooling
