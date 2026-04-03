# Air Quality Station — Screen Design Reference

Design for a 320×240 ILI9341 landscape display using LVGL on ESP32-S3.

**Design principles:** ISA-101 information hierarchy, EEMUA 201 dark cockpit, Gestalt grouping, Fitts' law touch targets.

---

## Page 1 — Air Quality Overview

**Purpose:** "Is the air quality OK?" — answerable in 2 seconds via AQI badge color.

![Page 1 - Air Quality Overview](page1_air_quality_1773714270163.png)

### Widget Mapping

| Element | LVGL Widget | Data Source | Update Trigger |
|---|---|---|---|
| AQI number | `label` (MONTSERRAT_28) | Template sensor (calc from PM2.5) | `on_value` |
| AQI badge bg | `obj` container (68×68 circle) | Lambda color from AQI range | `on_value` |
| Quality text | `label` ("Good"/"Moderate"/...) | Lambda from AQI value | `on_value` |
| PM2.5 value | `label` (MONTSERRAT_12) | `local_PM_2_5` | `on_value` |
| VOC bar | `bar` (0–500 range) | `local_voc` | `on_value` |
| VOC value | `label` (MONTSERRAT_20) | `local_voc` | `on_value` |
| NOx bar | `bar` (0–500 range) | `local_nox` | `on_value` |
| NOx value | `label` (MONTSERRAT_20) | `local_nox` | `on_value` |
| Air filter icon | `image` (binary, mdi:air-filter) | Static | — |

---

## Page 2 — Particulate Detail

**Purpose:** Detailed PM breakdown — all 4 mass concentration channels with bar visualization.

![Page 2 - Particulate Detail](page2_particulates_1773714283192.png)

### Widget Mapping

| Element | LVGL Widget | Data Source | Update Trigger |
|---|---|---|---|
| PM1.0 bar | `bar` (0–50 range) | `local_PM_1_0` | `on_value` |
| PM1.0 value | `label` (MONTSERRAT_14) | `local_PM_1_0` | `on_value` |
| PM2.5 bar | `bar` (0–50 range) | `local_PM_2_5` | `on_value` |
| PM2.5 value | `label` (MONTSERRAT_14) | `local_PM_2_5` | `on_value` |
| PM4.0 bar | `bar` (0–50 range) | `local_PM_4_0` | `on_value` |
| PM4.0 value | `label` (MONTSERRAT_14) | `local_PM_4_0` | `on_value` |
| PM10 bar | `bar` (0–50 range) | `local_PM_10_0` | `on_value` |
| PM10 value | `label` (MONTSERRAT_14) | `local_PM_10_0` | `on_value` |
| Particle size | `label` (MONTSERRAT_12, centered) | `pm_size` | `on_value` |

### Layout
- Flex column with SPACE_EVENLY, 6px row padding
- Each PM row: 310×36px container with bar overlay + labels
- Bar indicator color: `0x2E7D32` (green) at 70% opacity

---

## Page 3 — Home & Weather

**Purpose:** Indoor/outdoor conditions from Home Assistant sensors.

![Page 3 - Home & Weather](page3_home_weather_1773714295191.png)

### Widget Mapping

| Element | LVGL Widget | Data Source | Update Trigger |
|---|---|---|---|
| Indoor temp | `label` (MONTSERRAT_28) | `ha_indoor_temp` | `on_value` |
| Outdoor temp | `label` (MONTSERRAT_28) | `ha_outdoor_temp` | `on_value` |
| Indoor icon | `image` (mdi:home-thermometer) | Static | — |
| Outdoor icon | `image` (mdi:thermometer) | Static | — |
| Weather text | `label` (MONTSERRAT_16) | `ha_weather_desc` | `on_value` |
| Weather icon | `image` (28×28) | Mapped from condition text | `on_value` lambda |

