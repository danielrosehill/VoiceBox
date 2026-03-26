# VoiceBox: Portable Hardware Speech-to-Text Keyboard

## Overview

VoiceBox is a compact, portable hardware device that performs on-device speech-to-text inference and presents itself as a standard USB HID keyboard to any connected host. It replaces or supplements a traditional keyboard by converting spoken input into typed text, with no host-side software or driver installation required.

The device runs open-source STT models locally on an embedded NPU/TPU, supports user-configurable word lists for domain-specific correction, and connects via USB-C (as HID keyboard) or Bluetooth LE. It is fully OS-agnostic — any device that accepts a USB keyboard or Bluetooth keyboard can receive text from VoiceBox.

## Problem Statement

Speech-to-text has matured significantly and can now run via local inference with high accuracy. However, practical deployment faces several friction points:

1. **Host dependency**: Most STT solutions require GPU-accelerated hardware and model installation on the host machine.
2. **OS fragmentation**: Direct text-entry-level STT support varies wildly across operating systems, especially on Linux, ChromeOS, and embedded systems.
3. **Portability**: Users cannot easily carry their preferred model, word lists, and configuration between machines.
4. **Setup overhead**: Installing models, configuring custom dictionaries, and maintaining consistency across devices is burdensome.

## Core Concept

A self-contained hardware device that:

- Captures audio via an integrated high-quality microphone
- Performs STT inference entirely on-device using an embedded AI accelerator
- Emits recognized text as standard USB HID keystrokes (and/or Bluetooth LE keystrokes)
- Requires zero software installation on the host
- Allows the user to carry their model, configuration, and word lists physically between machines

## Functional Requirements

### F1: On-Device Speech-to-Text

- Run open-source STT models (e.g., Whisper, Whisper.cpp, Vosk, faster-whisper) on an embedded NPU, TPU, or edge AI accelerator.
- Target real-time factor (RTF) < 0.5 for streaming transcription (text appears as user speaks, not after a long pause).
- Support model hot-swapping: user can load different models for different languages or accuracy profiles.
- Minimum viable model size: Whisper small (~244M parameters) or equivalent.
- Stretch goal: Whisper medium (~769M parameters) at acceptable latency.

### F2: USB HID Keyboard Emulation

- Present as a standard USB HID keyboard when connected via USB-C.
- No drivers, no host software, no permissions required.
- Works on any OS: Windows, macOS, Linux, ChromeOS, Android, iOS (via adapter), embedded systems, game consoles, smart TVs.
- Keystroke injection should handle standard alphanumeric characters, punctuation, Enter, Tab, Backspace, and common modifier combinations.

### F3: Bluetooth LE Keyboard

- BLE HID profile for wireless connection to tablets, phones, and other Bluetooth-capable devices.
- Pairing with multiple devices (minimum 3 saved pairings).
- Seamless switching between USB and Bluetooth output.

### F4: Microphone Subsystem

- Integrated microphone optimized for near-field speech capture in varied environments.
- Primary design: gooseneck-mounted directional microphone (adjustable, positions near mouth).
- Alternative/additional: built-in MEMS microphone array with beamforming for desk use.
- Optional: 3.5mm aux input for external microphone (headset, lapel mic).
- Hardware-level noise suppression (dedicated DSP or pre-processing on the main SoC).

### F5: Custom Word Lists and Post-Processing

- User-configurable word list for find-and-replace correction (e.g., domain jargon, proper nouns, abbreviations).
- Word lists stored on-device, editable via companion app or USB file transfer.
- Support for multiple word list profiles (e.g., "Medical", "Legal", "Programming").
- Basic post-processing pipeline: capitalize sentences, expand abbreviations, apply word list replacements.

### F6: Device Administration

- **Companion app** (smartphone, cross-platform): manage models, word lists, device settings, firmware updates.
- **USB mass storage mode**: device mounts as a USB drive for direct file management (models, word lists, config).
- **Web UI** (stretch goal): device hosts a local web interface when connected via USB, accessible at a fixed IP or mDNS hostname.

### F7: Multi-Device Sync (Stretch Goal)

- If a user owns multiple VoiceBox units, sync word lists, model selection, and settings between them.
- Sync via cloud service (optional, user-controlled) or local network.

## Hardware Architecture

### Compute Platform

