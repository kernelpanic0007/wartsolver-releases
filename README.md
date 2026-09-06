# wartsolver-releases

Public-facing release bundles for **wartsolver** (Warthog / janushash miner).

> Private until launch. Source lives in a separate private repo.

## Download

Grab the latest zip from the [**Releases**](../../releases) page, unzip, and run — no installer.

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
