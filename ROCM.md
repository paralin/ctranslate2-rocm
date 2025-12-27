# CTranslate2 ROCm Fork

This is a fork of [sssshhhhhh/CTranslate2](https://github.com/sssshhhhhh/CTranslate2) (itself a fork of [OpenNMT/CTranslate2](https://github.com/OpenNMT/CTranslate2)) with AMD ROCm/HIP support.

## Tested Configuration

- **OS**: Debian Sid (trixie/forky)
- **GPU**: AMD Radeon RX 7700 XT (gfx1101, 12GB VRAM)
- **ROCm**: 7.1.1
- **Python**: 3.10

## Build Instructions (Debian Sid)

### Prerequisites

```bash
sudo apt install cmake build-essential libopenblas-dev git
```

ROCm must be installed at `/opt/rocm`. See [ROCm installation docs](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/).

### Clone and Build

```bash
git clone --recurse-submodules https://github.com/paralin/ctranslate2-rocm.git ~/ctranslate2
cd ~/ctranslate2
git checkout rocm

mkdir -p build && cd build

export HSA_OVERRIDE_GFX_VERSION=11.0.1  # for gfx1101
export AMDGPU_TARGETS=gfx1101
export ROCM_PATH=/opt/rocm

cmake .. \
  -DWITH_HIP=ON \
  -DWITH_MKL=OFF \
  -DWITH_OPENBLAS=ON \
  -DCMAKE_HIP_ARCHITECTURES=gfx1101 \
  -DCMAKE_BUILD_TYPE=Release \
  -DOPENMP_RUNTIME=COMP \
  -DCMAKE_HIP_COMPILER=/opt/rocm/lib/llvm/bin/clang++ \
  -DCMAKE_CXX_COMPILER=/opt/rocm/lib/llvm/bin/clang++ \
  -DCMAKE_C_COMPILER=/opt/rocm/lib/llvm/bin/clang \
  -DCMAKE_PREFIX_PATH=/opt/rocm \
  -DBUILD_CLI=OFF

make -j$(nproc)
sudo make install
```

### Install Python Bindings

```bash
export CTRANSLATE2_ROOT=/usr/local
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

# If using uv/whisperx venv:
cd ~/whisperx
uv pip install pybind11 ~/ctranslate2/python
```

### Verify Installation

```bash
export LD_LIBRARY_PATH=/usr/local/lib:/opt/rocm/lib:/opt/rocm/lib/llvm/lib:$LD_LIBRARY_PATH
export HSA_OVERRIDE_GFX_VERSION=11.0.1
export ROCM_PATH=/opt/rocm
export HIP_VISIBLE_DEVICES=0

python -c "import ctranslate2; print(ctranslate2.__version__); print(ctranslate2.get_supported_compute_types(cuda))"
```

Expected output:
```
4.6.2
{int8_float16, int8_bfloat16, bfloat16, int8_float32, int8, float16, float32}
```

## GPU Architecture

Set `CMAKE_HIP_ARCHITECTURES` based on your GPU:

| GPU | Architecture |
|-----|--------------|
| RX 7900 XTX/XT | gfx1100 |
| RX 7800 XT | gfx1101 |
| RX 7700 XT | gfx1101 |
| RX 7600 | gfx1102 |
| RX 6900/6800/6700 | gfx1030 |
| RX 6600 | gfx1032 |

## Upstream

- Original: [OpenNMT/CTranslate2](https://github.com/OpenNMT/CTranslate2)
- ROCm fork: [sssshhhhhh/CTranslate2](https://github.com/sssshhhhhh/CTranslate2)
