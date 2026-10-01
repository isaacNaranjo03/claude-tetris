---
name: weather
description: Look up the current weather for a city, always reported in metric units (Celsius, wind speed in m/s). Use when the user asks for the weather, temperature, or forecast for a named place.
---

# Weather

Report current weather conditions for a city, always in metric units, regardless of what units the source uses.

## Input

`args` is the city name (optionally with country/region, e.g. `Quito`, `Quito, Ecuador`, `Portland, OR`). If `args` is empty, ask the user which city they mean — don't guess or assume a "local" location.

## Steps

1. **Search** for the city's current weather, e.g. `WebSearch({query: "<city> weather now current temperature"})`.
2. **Extract** from the results: temperature (and "feels like" if available), conditions (e.g. partly cloudy, rain), humidity, wind speed, and chance of precipitation if reported.
3. **Convert every unit to metric** before presenting anything — never pass through a source's imperial number unconverted:
   - Temperature: if reported in °F, convert with `C = (F - 32) * 5/9`. Round to the nearest whole degree.
   - Wind speed: convert to m/s.
     - from mph: `m/s = mph * 0.44704`
     - from km/h: `m/s = km/h / 3.6`
     - from knots: `m/s = knots * 0.514444`
     Round to one decimal place.
   - If a source already reports °C or m/s, use it as-is (no double conversion).
4. **Present** a compact summary, metric only (don't show the original imperial figures alongside):

   ```
   **Current weather in <City>:**
   - 🌡️ Temperature: <N>°C (feels like <N>°C) — high <N>°C / low <N>°C
   - ☁️ Conditions: <text>
   - 💧 Humidity: <N>%
   - 🌧️ Rain chance: <N>%
   - 💨 Wind: <N> m/s <direction if known>
   ```

   Omit any line the sources didn't provide rather than guessing a value.

5. **Cite sources** as markdown links at the end, per `WebSearch`'s own citation requirement.

## Notes

- Always double-check the arithmetic on unit conversions — don't eyeball it.
- If multiple sources disagree, prefer the most specific/recent one and mention the discrepancy only if it's large (>3°C or >2 m/s).
