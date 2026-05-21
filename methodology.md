# Methodology

**Solana DEX LP Profitability: Empirical Analysis of Raydium and Orca SOL/USDC Pools**

This document describes the methodology for an empirical study of LP profitability on Solana CLMM DEXs. It is pre-registered before any data analysis begins to prevent specification search.

---

## 1. Object of study

Two CLMM SOL/USDC pools on the two main Solana DEXs, on the same fee tier, enabling a clean cross-protocol comparison:

| Protocol | Pair | Fee tier | TVL threshold |
|---|---|---|---|
| Raydium CLMM | SOL/USDC | 0.04% | >$5M ✓ |
| Orca Whirlpool | SOL/USDC | 0.04% | >$5M ✓ |

Analysis period: 2 years, with calendar dates fixed before analysis begins. At scope finalization, both pools have TVL above the $5M threshold (verified via Raydium and Orca UIs). Exact pool addresses and calendar dates are published in this document before extraction starts.

---

## 2. Data sources

- **On-chain history** — Helius archival RPC. We extract all position lifecycle events (OpenPosition, IncreaseLiquidity, DecreaseLiquidity, ClosePosition, CollectFee) and all swap events for the full period on both pools.
- **Pricing** — Pyth historical as primary source. Binance/Coinbase reference as validation.

---

## 3. Three levels of analysis

Analysis is performed in parallel at three levels. Each level answers a different research question.

- **Sub-position (imputed)** — unit for mathematically correct P&L. Each IncreaseLiquidity event creates a new sub-position with its own entry price. FIFO accounting on DecreaseLiquidity events.
- **Position (NFT)** — unit for descriptive statistics. One CLMM Position NFT equals one position. Position P&L equals the sum of P&L of all its sub-positions.
- **Wallet (within-pool)** — unit for behavioral analysis. All positions of one wallet address within one pool are aggregated. Addresses are not linked across pools — no heuristic clustering.

---

## 4. P&L computation

For each sub-position:

```
P&L = Fees_earned − |IL| − Gas
```

- **Fees earned** — cumulative fees over the sub-position's lifetime. Computed via CLMM `feeGrowthInside` snapshots × liquidity × time in-range.
- **IL (impermanent loss)** — difference between Value_LP (token value at exit at pool price) and Value_HODL (token value if the LP had simply held the tokens at entry price). Standard CLMM formula with range correction is used.
- **Gas** — total Solana transaction costs across all sub-position events. Headline results report pre-gas P&L; gas impact is presented in a separate section broken down by cohort.

---

## 5. Pre-registered cohort thresholds

All cohort thresholds are locked before analysis begins in this document. This protects against specification search and motivated reasoning.

| Dimension | Categories |
|---|---|
| Position duration | Short: <7 days \| Medium: 7–30 days \| Long: >30 days |
| Position size (USD at entry) | Retail: <$1k \| Mid: $1k–$50k \| Large: >$50k |
| Range width | Tight: ±0–10% \| Medium: ±10–50% \| Wide: ±50–200% (>±200% excluded) |
| Drawdown behavior | Held: Δ liquidity <20% \| Partial: 20–80% reduction \| Capitulated: >80% withdrawal or full close |

---

## 6. Drawdown behavioral analysis

- **Episode identification:** SOL drop of ≥25% from a local high within a ≤14-day window.
- **Behavior observation window:** [T−3 days, T+14 days] from the local trough.
- **LP classification:** Held (liquidity change <20%), Partial exit (20–80% reduction), Capitulated (>80% withdrawal or full close).
- **Outcome:** comparison of final P&L across the three cohorts over the entire analysis window.

---

## 7. Headline metrics

1. Total fees vs Total IL vs Net P&L per pool and in aggregate
2. % of profitable positions and % of profitable wallets (reported separately at position-level and wallet-level)
3. Cross-protocol comparison on the identical fee tier: Raydium 0.04% vs Orca 0.04% — the primary headline comparison
4. P&L breakdown by each cohort dimension (duration, size, range width, drawdown behavior)
5. Gas contribution: average % gas of position size by cohort

---

## 8. Robustness checks

Testing the stability of headline results against methodological variations. Performed after core analysis.

