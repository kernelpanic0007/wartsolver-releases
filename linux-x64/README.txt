wartsolver 0.1.274 — public bundle
================================
Requirements: Linux x86-64 (Ubuntu 24.04+ / glibc 2.38+), a working GPU driver.
  AMD:    the amdgpu driver. No ROCm install needed — the runtime is inside the binary.
  NVIDIA: the NVIDIA driver. (This bundle: CUDA embedded = yes (prebuilt plugin embedded))

Developer fee: none in this build. A fee may be added in a future version;
that will be called out at https://txbitmining.com/wartsolver before it ships.
Separate from the TXBit pool's 1% fee.

Community hashrate board: this miner sends GPU/CPU config (hardware, clocks,
janus/power) to https://warthog.txbitmining.com so real pairings can be listed
for everyone else. Same machine updates one row. Wallet and pool password are
not sent.

ONE FILE. Run it directly:
  ./wartsolver -p stratum+tcp://POOL:PORT -w YOUR_WALLET -r WORKER_NAME
  ./wartsolver --list-gpus    # what this machine's GPUs look like to the solver
  ./wartsolver --help
On hosts with <=16 hardware threads the solver defaults to a blocking GPU wait
(WART_GPU_BLOCKING_SYNC=0/1 overrides).

The event log line "[gpu] ... backend loaded: ..." names exactly which backend
won; the TUI tags each device [hip]/[cuda]/[ocl].