| Component | Purpose | Candidate Options |
|-----------|---------|-------------------|
| Main SoC | Application logic, USB/BLE stack, audio pipeline | Raspberry Pi CM4/CM5, Orange Pi 5 (RK3588S), BeagleBoard AI-64 |
| AI Accelerator | STT model inference | Google Coral Edge TPU (USB or M.2), Hailo-8/8L, Rockchip NPU (built into RK3588), Intel Movidius |
| Microcontroller | USB HID emulation, low-power BLE | RP2040, ESP32-S3, nRF52840 |

**Recommended primary path**: RK3588S-based SBC (e.g., Orange Pi 5) — has a built-in 6 TOPS NPU, sufficient for Whisper small/medium inference, and strong community support for RKNN toolkit.

**Alternative lean path**: Raspberry Pi Zero 2W + Google Coral USB Accelerator — smaller form factor, well-documented, but tighter thermal and power constraints.

### Audio

| Component | Purpose | Candidates |
|-----------|---------|------------|
| Primary mic | Near-field speech capture | Gooseneck-mounted electret or MEMS directional mic |
| Mic array (optional) | Beamforming for desk use | 2-4x MEMS microphones (e.g., SPH0645LM4H) with DSP |
| Audio codec/ADC | Analog-to-digital conversion | I2S MEMS mic (digital out), or dedicated ADC (e.g., PCM1808) |
| Noise suppression | Pre-processing | RNNoise (software), or dedicated DSP (e.g., Qualcomm QCC5171) |

### Connectivity

| Interface | Implementation |
|-----------|---------------|
| USB-C (primary) | USB 2.0: composite device — HID keyboard + mass storage + CDC (serial for debug) |
| Bluetooth LE | BLE HID profile via onboard BLE radio (ESP32-S3 or nRF52840) or USB BLE dongle |
| Wi-Fi (optional) | For companion app communication, OTA updates, multi-device sync |

### Power

- Powered primarily via USB-C from the host (bus-powered, 5V/500mA–900mA).
- Optional internal battery (Li-Po, ~2000–3000mAh) for Bluetooth-only use.
- If battery-powered: target 2–4 hours continuous dictation, 24+ hours standby.

### Storage

- On-device storage for models, word lists, firmware: 32–128GB eMMC or microSD.
- Whisper small: ~500MB; Whisper medium: ~1.5GB; leave room for multiple models and languages.

### Physical Design

- **Form factor**: Compact desktop puck or wedge, roughly 100mm x 80mm x 30mm (smaller than a deck of cards).
- **Gooseneck mic**: Flexible ~150mm stalk, detachable for transport.
- **Controls**: Physical mute button, mode toggle (USB/BLE), LED status indicators (listening, processing, muted, connected).
- **Enclosure**: 3D-printable shell for prototyping; injection-molded for production.
- **Weight target**: < 200g (without battery), < 300g (with battery).
- **Portability**: Fits in a jacket pocket or small bag compartment.

## Software Architecture

```
+--------------------+      +-------------------+      +------------------+
|   Audio Capture    | ---> |   STT Engine      | ---> | Post-Processing  |
| (ALSA/PulseAudio)  |      | (Whisper.cpp /    |      | (word lists,     |
| + noise suppression|      |  RKNN / Coral)    |      |  capitalization)  |
+--------------------+      +-------------------+      +------------------+
                                                              |
                                                              v
                                                      +------------------+
                                                      |  HID Emitter     |
                                                      | (USB HID keyboard|
                                                      |  + BLE HID)      |
                                                      +------------------+
```

### Key Software Components

| Component | Technology | Notes |
|-----------|-----------|-------|
| STT runtime | whisper.cpp (GGML), or RKNN SDK for RK3588 NPU | Model-agnostic abstraction layer |
| Audio pipeline | ALSA → RNNoise → VAD → STT engine | Voice Activity Detection to avoid processing silence |
| HID emitter | Custom firmware on RP2040/ESP32-S3, or Linux USB gadget mode | Translates text → HID keycodes |
| Configuration | JSON/YAML config files on filesystem | Model path, word lists, language, output mode |
| Companion app | React Native or Flutter | BLE communication for settings; HTTP for Wi-Fi mode |
| OTA updates | Mender.io or RAUC | A/B partition firmware updates |

### Voice Activity Detection (VAD)

- Silero VAD or WebRTC VAD to detect speech segments.
- Reduces power consumption and avoids spurious transcription.
- Configurable sensitivity threshold.

