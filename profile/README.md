<p align="center">
  <img src="https://media.githubusercontent.com/media/e-jerk/logo/main/logo-animated.png" alt="e/jerk logo" width="256">
</p>

# e-jerk

> **⚠️🚨 WARNING 🚨⚠️**
>
> **This is vibe coded slop. You should never use it unless you are truly brave.**

---

GPU-accelerated Unix utilities with Zig frontends and Metal/Vulkan compute backends.

## Installation

### Homebrew (macOS/Linux)

```bash
# Install all GPU utilities at once
brew install e-jerk/utils/gpu-utils

# Or install individually from each repo's tap
brew install e-jerk/grep/grep
brew install e-jerk/sed/sed
brew install e-jerk/find/find
```

These utilities override the system commands and support GPU acceleration:
- `grep` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode
- `sed` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode
- `find` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode

Use `--cpu` to force CPU-only mode.

## Packages

### Utilities

| Package | Description | Version |
|---------|-------------|---------|
| [grep](https://github.com/e-jerk/grep) | GPU-accelerated grep with SIMD Boyer-Moore-Horspool | v0.1.0 |
| [sed](https://github.com/e-jerk/sed) | GPU-accelerated sed with parallel substitution | v0.1.0 |
| [find](https://github.com/e-jerk/find) | GPU-accelerated find with vectorized glob matching | v0.1.0 |
| [gawk](https://github.com/e-jerk/gawk) | GPU-accelerated awk with parallel field splitting | v0.1.0 |

### Libraries

| Package | Description | Version |
|---------|-------------|---------|
| [gpu](https://github.com/e-jerk/gpu) | GPU capability detection and auto-selection library | v0.1.0 |
| [shaders-common](https://github.com/e-jerk/shaders-common) | Shared SIMD shader utilities for Metal and Vulkan | v0.1.0 |

## Architecture

All tools share the `e_jerk_gpu` library for:
- Hardware capability detection (threads, memory, buffer limits)
- Performance-based backend scoring (Entry/Mid/High/Ultra tiers)
- Automatic CPU/GPU selection based on workload characteristics
- Swappable compute backend interface

### SIMD Optimizations

- **CPU**: Zig `@Vector(16, u8)` and `@Vector(32, u8)` for parallel byte operations
- **Metal**: `uchar4` vectorized operations (4-byte SIMD)
- **Vulkan**: `uvec4` vectorized operations (16-byte SIMD)

## Supported Backends

| Backend | Platform | Status |
|---------|----------|--------|
| Metal | macOS | Supported |
| Vulkan | Linux, macOS (MoltenVK) | Supported |
| CUDA | Linux, Windows | Planned |

## License

Source code: [Unlicense](https://unlicense.org/) (public domain)
Binaries: GPL-3.0-or-later (due to linked dependencies)