### Weather Icon Mapping (14 conditions)
| Condition | Icon | Color |
|---|---|---|
| `sunny` | `icon_w_sunny` | Yellow `0xF0E000` |
| `clear-night` | `icon_w_night` | Gray `0xB0B0B0` |
| `cloudy` | `icon_w_cloudy` | Gray `0x909090` |
| `partlycloudy` | `icon_w_partly_cloudy` | Yellow `0xF0E000` |
| `rainy` | `icon_w_rainy` | Blue `0x3498DB` |
| `pouring` | `icon_w_pouring` | Blue `0x3498DB` |
| `snowy` | `icon_w_snowy` | White `0xFFFFFF` |
| `snowy-rainy` | `icon_w_snowy_rainy` | Light Blue `0xB0B0FF` |
| `fog` | `icon_w_fog` | Gray `0x909090` |
| `windy`/`windy-variant` | `icon_w_windy` | Gray `0xB0B0B0` |
| `lightning` | `icon_w_lightning` | Yellow `0xF0E000` |
| `lightning-rainy` | `icon_w_lightning_rainy` | Yellow `0xF0E000` |
| `hail` | `icon_w_hail` | Light Blue `0xB0B0FF` |
| `exceptional` | `icon_w_exceptional` | Red `0xFF3000` |

---

## Page 4 — Local Sensors

**Purpose:** On-board sensor readings — temperature, humidity, ambient light, radar distance.

### Widget Mapping

| Element | LVGL Widget | Data Source | Update Trigger |
|---|---|---|---|
| Temperature | `label` (MONTSERRAT_24) | `sht_temp` (converted to °F) | `on_value` |
| Humidity | `label` (MONTSERRAT_24) | `sht_humid` | `on_value` |
| Ambient light | `label` (MONTSERRAT_24) | `local_lux` | `on_value` |
| Still distance | `label` (MONTSERRAT_24) | `radar_still_distance` (converted to ft) | `on_value` |
| Temp icon | `image` (mdi:home-thermometer) | Static, red `0xFF3000` | — |
| Humid icon | `image` (mdi:water-percent) | Static, blue `0x3498DB` | — |
| Lux icon | `image` (mdi:brightness-6) | Static, yellow `0xF0E000` | — |
| Radar icon | `image` (mdi:radar) | Static, gray `0x909090` | — |

### Layout
- 2×2 grid of cards (154×82px each)
- Top row: Temperature (left), Humidity (right)
- Bottom row: Ambient Light (left), Still Distance (right)

---

## Status Bar (top_layer, all pages)

| Element | Widget | Position | Data Source |
|---|---|---|---|
| AQI value | `label` (MONTSERRAT_12) | TOP_RIGHT, x:-80, y:6 | PM2.5 AQI calc |
| Indoor temp | `label` (MONTSERRAT_12) | TOP_RIGHT, x:-30, y:6 | `ha_indoor_temp` |
| WiFi icon | `image` (16×16) | TOP_RIGHT, x:-4, y:5 | `wifi_rssi` (color-coded) |

---

## Navigation Design

- **Swipe left/right** on touchscreen to navigate (30px threshold)
- **4 page indicator dots** on each page (bottom, y:224)
  - Active page = white (`0xFFFFFF`), inactive = dark gray (`0x404040`)
  - Dot size: 10×10px, radius 5, spacing 16px apart
  - Starting x: 131 (centered for 4 dots across 320px)
- **Page wrap** enabled (Page 4 → swipe right → Page 1)
- **Auto-scroll** — pages cycle every 15s when presence detected

---

## Swipe Detection (top_layer overlay)

- Full-screen transparent `obj` (320×240) captures touch events
- `on_press`: records start position via `lv_indev_get_point()`
- `on_release`: calculates delta, determines direction
  - Horizontal dominant: dx < -30 → next page, dx > 30 → previous page
  - Vertical dominant: -dy < -30 → next, -dy > 30 → previous
- Any touch resets the auto-scroll timer
