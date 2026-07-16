# 🏡 Home Environment — Grafana dashboard

A Grafana dashboard for the environment sensors Home Assistant is writing into InfluxDB2:
temperature, humidity, PM2.5 / PM10, CO₂ and barometric pressure — with **conditional
colour thresholds** and **cross-metric comparison** panels.

![layout](layout — 6 coloured stat tiles on top, trend charts in the middle, comparison + snapshot table below)

---

## What's in it

| Panel | Type | What it shows |
|-------|------|---------------|
| Now — at a glance | 6 × stat tiles | Latest reading per sensor, tile background coloured by health band |
| Temperature / Humidity — by room | time series | One line per room, dashed guide lines at the "too warm / too humid" bands |
| Particulate matter / CO₂ | time series | PM2.5 + PM10 together, CO₂ with ventilation bands; area is colour-graded by level |
| Temperature vs Humidity | dual-axis time series | Temp (left axis) overlaid on humidity (right axis) to see how they move together |
| PM2.5 by room — right now | bar gauge | Compare current particulate levels across rooms side by side |
| All environment sensors | table | Current snapshot of every env sensor, filterable/sortable |

### Colour bands (conditional colouring)

- **Temperature (°C):** blue < 16 · teal 16–18 · 🟢 green 18–24 (comfortable) · 🟠 orange 24–27 · 🔴 red > 27
- **Humidity (%):** 🔴 red < 30 (dry) · 🟠 orange 30–40 · 🟢 green 40–60 (ideal) · 🟠 orange 60–70 · 🔴 red > 70 (damp)
- **PM2.5 (µg/m³, EPA AQI):** 🟢 0–12 good · 🟡 12–35 moderate · 🟠 35–55 unhealthy-sensitive · 🔴 55–150 unhealthy · 🟣 > 150 hazardous
- **PM10 (µg/m³):** 🟢 0–54 · 🟡 54–154 · 🟠 154–254 · 🔴 > 254
- **CO₂ (ppm):** 🟢 < 800 fresh · 🟡 800–1000 · 🟠 1000–1400 stuffy · 🔴 > 1400 ventilate

---

## Import

1. In Grafana: **Dashboards → New → Import**.
2. Upload `HA-Environment-Dashboard.json` (or paste its contents).
3. When prompted, pick your **InfluxDB2 (Flux)** data source.
4. Open the dashboard. At the top you'll see selector dropdowns — set them once (see below).

> Requires an InfluxDB2 data source configured in Grafana with **Flux** as the query language
> (Configuration → Data sources → InfluxDB → Query language: *Flux*, with your Org + API token).

---

## Configure it (top-of-dashboard dropdowns)

The dashboard is driven by template variables so it adapts to *your* sensor names without editing queries:

- **InfluxDB source** — which data source to query.
- **Bucket** — your InfluxDB2 bucket name. Defaults to `homeassistant`; change it to match yours.
- **Temp unit** — `°C` or `°F`. (This is also the InfluxDB *measurement* name HA uses.)
- **Temperature / Humidity / PM2.5 / PM10 / CO₂ / Pressure sensors** — multi-select lists,
  auto-populated from your data. Leave on **All**, or pick specific rooms.

---

## How the queries work (and how to tweak)

Home Assistant's InfluxDB integration uses this default schema:

- **Measurement name = the sensor's unit** → temperature is in a measurement called `°C`,
  humidity in `%`, PM2.5/PM10 in `µg/m³`, CO₂ in `ppm`, pressure in `hPa`.
- **The numeric value is the field `value`.**
- **`entity_id` and `domain` are tags** (indexed).

Because several sensor types share a unit (humidity **and** battery are both `%`; PM2.5 **and**
PM10 are both `µg/m³`), the queries also filter by `entity_id`. The sensor-picker variables use:

| Metric | Match rule |
|--------|-----------|
| Humidity | `_measurement == "%"` **and** `entity_id =~ /humid/` |
| PM2.5 | `_measurement == "µg/m³"` **and** `entity_id =~ /pm2/` |
| PM10 | `_measurement == "µg/m³"` **and** `entity_id =~ /pm10/` |
| CO₂ | `_measurement == "ppm"` **and** `entity_id =~ /co2/` |

If your entities are named differently (e.g. a PM2.5 sensor whose id doesn't contain `pm2`),
edit that variable: **Dashboard settings → Variables → (metric) sensors → Query**, and adjust the
`entity_id =~ /.../` regex to match your naming. Everything downstream picks it up automatically.

### Troubleshooting empty panels

- **A metric is empty** → its sensor-picker variable found nothing. Check the regex above against
  your real `entity_id`s (Data Explorer in InfluxDB, or HA → Developer Tools → States).
- **Wrong measurement name** → confirm what HA actually wrote. In InfluxDB Data Explorer, list
  measurements in your bucket; if you see `°F` instead of `°C`, switch the **Temp unit** dropdown.
- **Series labels** → panels show each sensor's HA **friendly name**. This build expects the HA
  `influxdb:` config to list `friendly_name` under `tags_attributes` (so it's stored as a tag).
  If a point has no friendly-name tag, the label falls back to `entity_id`. The *sensor-picker
  dropdowns* still list `entity_id`s — intentional, as entity ids are stable to filter on.

---

## Sources / references

- [Home Assistant — InfluxDB integration](https://www.home-assistant.io/integrations/influxdb/) (schema: `measurement_attr` defaults to `unit_of_measurement`; field defaults to `value`)
- [HA community — Grafana temperature with InfluxDB v2 + Flux](https://community.home-assistant.io/t/grafana-temperature-data-with-influxdb-v2-and-flux/290268)
- [InfluxData — How to integrate Grafana with Home Assistant](https://www.influxdata.com/blog/how-integrate-gafana-home-assistant/)
- PM2.5 / PM10 bands: US EPA AQI breakpoints · CO₂ bands: common indoor-air ventilation guidance
