# Option Pulse Pro (rebuild) — TradingView / Pine Script v6

A rebuild of the public logic of the "OptionPulse Pro" indicator (Jadhav Trading) for personal use.

The original reads the **ATM call (CE) and put (PE) premiums against a "zone"** and confirms with open interest:

| State | CE | PE | OI reading |
|---|---|---|---|
| LONG SETUP | above zone | below zone | short covering / long buildup |
| SHORT SETUP | below zone | above zone | long unwinding / short buildup |
| PREMIUM DECAY (avoid) | below zone | below zone | no long buildup |

The original's formulas are not public. From side-by-side screenshots the green "zone" line behaves like an anchored VWAP, the blue/pink "Main Trend" like a Supertrend, and the cloud like a fast EMA pair, so that is what is implemented, with every length exposed as an input.

## Files

- `option_pulse_pro.pine` — **overlay replica**. Add it to the option contract's own chart (e.g. `NSE:NIFTY261117C24900`, 1-minute), exactly like the original. Components, matched to the paid indicator's behaviour: Institutional Breakout Zone = anchored VWAP (green, anchored to the start of the contract's data), Main Trend - ITM = smooth Supertrend trailing line (blue below price, pink above; ATR measured over 5-bar spans, period 10, factor 2), Running Trend - Pullback Zone = EMA 9/21 cloud, optional Today's High/Low, background colour. It auto-detects the opposite leg (same strike and expiry, CE↔PE), follows Heikin Ashi if the chart uses it, and shows the combined CE/PE state with futures OI in a small table.
- `option_pulse_dashboard.pine` — **one-pane dashboard**. Add it to the index chart (`NSE:NIFTY`) or to any option chart of it; it picks the ATM CE and PE for the current expiry and shows both premiums, their zones, OI and the state.
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

## Known gaps vs. the original

- Supertrend settings (5-bar span, ATR 10, factor 2) and EMA 9/21 are fitted to screenshots of the paid indicator; the anchored VWAP matched to within 0.1. Compare the overlay with the paid one on the same chart and tune in Settings until the lines sit on top of each other.
- The anchored VWAP depends on how much history TradingView loads for the contract; the paid line likely has the same dependency.
- MCX Gold/Crude options: symbol format on TradingView not verified; use manual symbols.
- Needs real-time NSE data on TradingView (delayed data = 15-minute-late signals on a 1-minute chart).
