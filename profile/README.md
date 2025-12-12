# Sky1 Linux

Linux distribution for CIX Sky1 SoC (Radxa Orion O6) based on Debian Sid with mainline kernel support.

## Quick Start

```bash
# Add repository
wget -qO- https://sky1-linux.github.io/apt/key.gpg | sudo tee /usr/share/keyrings/sky1-linux.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/sky1-linux.asc] https://sky1-linux.github.io/apt sid main non-free-firmware" | sudo tee /etc/apt/sources.list.d/sky1-linux.list
sudo apt update

# Install everything for desktop use
sudo apt install sky1-desktop

# Or minimal (kernel + firmware + 5GbE)
sudo apt install sky1-minimal
```

## Repositories

### Core

| Repository | Description |
|------------|-------------|
| [apt](https://github.com/Sky1-Linux/apt) | APT repository (packages & installation guide) |
| [linux-sky1](https://github.com/Sky1-Linux/linux-sky1) | Linux 6.18 kernel with Sky1 patches |
| [sky1-firmware](https://github.com/Sky1-Linux/sky1-firmware) | GPU, DSP, VPU, WiFi firmware |

### Drivers (DKMS)

| Repository | Description |
|------------|-------------|
| [sky1-drivers-dkms](https://github.com/Sky1-Linux/sky1-drivers-dkms) | Out-of-tree kernel modules |
| ↳ r8126 | Realtek RTL8126 5GbE |
| ↳ vpu | ARM Linlon MVE v8 (video encode/decode) |
| ↳ npu | ARM Zhouyi V3 (30 TOPS AI accelerator) |

### Multimedia

| Repository | Description |
|------------|-------------|
| [ffmpeg-sky1](https://github.com/Sky1-Linux/ffmpeg-sky1) | FFmpeg with V4L2 AV1/VP9 support |
| [gstreamer-sky1](https://github.com/Sky1-Linux/gstreamer-sky1) | GStreamer with v4l2av1dec |
| [libva-v4l2-stateful](https://github.com/Sky1-Linux/libva-v4l2-stateful) | VA-API for Firefox/Chromium hardware decode |

## Hardware

| Component | Details |
|-----------|---------|
| SoC | CIX CD8180 (Sky1) |
| CPU | 12-core ARM (4x A720 big + 4x A720 medium + 4x A520 little) |
| GPU | Mali-G720-Immortalis (Panthor driver) |
| VPU | ARM Linlon MVE v8, 5 AEU cores |
| NPU | ARM Zhouyi V3, 30 TOPS |
| Ethernet | RTL8126 5GbE |

## Status

- Kernel 6.18 LTS with Panthor GPU
- Hardware video decode: H.264, HEVC, VP8, VP9, AV1
- Hardware video encode: H.264, HEVC, VP8, VP9
- WiFi/BT: RTL8852BE supported
- 5GbE: RTL8126 via DKMS

## License

- Kernel patches: GPL-2.0
- Firmware: Redistributable (see individual licenses)
- Userspace tools: Various open source licenses
