# 🌬️ Smart Air Quality Station & Dashboard Enclosure

Welcome to the official 3D printable enclosure for the **Air Quality Station**! This is a sleek, desktop-friendly housing designed to neatly organize a powerful environmental monitor. It precisely holds an ILI9341 touchscreen, a Seeed XIAO ESP32-S3 microcontroller, and an array of professional-grade air sensors.

Whether you want an independent desk monitor or a gorgeous smart home dashboard for Home Assistant, this enclosure gives your DIY electronics a premium, finished look.

---

## 📐 Enclosure Design & Features

I designed this housing from scratch to seamlessly combine aesthetics and engineering needs:

*   **Precision Fit:** Tailored exactly for the standard 2.8" ILI9341 LCD Touchscreen module, hiding the ugly PCBs while highlighting the display.
*   **Optimal Airflow:** Features strategically placed ventilation ports ensuring the internal Sensirion sensors get fresh room air without stagnation (vital for accurate PM2.5, VOC, and temperature readings).
*   **Radar Window:** Includes a specialized cutout/thin-wall region tailored for the HLK-LD2412 24GHz mmWave radar module to allow flawless presence detection.
*   **Internal Hub Mounting:** Includes mounting clearances for a standard 17×10 Gikfun solderable mini breadboard where all the wiring converges.
*   **Fully Parametric (Fusion 360):** I've included the original `.f3d` source file! Feel free to modify the clearances, tweak the USB routing, or adapt it to sensors you already have on hand.

---

## 🖨️ Printing Recommendations

*   **Material:** **PETG** is highly recommended—especially if the unit will sit near a sunny window—as the internal electronics can get slightly warm. PLA works perfectly fine too for shaded indoor use!
*   **Layer Height:** 0.20mm provides an excellent balance of detail and speed.
*   **Orientation & Supports:** Supports are **required**. I highly recommend printing the models in the orientation that places the visible outside "face" on your build plate (if you have a textured PEI sheet, it leaves a beautiful pattern on the bezel!).
*   **Provided Profiles:** The included Bambu Studio `.3mf` file is pre-configured with optimal support placements and orientations.

---

## 🛠️ The Build (Hardware Details)

If you're eager to build the full smart station, here is a quick look at everything packed inside this box:

### What it does:
This station runs **ESPHome + LVGL** to provide an animated, 4-page touchscreen UI. It actively measures EPA AQI, Particulate Matter (PM1.0 up to PM10), VOCs, NOx, Temperature, Humidity, and Ambient Light. The mmWave radar detects when you sit at your desk to automatically turn the display on!

### Core Bill of Materials:
*   **Microcontroller:** Seeed XIAO ESP32-S3 (with PSRAM & WiFi)
*   **Display:** 2.8" ILI9341 TFT with XPT2046 touch controller
*   **Sensors:**
    *   Sensirion SPS30 (Laser Particulate Sensor)
    *   Sensirion SGP41 (VOC + NOx Gas)
    *   Sensirion SHT41 (Temperature & Humidity)
    *   VEML7700 (Ambient Light)
    *   HLK-LD2412 (24GHz mmWave radar)
*   **Internals:** Gikfun 17×10 Protoboard & Dupont wires

---

## 💻 Software & ESPHome Configuration

This model is just the physical shell! 

For the complete project files—including **High-Resolution Wiring Diagrams**, **Pin Mappings**, and the full **ESPHome YAML Configuration** needed to make the display and sensors work together—please visit the project’s main repository on GitHub:

👉 **[Air Quality Station - GitHub Repository](https://github.com/arjunprabhu/AirQualityStation)** 

*Happy Printing & Making!*
