# wartsolver-releases

Public-facing release bundles for **wartsolver** (Warthog / janushash miner).

> Source lives in a separate private repo; the release bundles here are the supported public builds.

## Download

Grab the latest bundle from the [**Releases**](../../releases) page, unpack, and run — no installer.

## Windows x64 — contents of the zip

| file | |
|---|---|
| `wartsolver.exe` | the miner. **Both GPU backends built in** — NVIDIA (CUDA, fat: sm_75→sm_120) and AMD (HIP, embedded). One file, either vendor, no extra DLL. |
| `wartsolver-gui.exe` | optional GUI with per-GPU overclocking |
| `wart_sensors.exe` (+ `.config`) | **optional** CPU temp/power reader — separate on purpose (loads a kernel driver Defender may flag; the miner itself does not) |
| `README.txt` | full user guide |

**Version:** 0.1.274 · **Requires:** Windows 10/11 x64, a GPU driver (NVIDIA any recent / AMD Adrenalin), and the Microsoft Visual C++ 2015–2022 Redistributable (x64).

Quick start:
```
wartsolver.exe -p stratum+tcp://HOST:PORT -w YOUR_WALLET -r RIGNAME
```
or run `wartsolver-gui.exe`, enter pool + wallet, press Start.

This build validates a licence with txbitmining.com and will not mine without it.

## Linux x64 — contents of the tar.gz

| file | |
|---|---|
| `wartsolver` | the miner. **One file, both GPU backends embedded** — AMD (HIP, ROCm runtime bundled inside: no ROCm install needed) and NVIDIA (CUDA, prebuilt fat plugin) — plus builtin OpenCL fallback and static libstdc++. |
| `README.txt` | user guide |

**Version:** 0.1.274 · **Requires:** Linux x86-64, Ubuntu 24.04+ (glibc 2.38+), and a working GPU driver — AMD: the `amdgpu` kernel driver (no ROCm needed); NVIDIA: the NVIDIA driver.

Quick start:
```bash
tar xf wartsolver-0.1.274-linux-x64.tar.gz
cd wartsolver-0.1.274-linux-x64
./wartsolver -p stratum+tcp://HOST:PORT -w YOUR_WALLET -r RIGNAME
```

Useful first commands:
```bash
./wartsolver --list-gpus   # what this machine's GPUs look like to the solver
./wartsolver --tune        # auto-tune GPU batch/block size, then exit
./wartsolver --help        # full flag reference (also listed below)
```

Verify your download:
```bash
sha256sum -c wartsolver-0.1.274-linux-x64.tar.gz.sha256
```

The event log line `[gpu] ... backend loaded: ...` names exactly which backend won; the TUI tags each device `[hip]`/`[cuda]`/`[ocl]`. On hosts with ≤16 hardware threads the solver defaults to a blocking GPU wait (`WART_GPU_BLOCKING_SYNC=0/1` overrides).

Like the Windows build, this build validates a licence with txbitmining.com and will not mine without it.

## HiveOS

Use the dedicated HiveOS package `wartsolver-0.1.274.tar.gz` (NOT the `-linux-x64` one) — it bundles the same public Linux binary plus the `h-config.sh` / `h-run.sh` / `h-stats.sh` wrappers HiveOS needs, so hashrate, per-GPU temps/fans, and accepted/rejected shares all show up on the HiveOS dashboard.

**Requires a HiveOS image based on Ubuntu 22.04 or newer** (glibc ≥ 2.35). No ROCm install is needed on AMD — the runtime is embedded in the binary.

### Install via flight sheet (recommended)

1. Create a flight sheet and choose **Custom** as the miner, then open **Setup Miner Config** and fill in:

| field | value |
|---|---|
| Miner name | `wartsolver` (auto-filled from the URL) |
| Installation URL | `https://github.com/kernelpanic0007/wartsolver-releases/releases/download/v0.1.274/wartsolver-0.1.274.tar.gz` |
| Hash algorithm | leave as `----` |
| Wallet and worker template | `%WAL%` — the HiveOS worker name is appended automatically as `WALLET.<worker>`; use `%WAL%.%WORKER_NAME%` to set it explicitly |
| Pool URL | `stratum+tcp://HOST:PORT` |
| Extra config arguments | optional wartsolver flags, passed verbatim — e.g. `-t 16 --nvidia 0` (see the flag reference below) |

2. Apply the flight sheet. HiveOS downloads the package, installs it under `/hive/miners/custom/wartsolver/`, and starts mining.

### Install from the rig shell (alternative)

```bash
custom-get https://github.com/kernelpanic0007/wartsolver-releases/releases/download/v0.1.274/wartsolver-0.1.274.tar.gz
miner start
```

