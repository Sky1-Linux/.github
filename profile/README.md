# Sky1 Linux

**Making CIX Sky1 boards actually usable on mainline Linux.**

## The Problem

CIX Sky1 (CD8180) boards pack impressive hardware—a 12-core ARM CPU, Mali-G720 GPU, 30 TOPS NPU, hardware video codec—but mainline Linux doesn't support most of it. CIX Technology has begun upstreaming foundational SoC support (pinctrl, mailbox, PCIe, base DTS) in v6.19, but critical subsystems remain out-of-tree:

| Subsystem | Mainline v6.19 Status | Our Patches |
|-----------|----------------------|-------------|
| Display (DP/HDMI) | Not submitted | ~27,000 lines |
| Audio (HDA + DSP) | Stalled | ~8,000 lines |
| USB-C (PD + DP Alt Mode) | Not submitted | ~5,000 lines |
| GPU init sequence | Not submitted | ~200 lines |
| NPU (AI accelerator) | Not submitted | ~12,000 lines |
| VPU (video codec) | Not submitted | ~39,000 lines |
| Ethernet (5GbE / 2.5GbE) | Not submitted | ~53,000 lines |

**Result:** Unpatched mainline Linux 6.19 boots to a serial console with NVMe and basic PCIe. No display, no audio, no GPU, no networking.

## What We Provide

Sky1 Linux maintains patched kernels across multiple tracks:

| Track | Kernel | Patches | Status |
|-------|--------|---------|--------|
| **LTS** | Linux 6.18.7 | 78 patches | Stable, recommended |
| **RC** | Linux 6.19-rc7 | 12 patches (consolidated) | Testing |

The RC track has fewer patches because they are consolidated by subsystem. We fully replace the minimal upstream CIX drivers (PCIe, pinctrl, DTS) with production-quality, board-tested versions and add all subsystems not yet submitted upstream.

Users opt into tracks via APT components:

```bash
# LTS only (default, recommended)
deb https://sky1-linux.github.io/apt sid main

# LTS + Latest stable
deb https://sky1-linux.github.io/apt sid main latest

# All tracks including RC testing
deb https://sky1-linux.github.io/apt sid main latest rc
```

### Kernel & Drivers

| Feature | Status |
|---------|--------|
| Display output (4K@60 DP/HDMI) | Working |
| GPU acceleration (Vulkan/OpenGL ES) | Working via Panthor |
| Audio (HDA speakers + headphones) | Working |
| USB-C Power Delivery (up to 100W) | Working |
| USB-C DisplayPort Alt Mode | Working |
| WiFi 6 (RTL8852BE) | Working |
| Dual 5GbE (RTL8126) / 2.5GbE (RTL8125) | Working |
| PCIe (NVMe, GPU, WiFi) | Working, no kernel params needed |
| Hardware video decode (H.264/HEVC/AV1/VP9) | Working via VPU |
| AI accelerator (30 TOPS) | Working via NPU |

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
| [linux-sky1](https://github.com/Sky1-Linux/linux-sky1) | Kernel patches, configs, and build metadata (LTS/Latest/RC/Next tracks) |
| [linux](https://github.com/Sky1-Linux/linux) | Full kernel source (mainline + patches) |
| [sky1-firmware](https://github.com/Sky1-Linux/sky1-firmware) | GPU, DSP, VPU, WiFi firmware |
| [sky1-drivers-dkms](https://github.com/Sky1-Linux/sky1-drivers-dkms) | DKMS drivers (GPU, VPU, NPU) for vendor kernel compatibility |
| [sky1-linux-build](https://github.com/Sky1-Linux/sky1-linux-build) | Kernel package build scripts |

### Installer & Live ISO

| Repository | Description |
|------------|-------------|
| [sky1-live-build](https://github.com/Sky1-Linux/sky1-live-build) | Live ISO build configuration |
| [calamares-settings-sky1](https://github.com/Sky1-Linux/calamares-settings-sky1) | Installer branding and config |
| [plasma-setup](https://github.com/Sky1-Linux/plasma-setup) | KDE Plasma first-boot user creation wizard |

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

Our patchset includes drivers and fixes not available upstream:

- **linlon-dp / trilin-dpsub** — Display processor and DP transmitter (4K@60 HDMI/DP)
- **Panthor power sequence** — Sky1-specific GPU initialization for Mali-G720
- **CIX IPBLOQ HDA + SOF DSP** — Audio controller and DSP firmware loading
- **RTS5453** — USB-C PD controller for power negotiation and DP Alt Mode
- **ARM Linlon VPU** — Hardware video encode/decode (H.264, HEVC, AV1, VP9)
- **ArmChina Zhouyi NPU** — AI accelerator driver (30 TOPS, 3-core X2_1204MP3)
- **Realtek RTL8126/RTL8125** — 5GbE and 2.5GbE ethernet (in-tree, no DKMS)
- **PCIe hotplug fixes** — AER/PME coordination, WiFi scan offload robustness
- **Bus frequency scaling** — CI700/NI700 interconnect performance management

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
2. **Track upstream** — Four kernel tracks (LTS, Latest, RC, Next) follow mainline releases
3. **Complete stack** — Kernel, firmware, drivers, and multimedia tools
4. **Enable upstreaming** — Clean patches organized by subsystem, ready for submission
5. **Multi-board** — SoC-level support works across all Sky1 boards; only device trees differ

## License

- Kernel patches: GPL-2.0
- Firmware: Redistributable per vendor terms
- Userspace packages: Various open source licenses
