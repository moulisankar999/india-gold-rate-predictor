> **Moved.** This predictor is now the **Prediction** tab of [Gold · Silver · FX](https://moulisankar999.github.io/gold-silver-fx-dashboard/#predict) (repo `moulisankar999/gold-silver-fx-dashboard`). This site redirects there; the scraper workflow here is disabled. The old page is kept in `legacy/`.

# India Gold Rate Predictor

Live: https://moulisankar999.github.io/india-gold-rate-predictor/

Predicts today's India rates in INR — 24K (999) and 22K (916) gold per 1 g / 10 g, silver per 1 g / 1 kg,
ex-GST and with 3% GST — from live COMEX and MCX prices, calibrated against actual IBJA and Chennai (MJDTA) prints.

## Model
- **A · COMEX landed cost**: `COMEX USD/oz × USD/INR ÷ 31.1035 × 1.15 (customs duty)` → × (1 + premium)
- **B · MCX based**: `MCX near-month ÷ (1 + 6.5% × days-to-expiry / 365)` → × (1 + offset)
- Premium / offset: exponentially weighted mean (half-life 1.5 prints) of `actual / raw − 1` over the last
  10 IBJA prints (6 for Chennai).
- Blend: weights ∝ 1 / MSE of each method over its last 10 walk-forward backtest errors.
- Range: ± 1.28 × RMSE of the blended walk-forward errors (≈80%).
- 22K = 24K × IBJA's 916/999 ratio. Chennai 24K = 22K × 24/22.

## Data (GitHub Actions → `data` branch → `data/latest.json`)
`scraper/scrape.py` every ~10 min, 7 AM–midnight IST Mon–Sat (self-chaining runs; GitHub drops most `*/10` crons):
Yahoo Finance GC=F, SI=F, USDINR=X (15-min bars for 60 days); mcxlive.org gold/silver (quote, 5-day 1-min intraday,
daily closes); IBJA board + 30-day PDF; MJDTA board; goodreturns.in Chennai 10-day history; gold-api.com spot.
`history.json` accumulates MCX 5-min points and every print so the calibration/backtest window keeps growing.

Estimate only, not financial advice.

Tests: `npm test`
