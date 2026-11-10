# Option Pulse Pro (rebuild) — TradingView / Pine Script v6

A rebuild of the public logic of the "OptionPulse Pro" indicator (Jadhav Trading) for personal use.

The original reads the **ATM call (CE) and put (PE) premiums against a "zone"** and confirms with open interest:

| State | CE | PE | OI reading |
|---|---|---|---|
| LONG SETUP | above zone | below zone | short covering / long buildup |
| SHORT SETUP | below zone | above zone | long unwinding / short buildup |
| PREMIUM DECAY (avoid) | below zone | below zone | no long buildup |

The original's formulas are not public. The green zone line is an anchored VWAP (matched to 0.05). The blue/pink Main Trend was fitted against the paid indicator's exported values: a volume-weighted centre (elastic VWMA) ± 2 × ATR(14) band. Fitted jointly on CE and PE exports with volume: mean error about 2 points (under 1% of premium), flips on the same minutes. The Running Trend is a fast EMA running line with the "pullback wedge" filled between it and the Main Trend line, as in the original; a second EMA with a small cloud is available as an optional extra. Every length is an input.

## Files

- `option_pulse_pro.pine` — **overlay replica**. Add it to the option contract's own chart (e.g. `NSE:NIFTY261117C24900`, 1-minute), exactly like the original. Components, matched to the paid indicator's behaviour: Institutional Breakout Zone = anchored VWAP (green, anchored to the first bar of the contract's loaded data, which matched the paid line to within 0.1), Main Trend - ITM = volume-weighted band: elastic VWMA(62) of hlc3 ± 2 × ATR(14), lower band shown while bullish (blue), upper while bearish (pink), flipping when close crosses it, Running Trend - Pullback Zone = EMA 9/21 cloud, optional Today's High/Low, background colour. It auto-detects the opposite leg (same strike and expiry, CE↔PE), follows Heikin Ashi if the chart uses it, and shows the combined CE/PE state with futures OI in a small table.
- `option_pulse_dashboard.pine` — **one-pane dashboard**. Add it to the index chart (`NSE:NIFTY`) or to any option chart of it; it picks the ATM CE and PE for the current expiry and shows both premiums, their zones, OI and the state.
- `option_pulse_strategy.pine` — **backtest strategy** with the same engine, fixed risk:reward (default 1:3), stop at the Main Trend line, optional flip exit and intraday square-off. Long-only on option charts (buys the option when its own trend turns and the CE/PE state agrees); long and short on futures/stocks/commodities (long = "buy CE", short = "buy PE"). This file uses normal indentation, so paste it from a laptop.
- `tests/oi_probe.pine` — tiny check: does TradingView expose open interest for an NSE option contract? (The dashboard's "Option OI" row answers the same question.)

All scripts have **no indented lines** on purpose: Android browsers re-indent pasted lines, which breaks normal Pine code.

## Install

1. TradingView → Pine Editor → New indicator → select all, delete, paste the raw file → **Add to chart**.
2. Overlay: open the ATM CE chart and the ATM PE chart side by side (1-minute), add the overlay to both. The table on each shows the other leg too.
3. Dashboard: open `NSE:NIFTY` (1-minute) and add it. The table shows the two option symbols it picked, their premium, zone, position, OI and the final STATE.

## How it picks contracts

- **Strike**: ATM from the latest price, rounded to the strike step (NIFTY 50, BANKNIFTY 100). `Strike offset` shifts both legs OTM.
- **Expiry**: auto. NIFTY weekly on Tuesday, BANKNIFTY/FINNIFTY/MIDCPNIFTY monthly (last Tuesday), SENSEX weekly Thursday. If the computed date has no contract (exchange holiday), it falls back to one day earlier. There is also a **Manual expiry (YYMMDD)** override.
- **On an option chart**: root, expiry and strike are read from the chart symbol itself.
- **Symbol format**: TradingView's NSE format `NSE:NIFTY251028C26000` (root + yymmdd + C/P + strike). Any leg can be overridden with a manual symbol.

If the dashboard shows `n/a` for the premium, the built symbol is wrong. Search the contract in TradingView's symbol search, copy its exact name into *Manual CE/PE symbol*, and tell me the format so I can fix the auto builder.

## Open interest

Pine gets OI through a synthetic `<symbol>_OI` series. Tested 9 Nov 2026: available for NSE futures (`NSE:NIFTY1!_OI`), **not available for individual NSE option contracts**. So the setups are confirmed with futures OI (long buildup / short covering = bullish, short buildup / long unwinding = bearish). TradingView refreshes OI only every few minutes, so it lags the price.

## Alerts

Both scripts expose alert conditions (Long setup, Short setup, Premium decay, Any state change; the overlay also has Main Trend turned bullish/bearish). Create them from the chart's Alert dialog by choosing the indicator as the condition.

## Backtesting

1. Pine Editor → paste `option_pulse_strategy.pine` → Add to chart → open the **Strategy Tester** tab at the bottom.
2. On an option chart the test covers that contract's life only (a few weeks): there is no multi-year option history on TradingView.
3. For a multi-year test use **NSE:NIFTY1!** (continuous futures) on 1-minute, set the From date, and turn on **Deep Backtesting** in the Strategy Tester (Premium plan). Long trades stand in for "buy CE", short trades for "buy PE".
4. Defaults: 1 lot (75), R:R 1:3, entry on the Main Trend flip, exit on stop / target / opposite flip, no entries in the first 15 minutes, square-off 15:20. Account 20 lakh with 10% margin so a futures lot fits. Use regular candles, not Heikin Ashi, or the results are unrealistic.

## Backtest results (NSE:NIFTY1!, regular candles, Jan 2023 – Nov 2026, Deep Backtesting, 1 lot, Rs 40/order + 1 tick slippage)

| Run | Timeframe | Entry | Exit | R:R | Trades | Win rate | Profit factor | Net |
|---|---|---|---|---|---|---|---|---|
| 1 | 1-min | Main Trend flip | stop / target / flip | 1:3 | 2,988 | 30% | 0.99 | −1.1 lakh |
| B | 5-min | Pullback re-entry | stop / target / flip | 1:2 | 2,819 | 39% | 0.99 | −1.5 lakh |
| A | 5-min | Main Trend flip | stop / target / flip | 1:2 | 1,216 | 38% | 0.93 | −4.2 lakh |
| C | 5-min | Pullback re-entry | stop / target | 1:2 | 2,554 | 41% | 1.00 | +0.06 lakh |

Max drawdown 25–30% in every run; buy and hold made +7.5 lakh over the same period. A Heikin Ashi chart showed +13.7 lakh for run 1, which is an artefact of HA's untradeable prices.

**Conclusion:** the mechanical signals (Main Trend flip or pullback with the CE/PE state) have no edge on Nifty after costs. Use the overlay as a visual aid with manual decisions; do not connect it to a broker. Further parameter tuning would only overfit.

## Other markets (crude, gold, silver, stocks, futures)

The overlay's three drawings need only price and volume, so it works on any symbol and timeframe. On an option contract it also pairs the opposite leg and fills the CE/PE table; on anything else the table shows only this chart and the state reads ABOVE / BELOW / INSIDE ZONE. The VWAP anchor is automatic: first loaded bar on option contracts, session on everything else. Symbols without volume (e.g. a spot index) fall back to a plain EMA centre for the Main Trend. On a futures chart (`MCX:CRUDEOIL1!`) the OI row reads that contract's own OI. Lengths were fitted on Nifty options; other markets may want different lengths.

## Known gaps vs. the original

- Main Trend parameters (EVWMA 62, ATR 14, factor 2.0) are fitted on 9 Nov CE and PE data with volume; the factor came out at exactly 2.000 on both legs. The cloud's EMA 9/21 is by eye, since the paid cloud lines are not exportable. Compare the overlay with the paid one on the same chart and tune in Settings until the lines sit on top of each other.
- The VWAP anchors to the first loaded bar. For a weekly option the loaded 1-minute history covers the contract's whole life on paid plans, so this is stable; on the free plan with less history it will differ.
- MCX Gold/Crude options: symbol format on TradingView not verified; use manual symbols.
- Needs real-time NSE data on TradingView (delayed data = 15-minute-late signals on a 1-minute chart).