### Streaming vs. Batch Transcription

- **Streaming mode** (default): transcribe in ~1-3 second chunks as user speaks. Text appears in near real-time.
- **Batch mode**: user presses button to start, presses again to stop, entire utterance transcribed at once. Higher accuracy for long passages.
- Mode selectable via physical button or companion app.

## Open-Source STT Models (Candidates)

| Model | Parameters | Size on Disk | Languages | Notes |
|-------|-----------|-------------|-----------|-------|
| Whisper tiny | 39M | ~75MB | 99 | Fast but lower accuracy |
| Whisper base | 74M | ~142MB | 99 | Good balance for resource-constrained devices |
| Whisper small | 244M | ~466MB | 99 | Primary target for VoiceBox |
| Whisper medium | 769M | ~1.5GB | 99 | Stretch goal; excellent accuracy |
| Vosk (various) | Varies | 50MB–1.5GB | 20+ | Lightweight, good offline support |
| faster-whisper | Same as Whisper | Smaller (CTranslate2) | 99 | Optimized inference, lower memory |
| Distil-Whisper | ~166M | ~330MB | English-focused | Distilled; faster with minimal accuracy loss |

## Prior Art and Existing Solutions

Research these to avoid reinventing the wheel:

| Project/Product | Type | Relevance |
|-----------------|------|-----------|
| Mycroft Mark II | Open-source voice assistant hardware | Hardware design, microphone array, enclosure |
| Whisper.cpp | Optimized Whisper inference in C++ | Core STT engine candidate |
| Coral AI projects | Edge TPU inference examples | NPU integration patterns |
| Talon Voice | Voice coding toolkit | Post-processing, command grammar |
| Nerd Dictation | Linux offline STT keyboard input | Software-side HID injection approach |
| ESP32 Whisper projects | Microcontroller STT | Ultra-low-cost inference experiments |
| Reduced Keyboard (FrogPad, etc.) | Compact keyboard hardware | Form factor inspiration |
| Bee (formerly Limitless) | AI wearable recorder | Hardware miniaturization, mic design |

## Development Phases

### Phase 1: Proof of Concept
- Run Whisper.cpp on an SBC (Raspberry Pi 5 or Orange Pi 5).
- USB gadget mode to emit HID keystrokes from transcribed text.
- Wired USB microphone input.
- Validate end-to-end latency and accuracy.

### Phase 2: Integrated Prototype
- Custom PCB or SBC + HAT design with integrated mic and AI accelerator.
- 3D-printed enclosure with gooseneck mic mount.
- BLE HID support.
- Basic word list / post-processing pipeline.
- Companion app (BLE config).

### Phase 3: Refinement
- Optimized enclosure (smaller, better thermal management).
- Multiple model support and hot-swapping.
- OTA firmware updates.
- Multi-device sync.
- Battery operation for wireless use.

### Phase 4: Community Release
- Open-source hardware files (KiCad schematics, 3D-printable enclosure STLs).
- Open-source firmware and software.
- BOM (Bill of Materials) with sourcing links.
- Assembly guide and documentation.

## Success Criteria

- **Latency**: Text appears within 1–2 seconds of speech completion (streaming mode).
- **Accuracy**: Comparable to host-based Whisper small (< 10% WER on common English benchmarks).
- **Compatibility**: Works as a keyboard on at least 5 different OSes without any host software.
- **Portability**: Entire device fits in a pocket or small bag; setup time on a new machine is under 10 seconds (plug in USB or pair BLE).
- **Cost**: BOM target < $100 USD for the prototype; < $60 USD at modest scale.

## Open Questions

1. **NPU selection**: RK3588 built-in NPU vs. dedicated Coral/Hailo accelerator — which gives better perf/watt for Whisper-class models?
2. **Streaming latency**: Can sub-2s latency be achieved on edge hardware with Whisper small, or do we need a lighter model (Distil-Whisper, Whisper base)?
3. **USB HID limitations**: HID keyboard protocol sends keycodes, not characters — handling Unicode, IME input, and special characters across OSes needs investigation.
4. **Thermal management**: Sustained inference on a compact device will generate heat. Passive cooling sufficient, or do we need a micro-fan?
5. **Market/community validation**: Is there sufficient demand to justify the engineering effort, or does improving host-side STT make this unnecessary within 1–2 years?
