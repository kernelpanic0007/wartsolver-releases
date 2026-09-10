wartsolver 0.1.274 - Windows x64
================================

Unzip and run. There is no installer and nothing is written outside your user
profile.

  wartsolver.exe        the miner (console). BOTH GPU backends are built in -
                        NVIDIA (CUDA) and AMD (HIP) - so this one file mines on
                        either vendor with no extra DLL to keep beside it.
  wartsolver-gui.exe    optional window around it, with per-GPU overclocking.
  wart_sensors.exe      OPTIONAL CPU temperature/power reader (see below).
  wart_sensors.exe.config

Keep wartsolver.exe and wartsolver-gui.exe together. wart_sensors.exe is
optional and stands alone.

Developer fee: none in this build. A fee may be added in a future version;
that will be called out at https://txbitmining.com/wartsolver before it ships.
Separate from the TXBit pool's 1% fee.

Community hashrate board: this miner posts the rig to
https://warthog.txbitmining.com so real pairings can be listed. Same machine
updates one row. Wallet and pool password are not sent.
Sent: CPU model/threads/clock/voltage; GPU model/core+mem clocks and offsets/
power limit; janus, GPU SHA, CPU Verus MH/s; wall/GPU/CPU watts when readable;
miner version, OS, sample window, optional tweak notes.


REQUIREMENTS
------------

  Windows 10/11 x64.

  A GPU driver:
    NVIDIA  - any recent driver. The CUDA backend is built into
              wartsolver.exe; no CUDA toolkit needed.
    AMD     - AMD Software: Adrenalin Edition. The AMD (HIP) backend is built
              into wartsolver.exe and uses the HIP runtime Adrenalin installs
              (amdhip64_7.dll); no ROCm or HIP SDK needed.

  Microsoft Visual C++ 2015-2022 Redistributable (x64). Most systems already
  have it. Without it the AMD backend cannot initialise and the miner falls
  back to a slower OpenCL path on AMD cards - it will still mine, just slower.
  NVIDIA is unaffected. See TROUBLESHOOTING.

  .NET Framework 4.8 - only for the optional wart_sensors.exe. Already part of
  Windows 10/11.


QUICK START
-----------

  wartsolver.exe -p stratum+tcp://HOST:PORT -w YOUR_WALLET -r RIGNAME

  -t N            CPU mining threads (default: cores - 2)
  --status-api 1  serve stats on http://127.0.0.1:9500/status
  --list-gpus     show what was detected, then exit
  -h              full help

Or run wartsolver-gui.exe, fill in pool and wallet, press Start. The GUI asks
for Administrator because the overclock controls require it. Settings are
remembered in %APPDATA%\wartsolver\gui.ini.


CHECK YOUR GPU WAS DETECTED
---------------------------

  wartsolver.exe --list-gpus

  NVIDIA good:  [0] cuda:0  NVIDIA GeForce RTX ...
  AMD good:     [0] hip:0   AMD Radeon RX ... (gfx1100) ...
  AMD slow:     [0] hip:0   gfx1100 [AMD-APP ...] (opencl) ...

On AMD, "(opencl)" means the built-in HIP backend could not initialise and you
are on the slower fallback - almost always the missing Visual C++
redistributable above.


OVERCLOCKING
------------

Needs Administrator. Everything is restored to stock when the miner exits,
including on Ctrl-C and on closing the window.

  NVIDIA:  --cclk MHZ   lock core clock       --mclk MHZ   lock memory clock
           --pl W       power limit, watts    --coff/--moff MHZ  clock offsets
           --fan PCT    fixed fan duty

  AMD:     --amd-sclk MHZ   max core clock (a ceiling, not a lock)
           --amd-mclk MHZ   max memory clock
           --amd-pl N       power limit (RDNA: PERCENT of default)
           --amd-mv MV      voltage offset, negative to undervolt
           --amd-vcap MV    absolute voltage ceiling
           --amd-fan PCT    fixed fan duty

  --oc-probe    what each card supports, WITH ITS RANGES (run this first)
  --oc-reset    put every card back to stock and exit (recovery)

Ranges differ by card and driver - do not assume a knob exists or has the range
you expect. Run --oc-probe before choosing values.


TUNING NOTES
------------

Treat as a method, not numbers to copy.

  Voltage is the biggest efficiency lever on AMD. Capping voltage lets the card
  hold a higher clock in the same power budget.

  CPU threads can starve the GPU. Every CPU thread competes with the host work
  the GPU backend needs. If the GPU sits at low utilisation while the CPU is
  pegged, REDUCE -t and compare.

  Short tests rank settings; they do not prove them. Power rises as the card
  heats, so a 3-minute run understates steady-state watts. Let a candidate run
  for hours before trusting it. Read the pool line - the blue "avg janus paid"
  is what actually pays, and it needs a few hundred shares to settle.

  An overly aggressive undervolt fails SILENTLY, as rejected shares or a hang.
  Watch the reject counter for the first hour after any voltage change.


CPU TEMPERATURE / POWER (optional)
----------------------------------

wart_sensors.exe supplies the CPU temperature and wattage on the panel. Windows
has no user-mode way to read those, so it loads a kernel driver (WinRing0), and:

  - It only works when the miner runs ELEVATED. Unelevated, the CPU row shows
    "--" and nothing else is affected.
  - Windows Defender may quarantine WinRing0 (a known vulnerable driver). If the
    CPU row shows "--" while elevated, that is why. Either accept it - mining is
    unaffected - or add a Defender exclusion for this folder. This is expected
    and is why the sensor ships as a SEPARATE optional file, not inside the
    miner: wartsolver.exe itself contains no such driver.

Delete wart_sensors.exe if you do not want it. The miner works without it.


TROUBLESHOOTING
---------------

"AMD backend: The specified module could not be found. (error 126)"
  Error 126 usually means a DEPENDENCY is missing - most often the Visual C++
  redistributable, or an Adrenalin version that does not ship amdhip64_7.dll.
  Install the redistributable. The miner continues on OpenCL meanwhile.

"no GPU driver visible yet - waiting up to 30s"
  Driver not detected yet. If it appears, mining proceeds. WART_GPU_WAIT=0 skips
  the wait.

GPU found but very low hashrate, GPU utilisation low
  Too many CPU threads. Try -t with about half your logical processors.

Overclock settings did not apply
  Run as Administrator. Failures are reported per device - read the [oc] lines.

A previous run was killed and left the card clocked oddly
  wartsolver.exe --oc-reset

ENVIRONMENT VARIABLES

  WART_ENABLE_OPENCL=1   allow the OpenCL fallback for AMD (the GUI sets this)
  WART_NO_OPENCL=1       never use the OpenCL fallback
  WART_GPU_WAIT=N        seconds to wait for a GPU driver at startup (0 = off)


NOTE

This build validates a licence with txbitmining.com at startup and periodically,
and will not mine without it.
