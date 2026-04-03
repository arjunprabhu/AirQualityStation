# Air Quality Station — Requirements Document

## 1. Project Goal

Build a professional indoor air quality monitoring station using ESPHome with an **LVGL-based touchscreen UI**, real-time AQI calculation, multi-sensor integration, presence-aware display control, and full Home Assistant connectivity.

---

## 2. Hardware Configuration

| Setting | Value |
|---|---|
| MCU | Seeed XIAO ESP32-S3 |
| Framework | **esp-idf** |
| PSRAM | Octal, 80MHz |
| Display | ILI9341 320×240 with LVGL |
| Touchscreen | XPT2046 with calibration |
| I2C | GPIO5/GPIO6 @ 100kHz |
| SPI | GPIO7/GPIO8/GPIO9 @ 40MHz |
| UART | GPIO43 (TX) / GPIO44 (RX) @ 115200 baud |
| Backlight | GPIO4 (binary, presence-controlled) |

---

## 3. Page Structure (4 pages)

| Page | Purpose |
|---|---|
| **Page 1 — Air Quality** | AQI badge + VOC/NOx bars |
| **Page 2 — Particulates** | PM1.0/2.5/4.0/10 bars + particle size |
| **Page 3 — Home & Weather** | Indoor/outdoor temp + weather from HA |
| **Page 4 — Local Sensors** | SHT41 temp/humidity, VEML7700 lux, LD2412 distance |

### Status Bar (top_layer, all pages)
- AQI value, indoor temperature, WiFi signal icon (color-coded)

### Navigation
- Swipe left/right (30px threshold)
- Page indicator dots (white = active)
- Auto-scroll every 15s when presence detected (30s initial delay)
- Page wrap enabled

---

## 4. Sensors

### 4.1 Local Sensors

| Sensor | Platform | ID | Display |
|---|---|---|---|
| SPS30 PM1.0 | `sps30` | `local_PM_1_0` | Page 2 |
| SPS30 PM2.5 | `sps30` | `local_PM_2_5` | Page 1 AQI + Page 2 |
| SPS30 PM4.0 | `sps30` | `local_PM_4_0` | Page 2 |
| SPS30 PM10 | `sps30` | `local_PM_10_0` | Page 2 |
| SPS30 Size | `sps30` | `pm_size` | Page 2 |
| SGP4x VOC | `sgp4x` | `local_voc` | Page 1 |
| SGP4x NOx | `sgp4x` | `local_nox` | Page 1 |
| SHT41 Temp | `sht4x` | `sht_temp` | Page 4 |
| SHT41 Humid | `sht4x` | `sht_humid` | Page 4 |
| VEML7700 | `veml7700` | `local_lux` | Page 4 |
| LD2412 Presence | `ld2412` | `radar_presence` | Backlight |
| LD2412 Distance | `ld2412` | `radar_still_distance` | Page 4 |
| WiFi Signal | `wifi_signal` | `wifi_rssi` | Status bar |

### 4.2 AQI Calculation

US EPA breakpoints from PM2.5. On-device C++ lambda, exposed as `ha_exposed_aqi`.

| AQI | PM2.5 (µg/m³) | Color | Label |
|---|---|---|---|
| 0–50 | 0–12.0 | Green `0x00E000` | Good |
| 51–100 | 12.1–35.4 | Yellow `0xF0E000` | Moderate |
| 101–150 | 35.5–55.4 | Orange `0xFF6600` | Sensitive |
| 151–200 | 55.5–150.4 | Red `0xFF0000` | Unhealthy |
| 201–300 | 150.5–250.4 | Purple `0x8B00FF` | Very Bad |
| 301–500 | 250.5+ | Maroon `0x800000` | Hazardous |

### 4.3 HA Imports

| Entity | Display |
|---|---|
| `sensor.outdoor_temp_and_humidity_sensor_temperature` | Page 3 |
| `sensor.upstairs_thermostat_current_temperature` | Page 3 + status bar |
| `sensor.openweathermap_condition` | Page 3 icon + text |

### 4.4 SGP4x Compensation
Uses SHT41 `sht_temp` and `sht_humid` for humidity/temperature compensation.

---

## 5. UX Design

### Design Principles
- ISA-101 Level 1 overview (Page 1 AQI badge)
- EEMUA 201 dark cockpit (black bg, muted grays)
- Alarm colors reserved for AQI severity only
- Gestalt proximity (cards group related data)

### Typography
- `MONTSERRAT_28` — AQI, temperatures
- `MONTSERRAT_20`/`24` — sensor values
- `MONTSERRAT_14`/`16` — headers, weather
- `MONTSERRAT_12` — sub-labels, status

### Color Palette
| Role | Hex |
|---|---|
| Background | `0x000000` |
| Card | `0x1A1A1A` |
| Primary text | `0xFFFFFF` |
| Secondary text | `0xB0B0B0` |
| Muted | `0x666666`/`0x808080` |
| Card border | `0x333333` |
| Blue accent | `0x3498DB` |
| PM bar | `0x2E7D32` (70% opacity) |

### Icons
10 UI icons (20-24px) + 14 weather icons (28px), all MDI binary format.

---

## 6. Reliability

| Feature | Implementation |
|---|---|
| NaN handling | `--` for unavailable values |
| API timeout | `reboot_timeout: 0s` |
| Boot init | `refresh_all_widgets` script |
| Presence backlight | On/off via LD2412 |
| Radar restart | Daily 3:00 AM via SNTP cron |
| WiFi indicator | Color-coded icon |
| Fallback AP + captive portal | WiFi recovery |

---

## 7. Presence & Backlight

| Event | Action |
|---|---|
| Presence detected | Backlight on, start auto-scroll |
| Presence lost (2-min delay) | Backlight off, stop auto-scroll |
| Touch | Reset auto-scroll timer |
| Boot (no presence after 120s) | Backlight off |
