# AirHealth

A small consumer device that monitors indoor air quality and CO2, and shows the result at a glance with a single RGB light: green for good, yellow for caution, red for poor. It is aimed at home offices, bedrooms and living rooms.

The device connects to home WiFi and logs readings to a cloud backend every 60 seconds. The light works on its own, so it keeps indicating air quality when WiFi is down. There is no screen, no buzzer, and no app needed to read it.

## Status

This project is in the planning stage. The repository currently holds the project plan; firmware, PCB and enclosure files will be added as each stage is built.

## What it measures

| Measurement | Green | Yellow | Red |
|---|---|---|---|
| CO2 (ppm) | below 800 | 800 to 1,200 | above 1,200 |
| PM2.5 (µg/m³) | 0 to 12 | 12.1 to 35.4 | above 35.4 |
| VOC index | 0 to 100 | 101 to 200 | above 200 |

The light shows the worst status across all three measurements. Temperature and humidity are also read, as context for sensor accuracy.

## Hardware

| Part | Choice |
|---|---|
| Microcontroller | ESP32-C3 |
| CO2 sensor | Sensirion SCD40 (NDIR) |
| Air quality sensors, prototype | PMS5003 (particulates) and SGP30 (VOC) |
| Air quality sensor, PCB version | Sensirion SEN55 |
| Indicator | Single WS2812B NeoPixel |
| Power | USB-C bus power |

The first prototype uses breakout boards on a breadboard.

## Roadmap

`PLAN.md` covers each stage in detail:

1. Requirements and thresholds
2. Hardware selection
3. Firmware development
4. PCB design
5. Enclosure design
6. Prototype build and QA
7. Production planning
8. Cost and pricing analysis
9. Ecosystem integration (Matter)
10. Cloud backend and web dashboard
11. FCC compliance
