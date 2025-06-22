# Index Rebalancing Alpha Strategy

This project investigates price anomalies that arise when equities **enter or leave** major U.S. benchmarks (S&P 400, 500, 600).  
We build **event‑driven, beta‑hedged portfolios** and **grid‑search key parameters** to see if consistent alpha survives transaction costs and out‑of‑sample testing.

**Author**: Brian Plotnik  
**Focus**: Event‑driven trading, parameter tuning, risk‑adjusted evaluation

---

## Workflow Overview

1. **Data ingest** – download Yahoo Finance OHLCV price data and parse an index‑event sheet containing:
   - `Announced` date (index provider press release)
   - `Trade Date` (effective inclusion / deletion) 
2. **Feature engineering** – build overnight **gap**, 20‑day **event volume**, rolling 60‑day **β** vs SPY.
3. **Strategy engines** – Python functions generate timestamped trades and create a hedged equity curve.
4. **Parameter search (train sample)** – grid‑search on events **pre‑2024‑01‑01**.
5. **Out‑of‑sample test** – freeze parameters and run on 2024‑01‑01 → present.  
6. **Diagnostics** – cumulative P&L, rolling Sharpe, drawdowns, hit‑rate tables.

---

## Strategy Variants & Parameter Grids

| Variant | Entry | Exit | Grid‑searched parameters | Hedge |
|---------|-------|------|--------------------------|-------|
| **Momentum** (additions & deletions) | Open *day after* announcement | `Trade Date – k` (k ∈ 0‑3) | **Gap** threshold g ∈ {2%, 3%, 4%}  •  **Volume spike** v ∈ {3×, 5×, 7× ADV}  •  **Drift filter** d ∈ {None, 3%, 5%}  •  **Event type** ∈ {quarterly, ad‑hoc, all} | β‑weighted SPY |
| **Index‑level Momentum** | Same as above but evaluated **per index** (400/500/600) | k ∈ {0‑2} | Uses best (g,v,d) from global search, sweeps k & index | β‑weighted SPY |
| **Intraday Mean‑Reversion** | Close on `Trade Date` (captures event‑day overshoot) | Hold `h` days (h ∈ 1‑5) | **Hold length** grid only (simpler model) | β‑weighted SPY |

All grids optimise **Sharpe** on the training window; the **top‑Sharpe parameter tuple** is then applied, untouched, to the 2024‑2025 out‑of‑sample window.

---

## Headline Results

| Strategy | Best Params (train) | Sharpe OOS | Alpha (bps) | Notes |
|----------|--------------------|------------|-------------|-------|
| Momentum (All indices) | g = 3%, v = 5 × ADV, d = None, k_exit = 1 | 0.92 | 240 | Drift persists until T−1 |
| Momentum (S&P 600 only) | same g,v,d, k = 0 | 0.88 | 210 | Slightly lower slippage |

---

## Key Takeaways

- **Exit timing matters** – selling one trading day before inclusion added ~40 bps versus holding to T0.
- **No universal gap threshold** – 3 % worked best overall, but smaller caps (S&P 600) reacted at 2 %.
- **One‑day mean‑reversion** outperformed longer holds; event drift unwinds quickly.
- **β‑hedging reduced drawdown by ≈35 %** without diluting alpha.

---

## Libraries

`pandas • numpy • matplotlib • seaborn • scipy • statsmodels • sklearn • openpyxl • fredapi`

---

## Repo Layout

```text
index-rebalance-alpha/
├── index_event_alpha_study.ipynb
├── README.md             ← you are here
```

---

## Quick‑Start

```bash
git clone https://github.com/yourusername/index-rebalance-alpha.git
cd index-rebalance-alpha
python -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
jupyter notebook index_event_alpha_study.ipynb
```

---

## Limitations & Next Steps

- Daily bars / close‑to‑open fills; intraday slippage not modelled  
- Next: walk‑forward CV, ETF flow overlay, ML‑ranked signal strength.

---

## Disclaimer

Educational illustration only; not investment advice. Input data not redistributed.

---
