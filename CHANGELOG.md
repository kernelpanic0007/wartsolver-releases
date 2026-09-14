# Changelog

Notable changes to the public **wartsolver** release bundles, newest first.
Each version's downloads are on the [Releases](../../releases) page.

## 0.1.293 — 2026-09-14 · Windows x64 + Linux x64

Windows catches up to Linux again; both platforms ship the same build. No GPU,
algorithm or protocol change.

### Fixed
- **The miner now reports its real version to the pool.** `mining.subscribe` sent a
  hardcoded `wartsolver/0.1` regardless of the build, so pool-side statistics showed
  every version as 0.1. It now sends the same string `--version` prints.
- Internal reliability fix in the background hashrate-board reporter — no effect
  on mining, and nothing sent that was not sent before.

### Changed
- Linux: carries the Zen tuning from 0.1.289 (see below); no further performance
  change.

### Notes
- Windows: same GPU backends, overclock controls, and flags as 0.1.289.
  Single-file miner — NVIDIA (CUDA, sm_75 -> sm_120) and AMD (HIP) built into
  `wartsolver.exe`; no separate DLL.
- The Windows zip now ships a `.sha256` alongside it too, matching the Linux
  and HiveOS assets.
- Smart App Control / SmartScreen may still block the unsigned Windows exe; see
  the bundled `README.txt`. Code-signing is planned.

## 0.1.289 — 2026-09-14 · Windows x64 + Linux x64

Windows catches up to Linux after sitting at 0.1.278; both platforms ship the
same build from here. No GPU, algorithm or protocol change.

### Added (both platforms; first appeared in 0.1.284 on Linux)
- Pool rejections print the pool's own reason inline on the `REJECTED` line.
- New janus ideal/effective gap metric in the TUI and the `--status-api` JSON,
  showing what the delivery pipeline achieves against its own ceiling.

### Changed (Linux only)
- **Verus is faster on every AMD CPU.** The build now tunes for Zen rather than a
  generic x86 target. Generic tuning suppressed a BMI1 bit-extract instruction that
  every Zen part has had since 2017 (it is slow on some Intel chips); with it
  enabled, roughly 270 shift-and-mask pairs in the hot verus loop collapse to single
  instructions. Measured +1.6% to +5.2% total hashrate across nine rigs.
- The CPU instruction-set floor is unchanged — AVX2/BMI2/FMA plus AES-NI and PCLMUL —
  so this binary runs anywhere 0.1.284 ran (Haswell+, Zen+). No GPU, algorithm or
  protocol change.
- The public binary is now built with the same toolchain and configuration as the
  author's own fleet binary; the licence check is the only difference between them.

### Notes
- Windows: same GPU backends, overclock controls, and flags as 0.1.278.
  Single-file miner — NVIDIA (CUDA, sm_75 -> sm_120) and AMD (HIP) built into
  `wartsolver.exe`; no separate DLL.
- Smart App Control / SmartScreen may still block the unsigned Windows exe; see
  the bundled `README.txt`. Code-signing is planned.

## 0.1.284 — 2026-09-13 · Linux x64

Linux-only release; the Windows bundle stays at 0.1.278 for now.

### Fixed
- **HiveOS: accepted/rejected share counters no longer sit at zero.** The
  bundled `h-config.sh` now passes the event-log flag the stats wrapper reads,
  so the dashboard's `ar` counts track the miner instead of staying frozen.
- **Verus throughput protected against a code-layout regression.** Code added
  since 0.1.278 had shifted every hot verus function off its cache-line phase —
  a hazard that has cost 3–6% verus on this codebase before. The alignment pad
  was re-tuned per toolchain and the layout gate passes again. No algorithm or
  protocol change; the rest of the mining code is unchanged from 0.1.278.

### Added
- Pool rejections print the pool's own reason inline on the `REJECTED` line.
- New janus ideal/effective gap metric in the TUI and the `--status-api` JSON,
  showing what the delivery pipeline achieves against its own ceiling.

## 0.1.278 — 2026-09-11 · Windows x64 + Linux x64

Maintenance release. No changes to mining behavior, flags, or performance.

### Changed
- Internal reliability fixes in the background hashrate-board reporter — no
  effect on mining, and nothing sent that was not sent before.

### Notes
- Same GPU backends, overclock controls, and flags as 0.1.277.
- Linux x64 + HiveOS bundles ship under this tag too; their mining code is
  byte-identical to 0.1.277 (this version's fixes are Windows-side).
- Smart App Control / SmartScreen may still block the unsigned Windows exe; see
  the bundled `README.txt`. Code-signing is planned.

## 0.1.277 — 2026-09-11 · Windows x64

### Fixed
- **Integrated GPUs are no longer mined by default.** On a Ryzen APU the HIP
  backend was enumerating the integrated Radeon (e.g. 610M) and mining it
  alongside the real discrete card — a trickle of hashrate that stole CPU and the
  APU's shared power/thermal budget. iGPUs are now skipped
  (`WART_ALLOW_IGPU=1` to include one).
- **GUI no longer cross-applies overclocks after a GPU swap.** OC was kept per
  slot, so swapping cards (e.g. an AMD card out, an NVIDIA card in) could land an
  AMD voltage cap on an NVIDIA memory-offset box. Each row now remembers its card
  and resets to stock if a different one is present.
- **Thread count is clamped to the machine** — a `gui.ini` carried from a bigger
  box no longer shows e.g. "126 of 32".

### Added / changed
- **Single-file Windows miner** — NVIDIA CUDA (fat, sm_75 → sm_120) and AMD HIP
  are both built into `wartsolver.exe`; no separate DLL.
- **Single-file Linux miner** — AMD HIP (ROCm runtime bundled) + NVIDIA CUDA
  embedded in one `wartsolver` binary; generic Linux and HiveOS bundles.
- **Clearer status panel** — the pool line shows `avg janus paid` (the
  session-average the pool credits you) in blue, on one line.

### Notes
- On Windows, `wart_sensors.exe` ships **separately** and is optional (CPU
  temperature/power). It loads a kernel driver Defender may flag — the miner
  itself does not.
- Smart App Control / SmartScreen may block the unsigned Windows exe; code-signing
  is planned to remove that.

<!-- Next release — copy this block above:
## X.Y.Z — YYYY-MM-DD
### Fixed
### Added / changed
### Notes
-->