Notes:
- The wrapper starts the miner with `--status-api 1 --status-port 9500`; the HiveOS agent reads `http://127.0.0.1:9500/status` for the dashboard stats (port 9500 was chosen to not collide with bzminer's 4014).
- `miner log` shows the live miner output; the log file is `/var/log/miner/wartsolver/wartsolver.log`.

## Flags

Same flags on Linux and Windows (the NVIDIA overclocking group needs root/Administrator).

### Connection

| flag | purpose |
|---|---|
| `-n, --node URL` | Node RPC URL for solo mining (default: `http://127.0.0.1:3001`) |
| `-p, --stratum URL` | Mine to a **pool** over stratum instead of solo HTTP. Accepts `host:port` or `stratum+tcp://host:port`. |
| `-w, --wallet ADDR` | Wallet address (required; also the stratum username). On a pool the worker name is appended as `ADDR.<worker>`; pass `ADDR.<name>` yourself to choose it explicitly — a wallet that already has a suffix is never rewritten. |
| `-r, --worker NAME` | Worker name (default: the machine's hostname) |

### GPU selection

| flag | purpose |
|---|---|
| `--gpus SPEC` | Devices to use: `all` \| `cuda:0,hip:1` \| `0,2` (default: `all`) |
| `-d, --device INT` | GPU index (default 0; legacy alias for `--gpus`) |
| `--list-gpus` | Print every GPU found, with its backend, and exit |
| `--amd 0\|1` | Use AMD GPUs (default 1) |
| `--nvidia 0\|1` | Use NVIDIA GPUs (default 1). Both default on, so out of the box the solver uses whatever is in the box; composes with `--gpus`. |

### Performance / tuning

| flag | purpose |
|---|---|
| `-t, --threads INT` | CPU Verus threads (default: auto = cores − 2, reserving one for the feeder and one for the system) |
| `--pool-threads S` | How threads split across GPUs on a multi-GPU rig: `auto` (default, weights each pool by a short measured sha burst on its card), `even` (splits equally), or explicit per-slot counts, e.g. `126,126` |
| `-b, --batch INT` | GPU batch 2^N (default: auto-sized from the GPU's SM count; `0` forces auto) |
| `-s, --block-size INT` | CUDA threads per block (default: auto from device) |
| `--tune` | Auto-tune GPU batch & block size for max SHA256t rate, then exit |
| `--feed-margin F` | How far above worker capacity the GPU aims (default 1.10). Trades wasted work for risk of starving the workers — measure, don't guess. |
| `--feeder-shared 0\|1` | Let a feeder share its CPU with a verus worker rather than own it (default 0). Worth one worker thread per GPU; measure it — the feeder runs at SCHED_FIFO. |
| `--numa 0\|1` | Pin each GPU's workers to the NUMA node its card is on (default 1). `0` carves the machine proportionally instead. |
| `--v0-cut MODE` | `host` (default) trims surplus candidates during harvest; `off` disables the trim (a measurement mode, not a production one) |
| `--recalibrate` | Ignore any cached calibration, measure fresh (~2 min), and save. The cache is keyed to GPU, CPU, thread count and batch size — use this after an overclock or driver change, which the key cannot see. |
| `--cal-file PATH` | Calibration cache location (default: `wartsolver-calibration.json` beside the binary) |

### Output / monitoring

| flag | purpose |
|---|---|
| `-v, --verbose` | Per-interval diagnostics to stderr (pass rate, candidates/s, collect ms, D2H MB/s, overflows, CPU recv/drop/idle) and every event (shares, blocks, rejects) mirrored to stderr |
| `-L, --log-file PATH` | Append every event and a once-a-minute totals line to PATH, independent of stdout/stderr. `grep 'BLOCK FOUND' PATH` answers "did I find one?"; `grep '\[stat\]' PATH` shows session totals. |
| `--status-api 0\|1` | Serve `GET /status` over HTTP (default 0). Binds 127.0.0.1 only. |
| `--status-port N` | Port for `--status-api` (default 9500) |
| `--no-color` | Plain text, no ANSI colour (equivalent to setting `NO_COLOR`). Colour is on by default only when stdout is a terminal. |
| `--color` | Emit ANSI colour even when stdout is not a terminal — for wrappers that render the codes themselves. Cursor control stays off, so logs remain readable. |

### NVIDIA overclocking (needs root / Administrator; settings are restored on exit)

| flag | purpose |
|---|---|
| `--cclk MHZ` | Lock NVIDIA core clock via NVML (like `nvidia-smi -lgc`); applied to every NVIDIA GPU, reset on exit |
| `--mclk MHZ` | Lock NVIDIA memory clock (`nvidia-smi -lmc`); same rules |
| `--pl W` | NVIDIA power limit in watts; default restored on exit |
| `--coff MHZ` | NVIDIA core clock offset (±), zeroed on exit |
| `--moff MHZ` | NVIDIA memory clock offset (±), zeroed on exit |
| `--fan PCT` | Fix NVIDIA fans at PCT% duty; auto curve restored on exit |
| `--oc-reset` | Reset every NVIDIA GPU to stock clocks/power and exit — the recovery for locks a killed run left behind |

### Misc

| flag | purpose |
|---|---|
| `-V, --version` | Print the exact build (version, commit, build time) and exit |
| `-h, --help` | Show the built-in help |
