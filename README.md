# BTC cent forecast 01 — entry

Forecast Bitcoin's US-dollar price 240 minutes after each checkpoint, from
the case's last 72 hours of 1-minute closes. Score: mean absolute error in
cents (lower better); the house baseline is persistence (the checkpoint's
own close).

## Method

Log-space point forecast of the 240-minute log-return as a linear
combination of robust trend features, then `guess = last close * exp(r)`:

- `m240` — median of all overlapping 240-minute log-returns in the case
- `m240_24` / `m240_12` — medians of the most recent 1,440 / 720 of those
- `d4h`, `d1h`, `d12h` — log-returns over the last 240 / 60 / 720 minutes
- `dev24`, `dev4` — last log-price minus the median log-price of the last
  1,440 / 240 minutes

Weights (tuned locally, see `main.py` header): 6.31, −2.27, 1.85, −0.20,
−0.18, 0.01, −0.42, +0.48, clipped to ±5% of the last close.

Bitcoin's 240-minute returns show short-horizon continuation in this
regime, and the recency-weighted median combination captures it; the small
drift and deviation terms correct it.

## Local scores (mean absolute error, cents)

| set | entry | persistence baseline |
| --- | --- | --- |
| official practice p00..p05 | 19,427 | 44,522 |
| extra 20 checkpoints | 29,117 | 45,718 |
| dense 45 checkpoints | 24,732 | 41,330 |
| out-of-window Binance 137 checkpoints | 57,259 | 68,116 |

Wins vs persistence: 6/6 official, 16/20 extra, 36/45 dense, 89/137
out-of-window. The worst case on every set is still better than
persistence on that same case.

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Standard library only, offline, deterministic, ~0.003 s/case.
