# MAX SMC Indicator — Complete Unified Documentation

## 1. Core Architecture Overview
The MAX SMC indicator is an automated, institutional-grade quantitative mapping tool built for TradingView in Pine Script. It acts as a full-suite algorithm that strictly models smart money concepts (SMC) via 100% mechanical, rigid mathematical rules, eliminating all human bias from market structure reading.

The engine has been unified from multiple detached modules into a single `MAX_SMC_Unified.pine` script to prevent redundant loop scans and prevent maximum-token limitations inside the Pine Script editor.

---

## 2. The Twin Structure Engines
The entire logic hinges on two separate, asynchronously updating layers of structure. **They never mix variables**, ensuring short-term choppiness never invalidates the macro trend.

### A. External Structure (Pro Trend)
- **Role:** Maps the overarching direction of the market (HTF equivalent flow).
- **Trigger:** Relies on the `External Swing Confirm length` setting (default: 3). To form a Swing High (SH), there must be a sequence of 3 strictly consecutive bearish candles pulling away from the high. Conversely, a Swing Low (SL) requires 3 strictly consecutive bullish candles pulling away from the low.
- **BOS & CHoCH:** 
  - A valid external Break of Structure (BOS) ONLY occurs when price closes beyond the previous confirmed SH (in a bullish trend) or SL (in a bearish trend). 
  - A Change of Character (CHoCH) occurs when price breaks structures *against* the confirmed bias. 
  - The primary engine tracks this securely via the `trend` state (`1` for Bullish, `-1` for Bearish).

### B. Internal Structure (Current Trend)
- **Role:** Maps the local, high-frequency price legs inside the overarching External price legs.
- **Trigger:** Relies on fractional Displacement (ATR multipliers) rather than strictly rigid candle requirements. The `Internal Swing Confirm length` defaults to 2, tracking localized `iSH` and `iSL` formations.
- **iBOS & iCHoCH:** These dictate the `int_trend` state, marking immediate shifts in momentum regardless of macro positioning.

---

## 3. The Trend Classification & Multi-Timeframe (MTF) Panel
By combining the two structural layers above, the Top-Right Status Dashboard is generated.

- **PRO TREND:** Direct 1-to-1 reflection of the External Structure (`trend`).
- **CURRENT TREND:** Direct 1-to-1 reflection of the Internal Structure (`int_trend`).
- **ALIGNMENT:** 
  - `IN SYNC` (Bullish + Bullish OR Bearish + Bearish): You have full trend concurrence.
  - `COUNTER`: You are trading internal retracements against the Pro Trend.

### MTF Dynamics
Using `request.security()`, the dashboard actively clones the Pro Trend logic and runs it silently on three additional timeframes of your choosing (e.g., 4H, 1H, 15M). The formatting is beautifully compressed into 3 columns side-by-side, translating numerical timeframe values (`"240"`, `"60"`) seamlessly into human-readable text (`"4H"`, `"1H"`).

---

## 4. Supply & Demand Operations (POI Engine)
Points of Interest (POIs) derive directly from the macro `trend` bias to ensure the algorithm isn't plotting counter-trend traps.

1. **Order Blocks (OB):** Identifies pre-impulse consolidation boundaries right before an expansion candle sweeps away from the structure.
2. **Fair Value Gaps (FVG):** Maps 3-candle imbalances where the wicks of candle 1 and candle 3 fail to overlap.
3. **Inversion FVGs (IFVG):**
   - **Crucial Feature:** Standard FVGs that are violated via a solid candle close are automatically removed from the active zones list, restyled as a secondary color, and converted into IFVGs (since failed FVGs historically act as subsequent support/resistance).

*Note: Disabling visual outputs for POIs simply masks them on the chart; the backend math natively runs to ensure state changes (like FVG -> IFVG) continue uninterrupted mathematically.*

---

## 5. Liquidity Systems
Instead of subjectively spotting trendlines or double tops, the Liquidity Engine algorithmically catalogs all structural highs (BSL) and lows (SSL).

- **Tolerance Mechanics:** Utilizing an `ATR * 0.15` band tolerance, it groups closely clustered highs/lows together. If 2 or more pivot extrema touch this same price band, it confirms the zone as massive Institutional Liquidity.
- **Sweep Detection:** When price blasts through this level via wick/close, the zone is forcibly tagged as "Swept" (turning dotted/red) and then removed from the array to prevent chart clutter.

---

## 6. Real-World Troubleshooting & Quirks
- **Why did a BOS fail to map on a big dump?** 
If the external setting is set to `3`, a tiny 2-candle pullback during the dump is mathematically ignored. Price will crash right through that valley, but because it was never a confirmed swing, it won't map a BOS. To fix this, simply drop the `External Swing Confirm length` to `2`.
- **Lagging Trends on Fast Crashes:** The Pro/Current trend used to wait for a confirmed "Higher High" and "Higher Low" to switch. This was mathematically pure but slow on massive dumps. It was successfully updated so the panel now reads directly off the core `bos/choch` logic block, guaranteeing 0-bar lag on structural shifts.
