# Algorithmic Stablecoins Explained

Algorithmic stablecoins are a type of cryptocurrency that aims to maintain a stable value, typically pegged to a fiat currency like the US dollar, using **algorithms and smart contracts** instead of traditional collateral like cash or other cryptocurrencies.

## How They Work

Algorithmic stablecoins employ various mechanisms to maintain their peg:

- **Supply Adjustment:** The most common method involves algorithms that automatically adjust the supply of the stablecoin based on its price.
  - If the price goes **above** the peg, the algorithm **mints** new tokens, increasing the supply and theoretically pushing the price down.
  - If the price falls **below** the peg, the algorithm **burns** tokens, reducing the supply and aiming to increase the price.
- **Seigniorage:** Some algorithmic stablecoins use a two-token system. One token is the stablecoin, and the other is a governance or "share" token.
  - When the stablecoin's price is **above** the peg, the protocol mints more stablecoins and distributes them to holders of the governance token.
  - When the price is **below** the peg, the protocol may use incentives, like bonds, to encourage users to buy and burn the stablecoin, reducing its supply.
- **Rebasing:** Rebasing stablecoins adjust the total supply of tokens, proportionally affecting each holder's balance.
  - If the price is **above** the peg, the supply **increases**, and everyone's balance goes up.
  - If the price is **below** the peg, the supply **decreases**, and everyone's balance goes down.
  - The idea is that the market capitalization adjusts while individual holdings maintain their proportion of the total value.
- **Fractional-Algorithmic:** These stablecoins combine collateralization with algorithmic mechanisms. A portion of the stablecoin's value is backed by collateral, while the rest is managed algorithmically.

## Examples of Algorithmic Stablecoins

- **Ampleforth (AMPL):** Uses a rebasing mechanism to adjust supply daily based on price fluctuations.
- **Frax (FRAX):** A fractional-algorithmic stablecoin that is partially backed by collateral and partially stabilized by algorithms.
- **TerraClassicUSD (USTC):** Formerly TerraUSD (UST), this is a notable example of an algorithmic stablecoin that failed catastrophically, highlighting the risks involved. It used a two-token system with LUNA to maintain its peg.
- **Dai (DAI):** While often categorized as a crypto-collateralized stablecoin, DAI also uses algorithmic mechanisms to maintain its peg. It's over-collateralized with other cryptocurrencies, and the protocol adjusts collateral requirements to keep DAI's value close to $1 USD.
- **USDD (Decentralized USD):** An algorithmic stablecoin on the Tron blockchain that follows a similar design to the failed UST.

## Risks of Algorithmic Stablecoins

Algorithmic stablecoins are considered **riskier** than fiat-backed or over-collateralized stablecoins because their stability relies on complex algorithms and market demand. If the algorithms fail to maintain the peg, or if market confidence is lost, these stablecoins can experience significant price drops, as seen with the collapse of UST.

Despite the risks, algorithmic stablecoins offer the potential for more **decentralized and capital-efficient** stablecoins, and research and development in this area continue.
