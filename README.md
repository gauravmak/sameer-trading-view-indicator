# Option Pulse Pro (rebuild) — TradingView / Pine Script v6

A rebuild of the public logic of the "OptionPulse Pro" indicator (Jadhav Trading) for personal use.

The original reads the **ATM call (CE) and put (PE) premiums against a "zone"** and confirms with open interest:

| State | CE | PE | OI reading |
|---|---|---|---|
| LONG SETUP | above zone | below zone | short covering / long buildup |
| SHORT SETUP | below zone | above zone | long unwinding / short buildup |
| PREMIUM DECAY (avoid) | below zone | below zone | no long buildup |

The original's zone formula is proprietary, so here the zone is **configurable** (VWAP band, opening range, or EMA ± ATR band) and can be tuned until it matches the paid indicator.

## Files

- `option_pulse_pro.pine` — the indicator.
- `tests/oi_probe.pine` — 20-line check: does TradingView expose open interest for an NSE option contract?

## Install

1. TradingView → open a **1-minute chart of the index** (`NSE:NIFTY` or `NSE:BANKNIFTY`).
2. Pine Editor → New → paste `option_pulse_pro.pine` → **Add to chart**.
3. The dashboard (top right) shows the two option symbols it picked, their premium, zone, position, OI and the final STATE.

## How it picks contracts

- **Strike**: ATM from the latest price, rounded to the strike step (NIFTY 50, BANKNIFTY 100). `Strike offset` shifts both legs OTM.
- **Expiry**: auto. NIFTY weekly on Tuesday, BANKNIFTY/FINNIFTY/MIDCPNIFTY monthly (last Tuesday), SENSEX weekly Thursday. Holidays shift expiry, so there is a **Manual expiry (YYMMDD)** override.
- **Symbol format**: TradingView's NSE format `NSE:NIFTY251028C26000` (root + yymmdd + C/P + strike). Any leg can be overridden with a manual symbol.

If the dashboard shows `n/a` for the premium, the built symbol is wrong. Search the contract in TradingView's symbol search, copy its exact name into *Manual CE/PE symbol*, and tell me the format so I can fix the auto builder.

## Open interest

Pine gets OI through a synthetic `<symbol>_OI` series. This is confirmed for NSE futures (`NSE:NIFTY1!_OI`) and **unconfirmed for individual option contracts**. Run `tests/oi_probe.pine` once:

- If "CE option OI" shows a number → set *Confirm with = Option OI* in the main indicator.
- If it says NOT AVAILABLE → keep the default *Futures OI*.

TradingView refreshes OI only every few minutes, so OI confirmation lags the price.

## Alerts

Four alert conditions: Long setup, Short setup, Premium decay, Any state change. The script also fires a dynamic `alert()` message with the symbols and OI reading on each state change (use "Any alert() function call" when creating the alert).

## Known gaps vs. the original

- Zone definition is a guess until compared side by side with the paid indicator.
- MCX Gold/Crude options: symbol format on TradingView not verified; use manual symbols.
- Needs real-time NSE data on TradingView (delayed data = 15-minute-late signals on a 1-minute chart).
