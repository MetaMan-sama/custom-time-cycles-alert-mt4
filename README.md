# Custom Time Cycles Alert — MQL4 Script

A MetaTrader 4 script that identifies **price proximity to time-cycle highs and lows** by scanning the rolling extreme high and low prices over a configurable bar-count cycle window, then firing alerts when the current close approaches either boundary within a price buffer — with a `LastCycleBar` deduplication guard that prevents redundant alerts from firing on consecutive bars within the same price zone.

---

## Overview

Time cycle analysis operates on the premise that price tends to form significant turning points at regular, repeating bar-count intervals. This script operationalizes that concept by defining a rolling N-bar window (`CycleLength`) as one complete cycle, extracting its absolute high and low via `iHigh()` / `iLow()` iteration, and treating those extremes as the cycle's top and bottom. When price closes within `AlertBuffer` of either extreme, the script fires an alert — giving the trader advance warning that price is testing the cycle boundary, a level where reversals or breakouts historically concentrate. The `LastCycleBar` deduplication mechanism ensures only one alert fires per new bar at each cycle level, eliminating alert flooding during sustained boundary tests.

---

## Features

- **Rolling cycle extreme extraction** — `CalculateCycleHighLow()` iterates `CycleLength` bars, updating `CycleHigh` and `CycleLow` via `iHigh()` and `iLow()` with `DBL_MAX` initialization for unbiased low detection
- **Price buffer proximity trigger** — `MathAbs(currentPrice − CycleHigh) <= AlertBuffer` and `MathAbs(currentPrice − CycleLow) <= AlertBuffer` detect approach within configurable price distance
- **`LastCycleBar` deduplication** — compares `iB ars(symbol, timeframe)` to the stored `LastCycleBar` value; fires and updates only when a new bar has formed since the last alert, preventing multi-alert flooding on the same candle
- **Global state persistence** — `CycleHigh`, `CycleLow`, and `LastCycleBar` global variables retain values across loop iterations for accurate cross-cycle comparisons
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)
- Alert message includes both the current price and the precise cycle level being approached for direct trading context

---

## How It Works

1. Every minute, `CalculateCycleHighLow()` validates `iBars() >= CycleLength`, initializes `CycleHigh = 0.0` and `CycleLow = DBL_MAX`, then iterates `CycleLength` bars updating both extremes via `iHigh()` and `iLow()`
2. `currentPrice = iClose(..., 0)` is fetched
3. Two proximity conditions are evaluated with deduplication:
   - `|currentPrice − CycleHigh| <= AlertBuffer` and `LastCycleBar != iBars()` → **Price Near Cycle Top** — `LastCycleBar` updated to current `iBars()`
   - `|currentPrice − CycleLow| <= AlertBuffer` and `LastCycleBar != iBars()` → **Price Near Cycle Bottom** — `LastCycleBar` updated
4. `AlertCycle()` dispatches messages via all enabled notification channels with cycle level and current price

---

## Input Parameters

| Parameter      | Type            | Default     | Description                                                               |
|----------------|-----------------|-------------|---------------------------------------------------------------------------|
| `TradeSymbol`  | string          | `EURUSD`    | Symbol for analysis                                                       |
| `Timeframe`    | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for cycle calculation                                           |
| `CycleLength`  | int             | `20`        | Number of bars defining one complete cycle window                         |
| `AlertBuffer`  | double          | `0.0005`    | Maximum price distance from cycle high/low to trigger a proximity alert   |
| `EnableAlerts` | bool            | `true`      | Fire an on-screen/sound alert                                             |
| `EnableEmail`  | bool            | `false`     | Send an email notification                                                |
| `EnablePush`   | bool            | `false`     | Send a mobile push notification                                           |

---

## Alert Message Format

```
Price Near Cycle Top detected on EURUSD (Timeframe: PERIOD_H1)
Cycle Level: 1.08640, Current Price: 1.08618
```

---

## Installation

1. Copy `Custom_Time_Cycles_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

> **Note:** `AlertBuffer` should be calibrated to the typical pip range of your instrument and timeframe. On a 5-digit EURUSD broker, `0.0005` equals 5 pips. Adjust for higher-volatility instruments or longer timeframes.

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
