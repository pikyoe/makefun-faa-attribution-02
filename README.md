# polymarket-faa-attribution-02

Forecasts the Yes price of every market in a Polymarket case for the 240
minutes after its checkpoint, five quantiles (0.05, 0.25, 0.5, 0.75, 0.95)
per minute. Built for the make.fun competition "Forecast Polymarket's FAA
outage attribution prices minute by minute, 4 hours ahead".

## Model

A martingale with robust, locally-estimated volatility:

- **Median = last observed price.** Prediction-market prices are a
  martingale to first order; drift, trend and mean-reversion variants
  all scored worse on the practice cases.
- **Spread** grows with `sqrt(horizon)` around the median, scaled by a
  per-market sigma estimated robustly (median absolute deviation) from
  recent 1-minute price moves: 70% last 60 minutes, 30% last 6 hours,
  floored at 0.001 and capped at 0.008 price units per sqrt-minute.
  MAD is used instead of the standard deviation because prices move in
  steps with rare large jumps — one 20-point jump must not blow the
  bands up for four hours.
- **Heavy-tailed shape:** 1-minute moves are mostly zero (~88% of
  minutes) with occasional jumps, so inner quantiles are narrower than
  normal (x0.85) and outer quantiles wider (x1.5).
- All quantiles are clipped to [0, 1], rounded to 6 decimals and forced
  non-decreasing. Short histories and constant prices are handled via
  the sigma floor / fallback.

Practice score (p00–p04 scored against the following case's recorded
prices; p05's answers lie outside the kit): mean pinball loss **~0.348**,
vs ~0.393 for the house baseline on the same cases. Median error ~0.88,
90% coverage ~0.91.

## Run

```sh
python3 main.py
```

Reads `$MAKEFUN_INPUT_DIR` (else `/input/`) — every `*.in` file, in name
order — and writes one `<id>.out` per case to `$MAKEFUN_OUTPUT_DIR`
(else `/output/`). Python 3.13, standard library only, no network,
deterministic. Runs in well under a second per case.

Local test against the kit's practice cases:

```sh
MAKEFUN_INPUT_DIR=<kit>/input MAKEFUN_OUTPUT_DIR=out python3 main.py
```
