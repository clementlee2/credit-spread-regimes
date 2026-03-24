# credit-spread-regimes
Fitting a 3-state Gaussian HMM on US IG/HY OAS spreads (FRED, 2000–2024, n = 6,275) to detect persistent credit market regimes and evaluate a regime-conditional long/short signal. v2 adds walk-forward validation, a more realistic return proxy, robustness checks, a European extension, and transaction cost analysis.
 
## Notebooks
 
| Notebook | Description |
|---|---|
| `credit_regime_detection.ipynb` | v1 — Core HMM, feature engineering, in-sample backtest, regime characterization |
| `credit_regime_detection_v2.ipynb` | v2 — Walk-forward validation, LQD total return proxy, robustness checks, iTraxx EU extension, transaction cost sensitivity |
 
Start with v1 for the methodology, v2 for the validation layers.
 
## Methodology
 
**Features** (inputs to the HMM, not raw spreads):
 
| Feature | Captures |
|---|---|
| Daily IG OAS change | Spread momentum |
| 252-day rolling z-score of IG OAS | Relative level |
| 21-day realized vol of OAS changes | Volatility regime |
| HY/IG ratio 5-day momentum | Cross-asset risk appetite |
 
**Model:** `GaussianHMM`, 3 states, full covariance, 1,000 iterations. States labelled post-fit by mean IG OAS level: Tight, Recovery, Stress.
 
**Signal:** Long credit in Tight/Recovery, short in Stress. 1-day execution lag.
 
**P&L proxy:** `−ΔOAS` (v1); LQD ETF total return scaled to bps-equivalent (v2).
 
## Results
 
### v1 — In-sample (train = test, 2000–2024)
 
| Metric | Strategy | Buy & Hold |
|---|---|---|
| Sharpe | 0.36 | 0.11 |
| Max Drawdown | −3.9 bps | −5.8 bps |
 
> ⚠️ **In-sample only.** The model is fitted and evaluated on the same data. See v2 for out-of-sample results.
 
Regime mean durations: Tight 130 days · Recovery 59 days · Stress 30 days. Stress regime preceded elevated equity volatility around the GFC, COVID, and 2022 rate shock.
 
### v2 — Out-of-sample (walk-forward, 3yr train / 1yr test)
 
| Metric | Result |
|---|---|
| OOS windows | 21 |
| Mean OOS Sharpe — Strategy | 1.27 |
| Mean OOS Sharpe — B&H | 0.11 |
| Windows outperforming B&H | 57% |
 
### v2 — Return proxy comparison (in-sample, full period)
 
The choice of P&L proxy materially affects conclusions:
 
| Proxy | Strategy Sharpe | B&H Sharpe | Strategy P&L | B&H P&L |
|---|---|---|---|---|
| `−ΔOAS` (v1 proxy) | 0.30 | 0.16 | 2.9 bps | 1.6 bps |
| LQD total return | 0.37 | **0.64** | 3.6 bps | **6.2 bps** |
 
The `−ΔOAS` proxy captures spread direction but ignores carry. The LQD-based proxy, which includes carry and mark-to-market dynamics, shows buy-and-hold outperforming on both Sharpe and total P&L. The regime signal identifies spread direction correctly; the edge disappears once the carry component of credit returns is included.
 
## v2 Validation Layers
 
**Layer 1 — Walk-forward validation** (3yr train / 1yr test, rolling)
Addresses the in-sample overfitting in v1. OOS Sharpe and win-rate vs. B&H reported across all 21 windows.
 
**Layer 2 — LQD total return proxy**
Replaces `−ΔOAS` with LQD ETF daily returns (scaled) as a more realistic IG credit return approximation. Bloomberg alternative: `IBOXUMAE Index`. See return proxy comparison above.
 
**Layer 3 — Robustness**
 
- BIC/AIC across 2–4 states: 4 states achieves the best information criteria, but 3 states is chosen for economic interpretability (Tight/Recovery/Stress).
- Regime distribution stability: mean proportions are Tight 48.5%, Recovery 24.4%, Stress 27.1% across walk-forward windows — with standard deviations of 32%, 23%, and 30% respectively, indicating meaningful label instability across periods.
- Posterior probability threshold sweep (50%–90%): Sharpe improves from 0.37 at threshold 0.50 to 0.43 at threshold 0.85–0.90, at the cost of ~3% reduced market exposure.
 
**Layer 4 — iTraxx European extension**
 
> ⚠️ **iTraxx data is simulated** with a realistic correlation structure. Replace the simulation block with Bloomberg CSV exports (`ITRXEBE5 Index`, `ITRXEXE5 Index`) for production results. All EU findings below are illustrative only.
 
Same HMM methodology applied to iTraxx Main/Crossover with OAT/Bund spread as an additional feature. EU Stress regime occupies ~25% of the sample. US→EU stress concordance is largely contemporaneous (optimal lag: 0 days, corr: 0.26), suggesting the two markets co-move rather than the US leading.
 
**Layer 5 — Transaction cost sensitivity**
 
> ⚠️ **The strategy does not survive realistic transaction costs.**
 
With 112 position changes over the full sample (avg daily turnover ~1.8%), the breakeven cost is effectively **0 bps/trade**. At CDX IG bid-ask (~0.25 bps), net Sharpe collapses from +0.36 to **−1.16** and total P&L turns from +3.8 bps to **−24.2 bps**. This is the primary constraint on any live implementation. Mitigation paths: reduce signal frequency, apply a higher posterior threshold (reduces turnover to ~2.7%), or size positions against realized vol to cut the number of full flips.
 
## Data
 
All data is fetched at runtime — no files required.
 
| Series | Source | Notes |
|---|---|---|
| `BAMLC0A0CM` — US IG OAS | FRED | No API key required |
| `BAMLH0A0HYM2` — US HY OAS | FRED | No API key required |
| LQD ETF | Yahoo Finance (`yfinance`) | CDX IG total return proxy |
| iTraxx Main / Crossover | **Simulated** | Replace with Bloomberg export |
 
## Setup
 
```bash
pip install hmmlearn pandas-datareader fredapi yfinance matplotlib seaborn scipy
```
 
Run `credit_regime_detection.ipynb` first, then `credit_regime_detection_v2.ipynb`. No additional configuration needed for FRED/Yahoo data.
 
## Limitations
 
- **Transaction costs:** The `−ΔOAS` proxy strategy breaks even at ~0 bps/trade. Any real implementation requires either a much lower-turnover signal or a carry-inclusive return model.
- **Carry:** The LQD proxy results show buy-and-hold outperforms once carry is included. The regime signal adds directional value but does not generate alpha net of the carry component forgone during short periods.
- **iTraxx:** European results are illustrative pending real Bloomberg data.
- **No position sizing:** A vol-targeting overlay would improve risk-adjusted returns and reduce the impact of individual regime transitions.
- **Fixed training window:** Walk-forward uses a fixed 3yr window; adaptive sizing is a natural extension.
- **Regime label instability:** Regime proportions vary substantially across walk-forward windows (σ ≈ 23–32pp), which complicates out-of-sample regime interpretation.
- **Execution:** Signal assumes costless execution at close; real slippage compresses net Sharpe further.
