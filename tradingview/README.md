# MARKET DELIVERY & LIQUIDITY ENGINE PRO (MDLE PRO)

A non-repainting, multi-timeframe institutional **market delivery** model for
TradingView, written in **Pine Script v6** and optimized for the **1-Hour**
execution timeframe. It follows ICT / Smart Money Concepts and models price as
a strict delivery cycle implemented as a **state machine**:

```
Liquidity Build → Liquidity Sweep → Displacement → OTE Retracement → Target Delivery
```

A setup is only valid when these states occur **in order**.

> **File:** [`MDLE_PRO.pine`](./MDLE_PRO.pine) — paste into the TradingView
> Pine Editor and *Add to chart*.

---

## How it works

### Timeframe structure
| Role | Timeframe |
|------|-----------|
| HTF Bias #1 | Daily (configurable) |
| HTF Bias #2 | 4H (configurable) |
| Execution | 1H (the chart you run it on) |

Direction is **gated** by HTF bias — long setups require a bullish Daily **and**
4H read; short setups require both bearish. HTF values are pulled with
`lookahead_off` and read on the **previous confirmed** HTF candle, so bias never
repaints intrabar.

### The five states
1. **Liquidity Build / Consolidation** — equal highs/lows (EQH/EQL), range
   compression vs ATR, equilibrium (50%). Drawn as a neutral-gray box.
2. **Liquidity Sweep** — a wick takes out sell-side (EQL / PDL / PWL / swing low)
   or buy-side (EQH / PDH / PWH / swing high) liquidity **and rejects**.
3. **Displacement** — a body `> 1.5 × ATR` in the bias direction that produces a
   **Market Structure Shift / BOS** and (ideally) a **Fair Value Gap**.
4. **OTE Retracement** — Fibonacci is auto-anchored across the expansion leg; the
   **62%–79%** Optimal Trade Entry zone is highlighted (primary level **70.5%**).
5. **Target Delivery** — the active **Draw-on-Liquidity** target is selected and
   tracked; TP1/TP2/TP3 and SL are managed to completion.

### Confluence engine (0–100)
| Factor | Points |
|--------|-------:|
| HTF bias aligned | 20 |
| Liquidity sweep | 20 |
| Displacement | 20 |
| Fair Value Gap | 15 |
| Order Block | 10 |
| OTE zone | 10 |
| Session alignment (overlap = 5, killzone = 3) | 5 |

**Grading:** A+ (90–100), A (80–89), B (70–79), C (50–69). **Nothing below 70 is
signalled**, and a trade signal additionally requires score `≥ Minimum Score`
(default **80**) while price is inside the OTE zone.

### Risk management
- **Stop:** beyond the lower-of (sweep extreme / order-block extreme) **± 0.25 ATR**
  buffer.
- **Targets:** TP1 ≈ 1:1 (internal liquidity), TP2 ≈ 1:2 (prev day H/L),
  TP3 ≈ 1:3+ (prev week H/L / major swing) — all RR-configurable.

---

## Non-repainting design
- HTF requests use `lookahead_off` and the last **confirmed** HTF bar.
- All state transitions and signals fire on **`barstate.isconfirmed`** (bar close),
  so printed BUY/SELL labels never disappear.
- Pivots use a fixed right-hand offset (confirmed only after *N* bars).
- Drawing objects are pooled/deleted and bounded by the `max_*_count` limits.

## Alerts
Liquidity Sweep · Displacement Confirmed · OTE Zone Entered · BUY · SELL ·
TP1/TP2/TP3 Hit · Target Delivery Completed · State Change.

## Dashboard & signal panel
A configurable dashboard shows HTF bias, current state, sweep/displacement/OTE
status, confluence score, grade, entry, stop, TP1–3, current liquidity target,
distance to target, RR and session. The signal panel summarises any active
setup.

---

## Disclaimer
For research and education only. This is **not** financial advice. Markets carry
risk; always validate on your instrument and timeframe, and forward-test before
risking capital.
