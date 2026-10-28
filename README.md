# sasa contracts

The smart contracts behind [sasa](https://sasapad.fun), a meme coin launchpad **deployed on BNB Smart Chain (BSC)** and compatible with other EVM networks. A creator launches a coin in one step on BNB Chain, or races it on BNB Chain, Base, Robinhood Chain and Arc at once: for 5 minutes buyers pick a chain, the chain with the most money wins and becomes the coin's home, and its liquidity is locked forever.

On BNB Chain every coin trades against **BNB** in a **PancakeSwap Infinity** pool, priced in dollars with the **Chainlink BNB/USD** feed.

## Technology Stack
- **Blockchain:** BNB Smart Chain (primary) + EVM-compatible chains (Base, Robinhood Chain, Arc)
- **Smart Contracts:** Solidity 0.8.26
- **DEX:** PancakeSwap Infinity (CL pools + hooks) on BNB Chain; Uniswap v4 on the other chains
- **Cross-chain:** LayerZero v2, Across
- **Frontend:** Next.js + viem + wagmi ([sasapad.fun](https://sasapad.fun))
- **Development:** Foundry, OpenZeppelin v5.1.0

## Supported Networks
- **BNB Smart Chain Mainnet (Chain ID: 56)**
- **BNB Smart Chain Testnet (Chain ID: 97)**
- Base Mainnet (Chain ID: 8453)
- Robinhood Chain Mainnet (Chain ID: 4663)
- Arc Mainnet (Chain ID: 5042)

## Contract Addresses (BNB Chain)

| Network | Factory | Builder (PancakeSwap pools) | Trade Router | Hub |
|---|---|---|---|---|
| **BNB Mainnet (v8, current)** | `0x1Fe532901FE7c66251E55606021b4F2A7e12ABaa` | `0x68e8a49487421F2B1A976254feb205197C7bCBBF` | `0x4b43040e89F1440E4752C12addcD6A347e33E7E1` | `0xd211F61274284DC3Af39C428bd4bDf59c27836AE` |
| BNB Mainnet (v7) | `0x88e21D24C35034f80E854efBD5ACD16507a9169B` | `0x05aFA6adFcd1c40CF82174025ecBC2E1ED603Ca4` | `0xc8c1C173a1d3057619fCf45941c41f02B9E7E5dD` | `0x173b228d343E174D2671a61C294D3AF06946DEBf` |
| BNB Testnet | `0x12D1f65b5d4c11d8059Cc6739aC3cC321Eb42a9D` | `0x58068303Ca51EbB8FC25B594361186a52D6071E5` | `0xfF8f34C09cd042A52275BE1B30Eb377b5370cAB6` | `0x1044fe554262C92F06D0838b5dE320e12e54ea5D` |

Every BNB Chain address (hook, fee vault, consolidator, bridge, PancakeSwap CL pool manager, Chainlink feed, WBNB) is in [`deployments/race-mainnet-v8/56.json`](deployments/race-mainnet-v8/56.json), [`deployments/race-mainnet/56.json`](deployments/race-mainnet/56.json) and [`deployments/race/97.json`](deployments/race/97.json). BNB Chain network settings are in [`bnbconfig.json`](bnbconfig.json).

## Features
- **One-step launch on BNB Chain:** the coin, its PancakeSwap Infinity coin/BNB pool and the creator's first buy in one transaction.
- **5-minute multi-chain race:** BNB Chain competes with Base, Robinhood Chain and Arc; the winner's pool is deepened with the losers' money and locked forever.
- **Anti-sniper launch fee:** the pool hook starts the fee at 10% and lowers it to 2% over the first 2 minutes.
- **Dollar trading:** routers swap USDC → BNB → coin in one call, so users never need to hold BNB.
- **Locked liquidity and creator fees:** liquidity can't be pulled; trading fees are split 70% sasa / 30% creator.

## Layout
- `src/race/RaceBuilderPcs.sol`, `RaceLaunchHookPcs.sol`, `RaceTradePcs.sol`: the BNB Chain contracts (PancakeSwap Infinity coin/BNB pools, Chainlink BNB/USD)
- `src/race/RaceFactory.sol`, `RaceSeat.sol`, `RaceCross.sol`: launch and cross-chain race logic shared by every chain
- `src/race/RaceBuilder*.sol`, `RaceTradeEth.sol`: the same for Uniswap v4 chains
- `src/omni/`, `src/usd/`, `src/fees/`, `src/auto/`: coin, hub, consolidator, bridges, fee vault, limit orders
- `test/RacePcsFlow.t.sol`: the full BNB Chain flow on a local PancakeSwap Infinity
- `test/fork/`: the flows on the real BNB Chain (PancakeSwap Infinity, Chainlink, USDC)
- `script/race/`: deploy and wiring scripts (`--rpc-url bsc` for BNB Chain)

## Build and test
Libraries (see `foundry.toml` remappings): forge-std v1.9.6, OpenZeppelin v5.1.0, Uniswap v4-periphery, PancakeSwap Infinity core (`lib/infinity-core`), LayerZero v2.
```bash
forge build
forge test --match-path test/RacePcsFlow.t.sol          # BNB Chain flow, local PancakeSwap Infinity
forge test --match-path test/fork/RacePcsFork.t.sol --fork-url bsc   # on real BNB Chain
```

Security: contracts were tested end to end (unit, fuzz and mainnet-fork tests), not yet audited by a third party.
