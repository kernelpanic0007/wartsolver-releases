# Changelog

Notable changes to the public **wartsolver** release bundles, newest first.
Each version's downloads are on the [Releases](../../releases) page.

## 0.1.274 — 2026-09-11

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
