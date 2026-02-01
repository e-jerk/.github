<p align="center">
  <img src="https://media.githubusercontent.com/media/e-jerk/logo/main/logo-animated.png" alt="e/jerk logo" width="256">
</p>

# e-jerk

> **Warning**
>
> **This is vibe coded slop. You should never use it unless you are truly brave.**

---

GPU-accelerated Unix utilities with Zig frontends and Metal/Vulkan compute backends.

## Build Variants

Each utility is available in **two variants**:

| Variant | Description | Vulkan on macOS | `--gnu` flag |
|---------|-------------|-----------------|--------------|
| **pure** | Zig + SIMD + GPU only. Self-contained, no external dependencies. Source code is public domain (Unlicense). | No | Not available |
| **gnu** | Includes GNU utilities + Vulkan via MoltenVK for full POSIX compliance. Binaries are GPL-3.0-or-later. | Yes | Works |

## Installation

### Homebrew (macOS/Linux)

```bash
# Install pure builds (default, self-contained)
brew install e-jerk/utils/gpu-utils

# Install gnu builds (with GNU fallback)
brew install e-jerk/utils/gpu-utils-gnu

# Or install individually
brew install e-jerk/grep/grep       # pure
brew install e-jerk/grep/grep-gnu   # gnu
```

### Backend Selection

All utilities support:
- `--auto` — Automatically select optimal backend (default)
- `--gpu` — Force GPU (Metal on macOS, Vulkan on Linux)
- `--cpu` — Force CPU backend (SIMD-optimized)
- `--gnu` — Force GNU implementation (gnu build only)
- `--metal` — Force Metal backend (macOS only)
- `--vulkan` — Force Vulkan backend

Use `-V` or `--verbose` to see timing and backend information.

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

Each tool includes native implementations with GPU and SIMD optimization:

| Tool | GPU+SIMD Features | GPU VM Features | GNU Fallback (gnu build) |
|------|-------------------|-----------------|--------------------------|
| **grep** | `-e` `-E` `-G` `-P` `-F` `-i` `-w` `-v` `-n` `-c` `-l` `-L` `-q` `-o` `-A/-B/-C` `-r` `--color` | — | — |
| **sed** | `s///gi` `/d` `/p` `-i` `-n` `-E` | `y///` `-e` line addressing | Backrefs, hold space, branching |
| **find** | `-name` `-iname` `-path` `-ipath` `-regex` `-iregex` | — | `-exec` `-delete` `-newer` |
| **gawk** | `/pattern/` `{print $N}` `-F` `-i` `-v` `length` `substr` `index` `toupper` `tolower` `NR` `NF` `gsub` regex | BEGIN/END, variables, if/while/for, math functions | arrays, printf |

**Optimization key**: `[GPU+SIMD]` means the feature runs on GPU compute shaders (Metal/Vulkan) with SIMD-optimized CPU fallback. `[GPU VM]` means features are compiled to bytecode and executed on a GPU virtual machine. Use `--help` on any tool to see optimization annotations for each flag.

Use `--gnu` flag (gnu build only) for unsupported features to fall back to GNU implementation.

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
- **GPU Bytecode VM**: Complex AWK programs compile to bytecode and execute on GPU
  - Supports: variables, arithmetic, comparisons, control flow (if/while/for), math functions
  - Each GPU thread processes one input line independently
- Native built-in functions: `length()`, `substr()`, `index()`, `toupper()`, `tolower()`, `sin()`, `cos()`, `sqrt()`, `log()`, `exp()`, `int()`
- Native special variables: `NR`, `NF`
- Native GPU regex: Thompson NFA execution on Metal and Vulkan
- Full backend parity (CPU, Metal, Vulkan produce identical results)

## License

Source code: [Unlicense](https://unlicense.org/) (public domain)
Binaries: GPL-3.0-or-later (due to linked dependencies)
