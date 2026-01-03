# Sky1 Linux

**Making CIX Sky1 boards actually usable on mainline Linux.**

## The Problem

CIX Sky1 (CD8180) boards pack impressive hardware—a 12-core ARM CPU, Mali-G720 GPU, 30 TOPS NPU, hardware video codec—but mainline Linux doesn't support most of it. CIX Technology is upstreaming basic SoC support (pinctrl, mailbox, PCIe), but critical subsystems have no upstream path:

| Subsystem | Mainline Status | Lines of Code Missing |
|-----------|-----------------|----------------------|
| Display (DP/HDMI) | Not submitted | ~12,000 |
| Audio (HDA + DSP) | Stalled | ~8,000 |
| USB-C (PD + DP Alt Mode) | Not submitted | ~800 |
| GPU init sequence | Not submitted | ~1,500 |

**Result:** Mainline Linux 6.19 boots to a serial console with NVMe. That's it.

## What We Provide

Sky1 Linux maintains 70 patches on Linux 6.18.3 LTS plus a complete multimedia stack:

### Kernel & Drivers

| Feature | Status |
|---------|--------|
| Display output (4K@60 DP/HDMI) | Working |
| GPU acceleration (Vulkan/OpenGL ES) | Working via Panthor |
| Audio (HDA speakers + headphones) | Working |
| USB-C Power Delivery (up to 100W) | Working |
| USB-C DisplayPort Alt Mode | Working |
| WiFi 6 (RTL8852BE) | Working |
| Dual 5GbE (RTL8126) | Working |
| PCIe (NVMe, GPU, WiFi) | Working, no kernel params needed |

### Hardware Video (VPU)

The ARM Linlon MVE v8 VPU provides hardware-accelerated video encode/decode. We maintain patched applications that use the V4L2 stateful (M2M) API directly:

| Component | What We Added |
|-----------|---------------|
| **Firefox** | V4L2-M2M decode for H.264, HEVC, VP9, AV1 |
| **Chromium** | V4L2-M2M config package (uses unmodified Debian Chromium) |
| **FFmpeg** | V4L2 M2M support for AV1, VP9, HEVC, H.264 decode/encode |
| **GStreamer** | v4l2av1dec element for AV1 hardware decode |

Supported codecs:
- **Decode:** H.264, HEVC, VP8, VP9, AV1, MPEG-2, MPEG-4
- **Encode:** H.264, HEVC, VP8, VP9

## Quick Start

```bash
# Add repository
wget -qO- https://sky1-linux.github.io/apt/key.gpg | sudo tee /usr/share/keyrings/sky1-linux.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/sky1-linux.asc] https://sky1-linux.github.io/apt sid main non-free-firmware" | sudo tee /etc/apt/sources.list.d/sky1-linux.list
sudo apt update

# Full desktop (includes hardware video support)
sudo apt install sky1-desktop

# Or minimal server
sudo apt install sky1-minimal
```

## Repositories

### Core

| Repository | Description |
|------------|-------------|
| [apt](https://github.com/Sky1-Linux/apt) | APT repository with installation guide |
| [linux-sky1](https://github.com/Sky1-Linux/linux-sky1) | Linux 6.18.3 LTS with 70 Sky1 patches |
| [linux](https://github.com/Sky1-Linux/linux) | Full kernel source (mainline + patches) |
| [sky1-firmware](https://github.com/Sky1-Linux/sky1-firmware) | GPU, DSP, VPU, WiFi firmware |
| [sky1-drivers-dkms](https://github.com/Sky1-Linux/sky1-drivers-dkms) | 5GbE, VPU, NPU kernel modules |
| [sky1-linux-build](https://github.com/Sky1-Linux/sky1-linux-build) | Kernel package build scripts |

### Installer & Live ISO

| Repository | Description |
|------------|-------------|
| [sky1-live-build](https://github.com/Sky1-Linux/sky1-live-build) | Live ISO build configuration |
| [calamares-settings-sky1](https://github.com/Sky1-Linux/calamares-settings-sky1) | Installer branding and config |

### Multimedia (VPU Support)

| Repository | Description |
|------------|-------------|
| [firefox-sky1](https://github.com/Sky1-Linux/firefox-sky1) | Firefox with V4L2-M2M hardware decode |
| [chromium-sky1-config](https://github.com/Sky1-Linux/chromium-sky1-config) | Chromium V4L2-M2M config for Debian package |
| [ffmpeg-sky1](https://github.com/Sky1-Linux/ffmpeg-sky1) | FFmpeg 8.0 with V4L2 M2M codec patches |
| [gstreamer-sky1](https://github.com/Sky1-Linux/gstreamer-sky1) | GStreamer with v4l2av1dec element |
| [libva-v4l2-stateful](https://github.com/Sky1-Linux/libva-v4l2-stateful) | VA-API wrapper (deprecated) |

### Development Tools

| Repository | Description |
|------------|-------------|
| [acpi-to-dts-tools](https://github.com/Sky1-Linux/acpi-to-dts-tools) | ACPI to Device Tree conversion tools for board bringup |

**Have an O6N, MS-R1, OrangePi 6 Plus, or other Sky1 board?** Help us add support by running the hardware extraction script in ACPI mode and submitting the results to [acpi-to-dts-tools](https://github.com/Sky1-Linux/acpi-to-dts-tools). This data helps us create device trees for new boards.

## Key Kernel Patches

Our patchset includes drivers not available upstream:

- **linlon-dp / trilin-dpsub** — Display processor and DP transmitter
- **RTS5453** — USB-C PD controller for power negotiation and DP Alt Mode
- **Panthor power sequence** — Sky1-specific GPU initialization
- **ARM DMA-350 cyclic mode** — Required for audio streaming
- **CIX IPBLOQ HDA** — Audio controller driver
- **PCIe MSI quirk** — Prevents kernel crashes on device enumeration
- **Shared PHY coordination** — Fixes WiFi + 5GbE race conditions

## Hardware

| Component | Specification |
|-----------|---------------|
| **SoC** | CIX CD8180 (Sky1) — ARMv9 |
| **CPU** | 4x Cortex-A720 + 4x Cortex-A720 + 4x Cortex-A520 |
| **GPU** | Mali-G720-Immortalis MC10 |
| **VPU** | ARM Linlon MVE v8 (5 AEU cores) |
| **NPU** | ARM Zhouyi V3 (30 TOPS) |
| **Memory** | Up to 64GB LPDDR5 |

## Supported Boards

Currently tested and supported:
- **Radxa Orion O6** — Full support

Other Sky1 boards we aim to support:
- Radxa Orion O6N
- Minisforum MS-R1
- OrangePi 6 Plus
- MetaComputing ARM AI PC

Most of our work is at the SoC level and should work across all Sky1 boards:

| Component | Portability |
|-----------|-------------|
| GPU, VPU, NPU, Audio, USB-C PD | Universal (SoC drivers) |
| Display, PCIe, Ethernet, WiFi | Mostly universal (may need DT tweaks) |
| GPIO, regulators, board ID | Board-specific (device tree) |

If you have a Sky1 board and want to help add support, contributions are welcome.

## Project Goals

1. **Usable today** — Full hardware support on current mainline LTS
2. **Track upstream** — Rebase as new stable releases come out
3. **Complete stack** — Kernel, firmware, drivers, and multimedia tools
4. **Enable upstreaming** — Clean patches that could be submitted

## License

- Kernel patches: GPL-2.0
- Firmware: Redistributable per vendor terms
- Userspace packages: Various open source licenses
