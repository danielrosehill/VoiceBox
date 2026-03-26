# VoiceBox

> **Status: Work in Progress** — concept and research phase

Portable hardware speech-to-text device that emulates a USB HID keyboard. Plug in, talk, and text appears — no drivers, no host software, any OS.

![VoiceBox Concept Renders](renders/composite.png)

## Concept

VoiceBox is a compact, self-contained device that captures speech via an integrated microphone, runs STT inference on an embedded NPU, and emits the recognized text as standard USB HID keystrokes. It works on any device that accepts a USB keyboard — Windows, macOS, Linux, ChromeOS, Android, iOS, game consoles, smart TVs, kiosks — with zero setup.

Users carry their preferred STT model, custom word lists, and configuration physically between machines.

**[Full Specification](spec/SPECIFICATION.md)** | **[Original Idea](idea/raw.md)**

## Research

| Document | Summary |
|----------|---------|
| [Feasibility Analysis](research/01-feasibility.md) | Technical viability of on-device STT, USB HID emulation, power, and thermal |
| [Market Analysis](research/02-market-analysis.md) | Existing products, competitive landscape, and VoiceBox's unique niche |
| [Components and BOM](research/03-components-and-bom.md) | Suggested hardware, bill of materials, and cost estimates |

### Key Findings

- **No existing product** combines on-device STT with USB HID keyboard output
- **RK3588 NPU** can run SenseVoiceSmall at 20x real-time; whisper.cpp tiny at ~3x real-time on CPU
- **USB HID gadget mode** is mature on Linux SBCs (Raspberry Pi and RK3588)
- **Prototype BOM** estimated at ~$120-170; production target ~$70-100
- **Power draw** of 8-12W requires USB-C PD (not bus power alone)

## Hardware Target

- **SoC**: RK3588S (Orange Pi 5) — 6 TOPS NPU, quad A76 + quad A55
- **Form factor**: Desktop puck/wedge, ~100mm x 80mm x 30mm
- **Connectivity**: USB-C (HID keyboard) + Bluetooth LE
- **Microphone**: Gooseneck directional mic + onboard MEMS fallback
- **Storage**: 32-128GB (models, word lists, firmware)
