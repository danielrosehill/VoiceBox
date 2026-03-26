# Feasibility Analysis

## Summary

VoiceBox is **technically feasible** with current off-the-shelf components. The core building blocks (on-device STT, USB HID gadget mode, compact SBC hardware) all exist and have been independently validated. The main engineering challenge is integration and thermal management in a compact form factor.

## STT Inference on Edge Hardware

### RK3588 NPU (Recommended Path)

| Model | Runtime | RTF | Feasibility | Notes |
|-------|---------|-----|-------------|-------|
| SenseVoiceSmall (RKNN2) | RKNN NPU | **0.05** (20x real-time) | High | Best NPU option today; 5 languages, ~1.1 GB RAM |
| whisper.cpp tiny | CPU (A76 cores) | ~0.25-0.3 | High | No NPU needed, proven path |
| whisper.cpp base | CPU (A76 cores) | ~0.6 | Medium | Near real-time, usable for batch mode |
| whisper.cpp small | CPU (A76 cores) | ~1.0 | Low | Borderline real-time, not suitable for streaming |
| Whisper RKNN (INT8) | RKNN NPU | N/A | **Broken** | INT8 quantization produces garbage output (open issue) |

**Key finding:** Whisper on the RK3588 NPU via RKNN is currently broken for INT8 quantization. The best NPU-accelerated option is Alibaba's SenseVoiceSmall, which runs at 20x real-time on a single NPU core. For Whisper specifically, CPU inference with whisper.cpp tiny/base is the proven path.

### Alternative Accelerators

| Platform | Whisper Support | Performance | Cost | Verdict |
|----------|----------------|-------------|------|---------|
| Hailo-8L + RPi 5 | tiny/base only | 8.4x real-time (hybrid mode) | ~$70 add-on | Good but limited to small models |
| Google Coral Edge TPU | Not feasible | N/A | $60-75 | Whisper's flex-ops incompatible with Coral |
| Hailo-10H | Planned for small/medium | TBD | Not yet available | Future option |

### Verdict

The RK3588 path offers the best balance of performance, cost, and flexibility. SenseVoiceSmall on the NPU is the fastest option, with whisper.cpp on CPU as a proven fallback. Hailo-8L is viable but adds cost and limits model choice.

## USB HID Keyboard Emulation

### Maturity: High

- **Raspberry Pi**: Very well-documented. Uses `dwc2` overlay + `libcomposite` kernel module. Multiple mature projects (Key Mime Pi, pi-as-keyboard, keybird).
- **RK3588**: Has 3 DWC3 USB controllers with OTG support. USB gadget mode merged in mainline kernel. Less community documentation than Pi but functional.

### Gotchas

- Not all RK3588 boards expose OTG-capable USB ports physically — board selection matters.
- Device tree modifications may be needed on some boards.
- HID keyboard protocol sends keycodes, not characters — Unicode/IME handling requires careful mapping.
- Composite USB device (HID + mass storage) adds complexity but is well-supported in Linux gadget framework.

## Power Budget

| Component | Draw |
|-----------|------|
| RK3588 SoC idle | 2-3W |
| NPU active (sustained inference) | +2-3W |
| Audio capture + processing | ~0.5W |
| BLE radio | ~0.1W |
| **Total system (STT active)** | **8-12W** |

- USB-C bus power (5V/900mA = 4.5W) is **insufficient** for sustained NPU inference.
- **Requires USB-C PD** (5V/3A = 15W) or dedicated power supply.
- Battery option: 3000mAh Li-Po at 3.7V = ~11Wh → roughly 1-1.5 hours of continuous STT at 8-12W.

## Thermal Management

- RK3588 throttles at 85°C, shuts down at 115°C.
- **Passive cooling is feasible** for NPU-only workloads (~3W additional) with an aluminum heatsink case.
- **Sustained CPU inference** (whisper.cpp) pushes total SoC power higher and will eventually throttle in a sealed small enclosure.
- Recommendation: Aluminum case as heatsink for NPU path; add 25mm fan for CPU-heavy workloads.

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Whisper RKNN INT8 remains broken | Medium | Use SenseVoiceSmall or CPU-based whisper.cpp |
| RK3588 board lacks USB OTG port | High | Verify before purchasing; Orange Pi 5 confirmed |
| Power exceeds USB bus power | Medium | Require USB-C PD or dedicated power; document clearly |
| Thermal throttling in small enclosure | Medium | Aluminum case + optional fan; test early |
| Unicode/IME via HID keycodes | Low | Solvable with USB HID descriptor customization |
| OS-level STT improves, reducing demand | Low | VoiceBox still wins on universality and portability |
