# Air Quality Station — Device & Architecture Analysis

## Hardware Summary

| Component | Detail |
|---|---|
| **MCU** | Seeed XIAO ESP32-S3 (esp-idf framework) |
| **PSRAM** | Octal mode, 80MHz (8MB) |
| **Display** | ILI9341 (SPI) — 320×240 @ 90° rotation = landscape |
| **Touchscreen** | XPT2046 (SPI) — calibrated, swipe navigation |
| **PM Sensor** | Sensirion SPS30 — I2C (`0x69`) |
| **Gas Sensor** | Sensirion SGP4x — VOC + NOx, I2C (`0x59`) |
| **Temp/Humidity** | Sensirion SHT41 — I2C (`0x44`) |
| **Light Sensor** | VEML7700 — I2C (`0x10`) |
| **Radar** | HLK-LD2412 — 24GHz mmWave, UART (115200 baud) |
| **I2C Bus** | GPIO5 (SDA) / GPIO6 (SCL) @ 100kHz |
| **SPI Bus** | GPIO7 (CLK), GPIO8 (MISO), GPIO9 (MOSI) @ 40MHz |
| **UART** | GPIO43 (TX) / GPIO44 (RX) @ 115200 baud |
| **Backlight** | GPIO4 (binary output, presence-controlled) |

## Architecture

### Display Engine
- **LVGL** with 16-bit color depth (RGB565)
- 4-page UI with swipe navigation and animated transitions
- Event-driven updates via `on_value` sensor callbacks
- No display lambda — all rendering through LVGL widgets

### Sensor Pipeline
```
SPS30 ──┐
SGP4x ──┤── I2C Bus (100kHz) ──── ESP32-S3 ──── LVGL Display
SHT41 ──┤                              │
VEML7700┘                              ├──── Home Assistant (API)
                                       │
LD2412 ── UART (115200) ───────────────┘
```

### Data Flow
1. Sensors publish readings at `refresh_interval` (default 5s)
2. Each `on_value` callback updates corresponding LVGL widget
3. PM2.5 also triggers AQI calculation lambda
4. AQI value updates badge color, quality text, and status bar
5. HA-imported sensors (weather, temps) update via API subscription
6. WiFi RSSI updates color-code the status bar icon

### Presence System
- LD2412 radar detects human presence (2-min delayed_off filter)
- Presence on → backlight on + auto-scroll starts
- Presence off → backlight off + auto-scroll stops
- Touch → resets auto-scroll timer
- Boot → 120s grace period before turning off backlight

### Scheduled Tasks
- LD2412 restart daily at 3:00 AM via SNTP cron

### Reliability Features
- NaN-safe rendering on all sensor widgets
- `reboot_timeout: 0s` on API (survives HA disconnects)
- Boot-time `refresh_all_widgets` script re-publishes all sensor states
- Fallback AP + captive portal for WiFi recovery
- WiFi signal strength monitoring with visual indicator

## Entities Exported to Home Assistant

| Entity | Type | Description |
|---|---|---|
| PM 1.0/2.5/4.0/10 | sensor | Mass concentrations (µg/m³) |
| Typical Particle Size | sensor | Average particle diameter (µm) |
| VOC Index | sensor | SGP4x VOC index |
| NOx Index | sensor | SGP4x NOx index |
| Calculated AQI | sensor | US EPA AQI from PM2.5 |
| SHT41 Temperature | sensor | Local temperature (°C) |
| SHT41 Humidity | sensor | Local humidity (%) |
| Ambient Light | sensor | VEML7700 illuminance (lux) |
| Human Presence | binary_sensor | LD2412 presence (2-min delay) |
| Moving/Still Distance | sensor | LD2412 detection distances |
| Moving/Still Energy | sensor | LD2412 energy levels |
| Display Backlight | light | Binary on/off control |
| WiFi Signal | sensor | RSSI (dBm) + percentage |
| ESP Uptime | sensor | Device uptime (seconds) |
| LD2412 firmware/MAC | text_sensor | Diagnostics |

## HA Imports

| Entity | Usage |
|---|---|
| `sensor.outdoor_temp_and_humidity_sensor_temperature` | Page 3 outdoor temp |
| `sensor.upstairs_thermostat_current_temperature` | Page 3 indoor temp + status bar |
| `sensor.openweathermap_condition` | Page 3 weather icon + text |
