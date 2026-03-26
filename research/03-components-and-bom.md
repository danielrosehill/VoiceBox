# Suggested Components and Bill of Materials

## Recommended Architecture: RK3588-Based

### Core Components

| Component | Recommended Part | Purpose | Est. Cost |
|-----------|-----------------|---------|-----------|
| **Main SBC** | Orange Pi 5 (8GB) | RK3588S SoC, 6 TOPS NPU, USB-C OTG | $80-90 |
| **Alternative SBC** | Orange Pi 5B (8GB) | Same SoC, onboard eMMC + WiFi/BT | $90-100 |
| **Microcontroller** | ESP32-S3 DevKit | BLE HID keyboard, secondary USB HID | $8-12 |
| **Primary Mic** | Gooseneck electret/condenser | Near-field directional speech capture | $10-20 |
| **MEMS Mic (backup)** | INMP441 or SPH0645LM4H (I2S) | Digital mic, direct to SoC | $3-5 |
| **Storage** | 64GB microSD (A2 rated) | Models, word lists, OS | $10-15 |
| **USB-C Cable/Connector** | USB-C to USB-C/A cable | Host connection (HID + power) | $5-8 |

### Optional Components

| Component | Part | Purpose | Est. Cost |
|-----------|------|---------|-----------|
| BLE Module | nRF52840 Dongle | Bluetooth LE HID (if ESP32-S3 not used) | $10-15 |
| External Mic Jack | 3.5mm TRRS connector | Headset/lapel mic input | $2-3 |
| Battery | Li-Po 3000mAh 3.7V | Wireless operation (~1-1.5hr STT) | $10-15 |
| Battery Management | TP4056 + boost converter | Charging + 5V regulation | $3-5 |
| Cooling | 25mm fan (5V) | Active cooling for sustained CPU inference | $3-5 |
| Status LEDs | WS2812B (NeoPixel) x3 | Listening/processing/muted indicators | $2-3 |
| Buttons | Tactile switches x3 | Mute, mode toggle, power | $1-2 |

### Enclosure

| Option | Purpose | Est. Cost |
|--------|---------|-----------|
| 3D-printed (PLA/PETG) | Prototyping | $5-10 (filament) |
| Aluminum heatsink case | Passive cooling + production | $15-25 |
| Custom injection-molded | Production scale | $3-5/unit (after $2-5K tooling) |

## BOM Summary

### Prototype (Phase 1 — PoC)

| Item | Cost |
|------|------|
| Orange Pi 5 (8GB) | $85 |
| 64GB microSD | $12 |
| USB MEMS microphone | $15 |
| USB-C cable | $5 |
| Breadboard + wires | $5 |
| **Total** | **~$122** |

### Integrated Prototype (Phase 2)

| Item | Cost |
|------|------|
| Orange Pi 5B (8GB, onboard eMMC + WiFi/BT) | $95 |
| ESP32-S3 (BLE HID) | $10 |
| Gooseneck directional mic + I2S MEMS backup | $25 |
| 3D-printed enclosure | $10 |
| Status LEDs + buttons | $5 |
| USB-C connector + cable | $8 |
| Misc (PCB adapter, wiring, connectors) | $15 |
| **Total** | **~$168** |

### Production Target (Phase 4)

| Item | Target Cost |
|------|-------------|
| RK3588S compute module (custom PCB) | $35-45 |
| Audio subsystem (MEMS + codec) | $5-8 |
| BLE module | $3-5 |
| Storage (32GB eMMC) | $5-8 |
| Enclosure (injection-molded aluminum) | $8-12 |
| Gooseneck mic assembly | $5-8 |
| USB-C + connectors | $3-5 |
| PCB + assembly | $8-12 |
| **Target BOM** | **~$72-103** |

*At modest scale (500+ units), the $60-80 BOM target from the spec is achievable but aggressive. The RK3588S compute module is the main cost driver.*

## Alternative: Budget Path (Raspberry Pi)

| Item | Cost |
|------|------|
| Raspberry Pi 5 (4GB) | $60 |
| Hailo-8L AI HAT | $70 |
| USB audio + mic | $15 |
| microSD 32GB | $8 |
| 3D-printed case | $10 |
| **Total** | **~$163** |

*More expensive than Orange Pi path, but better documented USB gadget support. Hailo-8L limits to Whisper tiny/base.*

## Software Stack (No Cost)

| Component | Software | License |
|-----------|----------|---------|
| OS | Armbian / DietPi | Open source |
| STT (NPU) | SenseVoiceSmall + RKNN | Apache 2.0 / Proprietary SDK |
| STT (CPU) | whisper.cpp | MIT |
| VAD | Silero VAD | MIT |
| Noise suppression | RNNoise | BSD-3 |
| USB HID | Linux USB gadget (libcomposite) | GPL |
| BLE HID | ESP-IDF BLE HID library | Apache 2.0 |
| Companion app | React Native / Flutter | MIT/BSD |

## Sourcing Notes

- Orange Pi boards available from official store, AliExpress, Amazon
- ESP32-S3 widely available from Espressif official distributors
- MEMS microphones from Adafruit, SparkFun, or direct from LCSC/Mouser
- Gooseneck mic assemblies from audio equipment suppliers or AliExpress
- 3D printing via local printer or services (JLCPCB, PCBWay)
