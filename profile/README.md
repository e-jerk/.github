<p align="center">
  <img src="https://media.githubusercontent.com/media/e-jerk/logo/main/logo-animated.png" alt="e/jerk logo" width="256">
</p>

# e-jerk

> **Warning**
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
brew install e-jerk/gawk/gawk
```

These utilities override the system commands and support GPU acceleration:
- `grep` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode
- `sed` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode
- `find` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode
- `gawk` - Use `--gpu`, `--metal`, or `--vulkan` for GPU mode

Use `--cpu` to force CPU-only mode, or `--gnu` for full GNU compatibility.

## Packages

### Utilities

| Package | Description | Tests | Version |
|---------|-------------|-------|---------|
| [grep](https://github.com/e-jerk/grep) | GPU-accelerated grep with context lines, recursive search, color output | 42/42 | v0.1.0 |
| [sed](https://github.com/e-jerk/sed) | GPU-accelerated sed with multiple expressions, line addressing | 37/37 | v0.1.0 |
| [find](https://github.com/e-jerk/find) | GPU-accelerated find with size/time filters, prune support | 36/36 | v0.1.0 |
| [gawk](https://github.com/e-jerk/gawk) | GPU-accelerated awk with built-in functions, NR/NF variables | 32/32 | v0.1.0 |

**Total: 147 GNU compatibility tests passing**

### GNU Feature Parity Summary

Each tool includes native implementations of common features, with GNU-compatible fallback for advanced features:

| Tool | Native Features | GPU-Accelerated | GNU Fallback |
|------|----------------|-----------------|--------------|
| **grep** | `-i` `-w` `-v` `-F` `-n` `-c` `-l` `-L` `-q` `-o` `-A/-B/-C` `-r` `--color` | Pattern matching | `-E` `-P` regex |
| **sed** | `s///gi` `/d` `/p` `y///` `&` `-i` `-n` `-e` line addressing | Substitution | Backrefs, hold space, branching |
| **find** | `-name` `-iname` `-path` `-type` `-maxdepth` `-not` `-empty` `-size` `-mtime` `-prune` | Glob matching | `-exec` `-delete` `-regex` |
| **gawk** | Pattern matching, fields, `-F` `gsub` `length` `substr` `index` `toupper` `tolower` `NR` `NF` | Pattern + fields | Full AWK language |

Use `--gnu` flag for any unsupported feature to fall back to GNU implementation.

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

### Performance Highlights

| Tool | Pattern Type | CPU | GPU | Speedup |
|------|-------------|-----|-----|---------|
| grep | Single char | 128 MB/s | 2.2 GB/s | **17x** |
| sed | Single char | 177 MB/s | 2.8 GB/s | **16x** |
| grep | Common word | 335 MB/s | 2.4 GB/s | **7x** |
| find | Case-insensitive | 3.5M paths/s | 15M paths/s | **4x** |
| gawk | Pattern match | 357 MB/s | 476 MB/s | **1.3x** |

*Results measured on Apple M1 Max.*

## Supported Backends

| Backend | Platform | Status |
|---------|----------|--------|
| Metal | macOS | Supported |
| Vulkan | Linux, macOS (MoltenVK) | Supported |
| CUDA | Linux, Windows | Planned |

## Building from Source

Each tool uses the same build pattern:

```bash
# Build with optimizations
zig build -Doptimize=ReleaseFast

# Run tests
zig build test      # Unit tests
zig build smoke     # Integration tests (GPU verification)
zig build bench     # Benchmarks
bash gnu-tests.sh   # GNU compatibility tests
```

Requirements:
- Zig 0.15.2+
- glslc (for Vulkan shader compilation)
- macOS Metal or Vulkan runtime

## Recent Changes

### grep
- Native `-A/-B/-C` context lines with proper group separators
- Native `-r` recursive search with combined options
- Native `--color` output with ANSI highlighting

### sed
- Native `-e` multiple expressions
- Native line addressing (`2s/...`, `2,4s/...`, `$s/...`)

### find
- Native `-size [+-]N[ckMG]` file size filtering
- Native `-mtime/-atime/-ctime [+-]N` time filtering
- Native `-prune PATTERN` directory pruning

### gawk
- Native built-in functions: `length()`, `substr()`, `index()`, `toupper()`, `tolower()`
- Native special variables: `NR`, `NF`
- Full backend parity (CPU, Metal, Vulkan produce identical results)

## License

Source code: [Unlicense](https://unlicense.org/) (public domain)
Binaries: GPL-3.0-or-later (due to linked dependencies)
