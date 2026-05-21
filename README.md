# Solana DEX LP Profitability: Empirical Analysis of Raydium and Orca SOL/USDC Pools

First systematic empirical study of LP profitability on Solana DEX, based on 2 years of on-chain data from Raydium CLMM and Orca Whirlpool SOL/USDC 0.04% pools.

## Research question

Are LPs on Solana CLMM DEXs profitable? Following the methodology of Loesch et al. (2021) for Uniswap v3, this study provides the first systematic empirical baseline for Solana — measuring fees earned, impermanent loss, and gas costs across two years of position-level data, with cohort analysis by duration, size, range width, and drawdown behavior.

## Status

🟡 **Pre-registration phase.** Methodology locked. Analysis begins upon grant approval (Superteam, requested).

## Methodology

See [methodology.md](./methodology.md). The methodology was pre-registered before any data analysis began to prevent specification search. All cohort thresholds, drawdown criteria, and pricing methodology are fixed in advance.

## Scope

- **Pools:** Raydium CLMM SOL/USDC 0.04% + Orca Whirlpool SOL/USDC 0.04%
- **Period:** 2 years (dates fixed in methodology.md upon analysis start)
- **Data source:** Helius archival RPC + Pyth historical pricing
- **Levels of analysis:** sub-position / NFT / wallet

## Author

**Artur Sundetov**
Founder, Gexabyte (blockchain development, est. 2018) · Founder, BalanceX

### Conflict of interest disclosure

The author is the founder of BalanceX, a Solana market-making protocol operating in an adjacent domain to this research. Mitigations:

1. All data published open-source as public good
2. All code published under MIT license
3. Findings published regardless of impact on BalanceX narrative
4. Results give BalanceX no preferential access — the ecosystem receives everything simultaneously upon publication

Gexabyte is a blockchain development consultancy and is not affiliated with any protocol covered in this research.

## License

Code: MIT (see [LICENSE](./LICENSE))
Data and report (when published): CC-BY 4.0

## Citation

Citation file will be added upon publication.

## Contact

For questions about methodology or reproduction: [Twitter @sundetar]
