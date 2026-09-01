# Mech Model FVG/iFVG Indicator

A TradingView Pine Script (v5) indicator implementing a mechanical
swing-break + HTF Fair Value Gap + LTF Inverse FVG entry model, similar in
spirit to PB Blake's "mech model." Works on any TradingView chart, including
**FX Replay** (fxreplay.com), since it renders on TradingView's charting
library — the indicator triggers on replay bars exactly as it would live.

## Logic

1. **Swing structure** — tracks pivot highs/lows (`Swing Lookback` bars each
   side).
2. **Break of structure** — when price closes (or wicks, if configured)
   beyond the most recent unbroken swing high/low.
3. **Key area filter** — the break only "arms" a setup if the broken swing's
   *price level* sits inside an unmitigated Fair Value Gap on the HTF
   (15m minimum, default 15m).
4. **Entry trigger** — while armed, the script watches the LTF (30s minimum;
   defaults to your chart's own timeframe) for a Fair Value Gap to form and
   then fully invert — i.e., a candle **closes** completely through it. That
   inversion candle's close must also land back inside the HTF key area.
   When it closes, a `BUY`/`SELL` label prints on that bar.
5. A setup disarms after it fires, after a bar timeout, or (optional) if
   price closes back past the swing that triggered it (failed structure).

## Install

1. Open TradingView (or FX Replay, which uses the same charts) → **Pine
   Editor**.
2. Paste the contents of `mech-model-fvg-ifvg.pine`.
3. **Add to Chart**.
4. Recommended starting setup: view chart at your intended LTF (e.g. 1m or
   lower on FX Replay), leave `Use Current Chart Timeframe for iFVG` on, and
   set `HTF Timeframe` to 15m or higher.

## Key inputs

- `Swing Lookback` — pivot sensitivity; smaller = more (noisier) swings.
- `HTF Timeframe` — the higher timeframe FVGs are pulled from (15m+).
- `HTF FVG Mitigation` — whether a zone is invalidated on any touch or only
  a full close-through.
- `Use Current Chart Timeframe for iFVG` — off lets you specify a custom LTF
  (e.g. `30S`) independent of your chart's timeframe (requires a data plan
  with sub-minute resolution).
- `Max Bars to Wait for iFVG After Break` — setup timeout.
- `Only First Qualifying iFVG per Setup` — fire once per armed setup vs.
  every qualifying inversion.
- `Disarm if Price Closes Back Past Broken Swing` — cancels the setup on
  failed structure.

Alerts are wired via `alertcondition()` for both BUY and SELL so you can set
TradingView alerts without polling the chart.

## Notes / caveats

- "PB Blake's mech model" isn't a documented public spec, so this is an
  interpretation of the rules you described (swing break confirmed by an
  untapped HTF FVG, entry confirmed by an LTF inverse FVG closing inside
  that same zone). Tune the inputs against your own chart review — swing
  lookback and mitigation mode in particular will change signal frequency
  a lot.
- All HTF/LTF FVG detection uses only fully closed bars (via `[1]`/`[2]`/`[3]`
  offsets and `barmerge.lookahead_off`), so it does not repaint.
- Sub-minute custom LTF timeframes require a TradingView plan with
  intrabar/seconds resolution.
