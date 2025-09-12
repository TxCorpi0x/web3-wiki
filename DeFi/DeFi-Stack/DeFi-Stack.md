# DeFi Stack Overview

The **DeFi stack** (Decentralized Finance stack) is the layered infrastructure that supports decentralized financial applications and services on blockchain networks.

---

## 1. Settlement Layer (Base Layer)

- **What it is**: The foundational blockchain where transactions are recorded. [Learn More](./defi-stack-settlement.md)
- **Examples**: Ethereum, Solana, Avalanche, Binance Smart Chain.
- **Purpose**: Provides security, decentralization, and consensus mechanisms. This is where tokens live and smart contracts are executed.

---

## 2. Asset Layer

- **What it is**: Digital assets that are native or issued on the blockchain. [Learn More](./defi-stack-asset.md)
- **Examples**:
  - Native tokens: ETH, SOL, BNB.
  - Tokenized assets: USDC (stablecoins), Wrapped Bitcoin (WBTC), NFTs.
- **Purpose**: Represents value, whether it's currency, a share of an asset, or collateral.

---

## 3. Gateway

- **What it is**: A DeFi gateway provides access to decentralized financial services, often bundling various layers of the DeFi stack (especially the Application, Aggregation, and sometimes Protocol layers) into a single, easy-to-use platform or tool. [Learn More](./defi-stack-gateway.md)
- **Examples**:
  - Wallets
  - Dashboards
  - dApp / Browser interfaces
  - On-Ramp Platforms
  - Web3 APIs/SDKs

---

## 4. Protocol Layer (Core DeFi Logic)

- **What it is**: Smart contracts that implement financial logic. [Learn More](./defi-stack-protocol.md)
- **Examples**:
  - **Lending/borrowing**: Aave, Compound.
  - **Decentralized exchanges (DEXs)**: Uniswap, Curve.
  - **Derivatives**: Synthetix, dYdX.
  - **Stablecoins protocols**: MakerDAO (DAI), Frax.
- **Purpose**: Enables financial services like trading, lending, staking, etc., without centralized control.

---

## 5. Application Layer (User Interface)

- **What it is**: The front-end apps that users interact with [Learn More](./defi-stack-application.md).
- **Examples**: MetaMask, Zapper, DeBank, DEX frontends.
- **Purpose**: Simplifies the user experience, letting people access DeFi protocols easily via web or mobile apps.

---

## 6. Aggregation Layer (Composability / Meta-Protocols)

- **What it is**: Tools and platforms that aggregate multiple DeFi services into one interface [Learn More](./defi-stack-aggregation.md).
- **Examples**: Yearn Finance (yield aggregator), 1inch (DEX aggregator), Instadapp (DeFi manager).
- **Purpose**: Helps users find the best rates, automate strategies, or combine services across protocols.

---

## Optional Add-ons

- **Oracles**: Bring real-world data (prices, events) onto the blockchain.
  - *Examples*: Chainlink, Pyth.
- **Bridges**: Allow assets to move between blockchains.
  - *Examples*: Wormhole, Multichain.
- **Identity & Compliance**: Decentralized identity (DID), KYC modules for compliant DeFi.

---

## Summary Table

| Layer               | Purpose                                            | Example Projects                    |
|--------------------|----------------------------------------------------|-------------------------------------|
| Settlement Layer   | Blockchain infrastructure                          | Ethereum, Solana                    |
| Asset Layer        | Digital tokens/value                               | USDC, WBTC, NFTs                    |
| Gateway            | Wallets/Dashboards                                 | Cosmostation, Metamask              |
| Protocol Layer     | Financial logic via smart contracts                | Aave, Uniswap, MakerDAO             |
| Application Layer  | User interface for interaction                     | MetaMask, Zapper                    |
| Aggregation Layer  | Combines services, optimizes user experience       | Yearn, 1inch, Instadapp             |
