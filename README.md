# ROCm + Pytorch Installation for gfx1102 LLVM AMD GPUs 

This is a guide on how to install PyTorch 2.9.0 + ROCm 6.4 in Ubuntu 24.04 for unsupported gfx1102 LLVM AMD GPUs:

- AMD Radeon RX 7600
- AMD Radeon RX 7600 XT
- AMD Radeon RX 7650 GRE
- AMD Radeon RX 7600M (Mobile)
- AMD Radeon RX 7600M XT (Mobile)
- AMD Radeon RX 7600S (Mobile)
- AMD Radeon RX 7700S (Mobile)
- AMD Radeon PRO W7500 (Workstation)
- AMD Radeon PRO W7600 (Workstation)
- AMD Radeon RX 7500 XT
## ROCm 6.4 Installation
## #1 System update + essentials

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget gpg software-properties-common
```

## #2 Add ROCm GPG key
```bash
sudo mkdir -p /etc/apt/keyrings
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null
```

Without this, APT will reject the ROCm repository. GPG Keys are stored in /etc/apt/keyrings

## #3 Add ROCm 6.4 repository
This will allow your system to download and install ROCm components via APT.
```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/6.4 noble main" | \
sudo tee /etc/apt/sources.list.d/rocm.list
```
Then pin ROCm repo (VERY IMPORTANT) to prevent Ubuntu packages from overriding ROCm ones:
```bash
echo -e 'Package: *\nPin: release o=repo.radeon.com\nPin-Priority: 600' | \
sudo tee /etc/apt/preferences.d/rocm-pin-600
```

Lastly, update package lists
```bash
sudo apt update
```

## #4 Install ROCm core stack
```bash
sudo apt install -y rocm-hip-sdk rocm-dev rocm-libs
```

## #5 Add user permissions to GPU-related groups
This allows your GPU to access AMD GPU device nodes.
```bash
sudo usermod -aG video $USER
sudo usermod -aG render $USER
```
Then reboot
```bash
reboot
```

## #6 Verify installation after reboot

```bash
/opt/rocm/bin/rocminfo | grep gfx
```
```bash
/opt/rocm/bin/rocminfo
/opt/rocm/bin/hipcc --version
```

If you see something like...

```bash
gfx1100

Name: AMD Ryzen... 
Name: gfx1100 

HIP version: 6.4.0
clang version 17.x.x
Target: x86_64-unknown-linux-gnu
```
then congrats, ROCm 6.4 is installed! 

If not then cry. 

## For next steps...
Install [miniconda](https://www.anaconda.com/docs/getting-started/miniconda/install/overview) first then create a conda environment (Python 3.11)

```bash
conda create -n amd-ai python=3.11 -y
conda activate amd-ai
```

then upgrade `pip`
```bash
pip install --upgrade pip
```


## Install PyTorch + ROCm 6.4

```bash
pip install torch==2.9.0+rocm6.4 torchvision==0.24.0+rocm6.4 torchaudio==2.9.0+rocm6.4 \
--index-url https://download.pytorch.org/whl/rocm6.4
```

## Verify Pytorch installation
```bash
python
```

```bash
import torch

print("Torch version:", torch.__version__)
print("HIP version:", torch.version.hip)
print("GPU available:", torch.cuda.is_available())
print("GPU name:", torch.cuda.get_device_name(0))
```

You should see something like this:

```bash
Torch version: 2.9.0+rocm6.4
HIP version: 6.4.x
GPU available: True
GPU name: AMD Radeon...
```
## Support

Please create an issue if there are problems in your installation.

## Acknowledgements

 - https://github.com/nktice/AMD-AI
 - https://github.com/ROCm/TheRock/blob/main/RELEASES.md#rocm-for-gfx110X-all
 - https://www.reddit.com/r/ROCm/comments/1qh4bx2/rx_7600_gfx1102_rocm_pytorch_any_way_to_make_it/
 - https://github.com/patientx/ComfyUI-Zluda/issues/170#issuecomment-2972793016
