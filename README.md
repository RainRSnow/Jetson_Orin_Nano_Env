# Jetson_Orin_Nano_Env

Environment deployment and setup notes for NVIDIA Jetson Orin Nano running JetPack 6.2. This repository collects scripts, tips, and references to get a working development environment (Python, PyTorch, CUDA, TensorRT, etc.) on the Jetson Orin Nano Engineering Developer Kit.

## Table of contents
- [Jetson\_Orin\_Nano\_Env](#jetson_orin_nano_env)
  - [Table of contents](#table-of-contents)
  - [Overview](#overview)
  - [Supported hardware \& baseline software](#supported-hardware--baseline-software)
  - [Quick links \& resources](#quick-links--resources)
  - [Environment setup (recommended)](#environment-setup-recommended)
  - [JetPack 6.2 installation](#jetpack-62-installation)
  - [PyTorch on Jetson](#pytorch-on-jetson)
    - [PyTorch 2.3 (official stable)](#pytorch-23-official-stable)
    - [PyTorch 2.8 (latest)](#pytorch-28-latest)
    - [Warning about pip installs](#warning-about-pip-installs)
  - [Troubleshooting \& notes](#troubleshooting--notes)
  - [Contributing](#contributing)
  - [License](#license)

## Overview
Jetson devices (ARM + NVIDIA Tegra) differ from x86 desktops: many libraries and drivers must be matched to the L4T/JetPack and sometimes compiled from source. This README documents a recommended baseline environment and provides the commands and references used to prepare a Jetson Orin Nano (JetPack 6.2 / L4T 36.4.7) development system.

## Supported hardware & baseline software
Target device:
- NVIDIA Jetson Orin Nano Engineering Reference Developer Kit Super

Baseline OS and components observed on a tested system:

```
L4T 36.4.7 [ JetPack 6.2 ]
  Ubuntu 22.04.5 LTS
  Kernel Version: 5.15.148-tegra
CUDA 12.6.68
  CUDA Architecture: 8.7
OpenCV: 4.8.0
  OpenCV CUDA: NO
cuDNN: libcudnn9 (package installed)
TensorRT: 10.3.0.30
VisionWorks: NOT_INSTALLED
VPI: 3.2.4
Vulkan: 1.3.204
GCC: gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04.2)
Python: Python 3.10.12
```

Note: Some components (OpenCV with CUDA, VisionWorks, etc.) may require custom builds or JetPack-specific binaries.

## Quick links & resources
- Jetson Orin Nano tutorial (quick start video): [Jetson Orin Nano Tutorial: SSD Install, Boot, and JetPack Setup](https://youtu.be/q4fGac-nrTI?si=iwUvrzVgH_T1VUZr)
- JetsonHacks (collection of scripts & repos): [jetsonhacks on GitHub](https://github.com/jetsonhacks?tab=repositories)
- Jetson Orin setup scripts (JetsonHacks): [jetsonhacks/jetson-orin-setup](https://github.com/jetsonhacks/jetson-orin-setup)
- Pre-built Jetson Python wheels (community): [Jetson AI Lab PyPI index](https://pypi.jetson-ai-lab.io/)
- (Local/internal) Zhao's Jetson repo (reference in original notes): csv:/home/zhao/git/jetson.git — adapt to your accessible repo URL or mirror if needed.

## Environment setup (recommended)
Jetson deployments often require JetPack-specific packages or builds. Before installing additional software, apply a basic setup. Replace any internal clone URLs with the correct remote if required.

Example (from local/internal repo):
```bash
git clone https://github.com/RainRSnow/Jetson_Orin_Nano_Env.git
cd Jetson_Orin_Nano_Env/setup
chmod +x setup_jetson.sh
./setup_jetson.sh
```

What the `setup_jetson.sh` helper typically performs:
- System packages: `sudo apt update` and `sudo apt upgrade -y`
- Installs Chromium: `sudo apt install chromium-browser`
- Installs pip3: `sudo apt install python3-pip -y`
- Installs/updates jetson-stats (includes `jtop`): `sudo pip3 install -U jetson-stats`
- Installs Visual Studio Code via helper script `scripts/install_vscode.sh` (if present)
- Optionally pins GNOME Terminal, Chromium, and VS Code to the dock and adjusts GNOME Terminal font size (if helper scripts are present)

Run the setup script and review its output; adjust or skip steps that don't apply to your environment.

## JetPack 6.2 installation
JetPack is the official SDK for Jetson devices and provides the compatible L4T, drivers, CUDA, TensorRT, etc.

To install using the apt package (on systems where `nvidia-jetpack` is available):
```bash
sudo apt update
sudo apt install nvidia-jetpack
```

Note: `nvidia-jetpack` may include subpackages such as `nvidia-jetpack-runtime` and `nvidia-jetpack-dev`. Install only the subpackages you need if you want a lighter installation.

Always confirm the JetPack/L4T version after installation:
```bash
head -n 1 /etc/nv_tegra_release
```

## PyTorch on Jetson
Installing PyTorch on Jetson requires matching the wheel (aarch64) to your JetPack/CUDA/cuDNN combination. Using prebuilt, tested install scripts or wheels is strongly recommended.

### PyTorch 2.3 (official stable)
Reference: [NVIDIA Developer Forums - PyTorch for Jetson](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048)

From the helper repo:
```bash
cd Jetson_Orin_Nano_Env/setup/scripts/utils
chmod +x install_pytorch23_jetpack6.sh
sudo ./install_pytorch23_jetpack6.sh
```

If successful, expected installed packages:
- torch-2.3.0-cp310-cp310-linux_aarch64.whl
- torchaudio-2.3.0+952ea74-cp310-cp310-linux_aarch64.whl
- torchvision-0.18.0a0+6043bc2-cp310-cp310-linux_aarch64.whl

### PyTorch 2.8 (latest)
From the helper repo:
```bash
cd Jetson_Orin_Nano_Env/setup/scripts/utils
chmod +x install_pytorch_jetson.sh
sudo ./install_pytorch_jetson.sh
```

If successful, expected installed packages:
- torch-2.8.0-cp310-cp310-linux_aarch64.whl
- torchaudio-2.8.0-cp310-cp310-linux_aarch64.whl
- torchvision-0.23.0-cp310-cp310-linux_aarch64.whl

### Warning about pip installs
Do NOT run a plain `pip install torch torchaudio torchvision` on Jetson unless you know the wheel target and compatibility. PyPI or generic wheels may install versions incompatible with your JetPack/CUDA/cuDNN and will result in loss of CUDA support or runtime errors. Always use wheels built for your JetPack / L4T / CUDA version or the community-provided Jetson wheels.

## Troubleshooting & notes
- Match versions: JetPack (L4T), CUDA, cuDNN, and TensorRT versions must be compatible. Changing one component (e.g., CUDA) often requires rebuilding or reinstalling other libraries.
- OpenCV CUDA: Many Jetson installs use OpenCV without CUDA-enabled builds. If you need OpenCV with CUDA support, build OpenCV from source against your CUDA and gstreamer stack.
- Use `jetson_stats` / `jtop` to inspect GPU usage, temperature, and processes.
- If a helper script references local/internal paths (e.g., csv:/home/zhao/...), replace them with accessible remote URLs or mirror the content onto your network.
- Keep a separate backup image of a working SD/eMMC storage before performing major upgrades.

## Contributing
If you have additional setup scripts, tested wheels, or instructions that simplify Jetson Orin Nano setup, please open an issue or PR in this repository (or add a folder with scripts and a README describing usage).

## License
This project is licensed under the MIT License — see the [LICENSE](./LICENSE) file for details.