- **Pricing sensitivity** — rerun with Binance/Coinbase prices instead of Pyth; compare.
- **Mark-to-market vs closed-only** — main result with open positions (mark-to-market at cutoff) vs closed positions only.
- **Cohort threshold sensitivity** — shift size and duration thresholds by ±10%; verify cohort ranking stability.
- **Methodology sensitivity** — compare imputed sub-positions vs naive (one NFT with averaged entry price).

---

## 9. Open positions at cutoff

Positions still open at the end of the analysis window are included in main analysis via mark-to-market at the cutoff date. A separate section reports analysis restricted to closed positions only as a robustness check.

---

## 10. Reproducibility

1. `methodology.md` — this document, published in public GitHub before analysis begins
2. Full data extraction and processing pipeline — public GitHub, MIT license
3. Processed dataset at all three aggregation levels — public GitHub, CSV format
4. README with full reproduction instructions

---

## 11. Relation to prior work

The methodology structurally follows Loesch, Hindman, Richardson, Welch (2021) — *Impermanent Loss in Uniswap v3* — the canonical Topaz Blue × Bancor study on empirical LP P&L on Uniswap v3 ([arXiv:2111.09192](https://arxiv.org/abs/2111.09192)). This enables direct comparability of results between the Ethereum and Solana ecosystems.

Extensions relative to Topaz Blue:

- Gas added as a separate cost line (on Ethereum gas dominates and is often handled implicitly via period selection; on Solana cheap gas requires explicit accounting for actively rebalancing LPs)
- Full cohort analysis across 4 dimensions
- Solana-specific behavioral analysis during drawdown episodes
- Cross-protocol comparison on identical fee tier (Raydium vs Orca on 0.04% SOL/USDC)
- Extended robustness checks (4 sensitivity tests)

---

## 12. Cross-wallet ownership clustering

Cross-wallet ownership clustering is not performed — we do not attempt to identify which different addresses belong to the same owner. This is a deliberate methodological choice, following the approach of Topaz Blue × Bancor (Loesch et al. 2021).

**Rationale:**

- Heuristic clustering methods (common deposit, funding, behavioral) always produce contestable results — different methods give different groupings, and there is no objective correctness criterion
- On Solana, wallets are cheap to create, and many applications programmatically generate a new address per position — heuristics that work on Ethereum produce weaker results on Solana
- For the core research question (how LP capital behaves in pools), per-position and per-wallet-within-pool analyses are sufficient
- Following Topaz Blue methodology enables direct comparability of results between Ethereum and Solana

**Implication for interpreting results:** "unique wallets" counts overestimate the number of unique LP entities, since professional market makers routinely operate multiple wallets. This is a known limitation, explicitly noted in the Limitations section of the final report.

**Exception to this rule:** when a position is owned by a smart contract (e.g., a vault aggregator built on top of Raydium or Orca), we treat the contract address as the wallet-level subject rather than the position NFT holder. This is clean aggregation based on on-chain code structure, not heuristic clustering — it is methodologically safe and more accurately reflects ownership structure for delegated liquidity management products.

---

## 13. What else is out of scope

- Standard AMM (constant product) Raydium pools — focus is on CLMM only
- Stable/stable pools (USDC/USDT, etc.) — left for future work
- Within-protocol cross-tier comparison (Raydium 0.01% vs 0.04% vs 0.05% and similarly for Orca) — left for future work; this study focuses on a clean cross-protocol comparison on identical fee tier
- LVR (Loss-Versus-Rebalancing) — reserved for Phase 2, building on the theoretical work of Nezlobin & Tassy (2025, [arXiv:2505.05113](https://arxiv.org/abs/2505.05113))
- MEV decomposition (JIT liquidity, sandwich attacks) — out of scope for this study

---

## References

- Loesch S., Hindman N., Richardson M. B., Welch N. (2021). *Impermanent Loss in Uniswap v3.* [arXiv:2111.09192](https://arxiv.org/abs/2111.09192).
- Aigner A., Dhaliwal G. (2021). *Uniswap: Impermanent Loss and Risk Profile of a Liquidity Provider.*
- Milionis J., Moallemi C. C., Roughgarden T., Zhang A. L. (2022). *Automated Market Making and Loss-Versus-Rebalancing.* [arXiv:2208.06046](https://arxiv.org/abs/2208.06046).
- Nezlobin A., Tassy M. (2025). *Loss-Versus-Rebalancing under Deterministic and Generalized block-times.* [arXiv:2505.05113](https://arxiv.org/abs/2505.05113).
