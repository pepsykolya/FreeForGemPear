# CMFD (Common Foundry) in FreeForGemPear — notes

Since 1.6.0 FreeForGemPear also mines **Common Foundry (CMFD, ForgeMatrix V4)**. The full user guide (pools, HiveOS,
Windows, options, troubleshooting) is in [MANUAL.md, section 13](../MANUAL.md#13-common-foundry-cmfd-new-in-160); AMD
Radeon on Windows is in [section 14](../MANUAL.md#14-amd-radeon-windows-new-in-160). This page sums up how CMFD mode
behaves and what each release changed, with the measured results.

## How CMFD mode works for you

* **Engine.** The CMFD work is done by **FreeForgeMiner** (MIT, <https://github.com/pepsykolya/freeforgeminer>), which ships
  in the `cmfd` folder of the package together with its licence notices (in the package). The FreeForGemPear License does
  not apply to the `cmfd` folder.
* **Coin by pool URL.** A `cmfd+tls://IPv4:PORT?pin=<64 hex>` pool URL selects CMFD (`--algo cmfd|pearl` forces it;
  `--algo pearl` with such a URL is an error). The wallet is a `cmfd1...` address (Common Foundry wallet 1.0.16+) or the
  older 64-hex form; test-network `tcmfd1...` addresses are refused.
* **One engine per GPU.** FreeForGemPear prepares the model, starts the engine on every mining GPU, shows the engine lines
  with a `[GPUn]` prefix and prints its own stats line every 10 s:
  `[ 290s] GPU0 37.04 (190 W, 701390 FW/kWh) | total 37.04 FW/s | … | user sub 84 acc 80 rej 0 | stale 4 | height 25089`.
  An engine that exits is restarted after 15/30/60/120/300 s; after 5 failed starts in a row (each under 2 minutes) that GPU
  is left alone (no reconnect storm — pools ban IPs for that); if every GPU failed, the miner exits with an error. The log
  gives hints for "out of memory" and for GPU faults (illegal memory access / `INTEGRITY | ERROR` → remove the offset /
  undervolt). Ctrl-C / SIGTERM stops the engines cleanly within about 10 s; a second Ctrl-C stops them at once.
* **Shared with Pearl:** `--pool/--wallet/--worker/--devices/--minutes`, the clock locks (`--lock-cclock/--lock-mclock/
  --moffset`, applied before the engines start and undone on exit), the power reading in the stats line, the HiveOS package
  and the Windows start files. CMFD options: `--cmfd-mode speed|eco`, `--cmfd-batch N`, `--cmfd-model DIR`,
  `--cmfd-dir DIR`, `--per-gpu-workers`, `--cmfd-skip-prepare`; environment `FFGP_CMFD_DIR`, `FFGP_CMFD_MODEL`,
  `FFGP_CMFD_WORK`.
* **Model.** 6.4 GB, downloaded once and verified at each start. Default folder `/hive/miners/custom/cmfd-model` on HiveOS
  (shared with FreeForgeMiner: one download per rig), else `cmfd-model` next to the binary. The engine's work folder
  (`FFGP_CMFD_WORK`) is `/hive/miners/custom/freeforgempear-data/cmfd-work` on HiveOS (not `/hive-config`, which is flash),
  else `cmfd-work` next to the binary.
* **Dev fee: 1 %**, inside the engine: 36 s per hour per GPU, random slot, on the pool you chose, worker `ffm-fee`. The Pearl
  fee (`ffgp-devfee`) does not run in CMFD mode, so the total stays 1 %.
* **HiveOS:** the coin follows the flight-sheet pool URL; set *Hash algorithm* `forgematrix_v4`. The dashboard shows FW/s per
  card (algorithm `forgematrix_v4`, as with FreeForgeMiner).
* **Windows:** `start-cmfd.bat` (in the package) — `WALLET`, `POOL` (default Aria), `WORKER`, `DEVICES`, `MODE`, clock locks,
  `AUTO_UV` / `UV_MAX` / `UV_MIN` (NVIDIA), `UV_GOAL` (AMD), `EXTRA`.

## 1.6.0 — CMFD mining

* CMFD mining on NVIDIA (Linux, HiveOS, Windows) and on AMD Radeon (Windows).
* Pools: Aria / AriaBrain (default), Midas Mining, cmfd-pool.online (URLs with certificate pins: MANUAL.md section 13).
* `cmfd1...` addresses are checked before mining starts (a typo is refused).
* `--algo cmfd --self-test` checks the GPU without a pool; with the model present it also runs the official Common Foundry
  1.0.16 test vector.
* **Pearl undervolt offsets break CMFD.** With the Pearl offsets left on the card (+345 on an RTX 4070 Ti, +200 on an RTX
  3070) the engine fails at once ("illegal memory access" / "misaligned address"). So in CMFD mode a positive core offset is
  set to 0 before the engine starts and put back on exit (also after a crash, at the next start). `--uv-offset N` forces N
  at your own risk; without root / Administrator the log warns and mining goes on.
* The Linux / HiveOS engine runs on every HiveOS image (Ubuntu 20.04 and 22.04).

Measured (5-minute runs, core offset 0, auto batch, mode speed):

| GPU | FW/s avg | W avg | FW/kWh | accepted | rejected | stale |
|---|---|---|---|---|---|---|
| RTX 4070 Ti (PL 190 W), batch 64 | 36.76 | 187.4 | 706 k | 84 | 0 | 4 |
| RTX 3070 (PL 135 W), batch 32 | 14.11 | 134.8 | 377 k | 19 | 0 | 2 |

Longer runs: RTX 4070 Ti 30 min, 36.94 FW/s at 189.7 W, 443 accepted / 0 rejected / 58 stale; Midas Mining 2 min, 36.7 FW/s,
26 / 0 / 2; RTX 3070 on Aria 25 min, 14.34 FW/s at 134.7 W (Aria's share difficulty is high: about one share per 20 000 FW).
Stale shares (the block changed while the share was in flight) are harmless.

## 1.6.1 — hotfix

* HiveOS `miner stop` could leave GPU1 and up undervolted with locked clocks until the next start (only GPU0 was put back).
  Now every card's offset and clocks are restored on stop, also when the log pipe is closed first.
* Windows: closing the console window, logging off or shutting down restores the offset and clocks.
* CMFD: once per GPU after 30 s of mining the miner warns if the memory clock is locked low. A Pearl OC with *Mem lock* 810
  cuts CMFD about 6×: an RTX 3070 gave 3.7 FW/s with the memory locked at 810 MHz and 21.8 FW/s at ~237 W with the memory free.
* Pearl `--self-test` without `--devices` checks every GPU.

## 1.6.2 — per-card undervolt limits

* `--uv-max`, `--uv-min`, `--uv-offset` (and `UV_MAX` / `UV_MIN` / `UV_OFFSET` in the .bat files) take one value for all GPUs or
  a comma list with exactly one value per GPU in `--devices` order (`-` = that GPU's default); a wrong length is an error.
* Without `--uv-max`: CMP 30HX/40HX +150, CMP 50HX and Turing RTX 20xx / GTX 16xx +225, all others +495. RTX 50 starts the
  search at +300 or more, RTX 40 at +200 or more. An explicit `--uv-max` above the card default is used, with a WARNING.
* Start log: `auto-uv: GPUn (bus .., NAME): range +A..+B (max: <source>; start: <source>)`.
* A saved offset above a lowered limit runs at the limit (after a quick check); when the limit is raised again, the card is
  tuned again.
* CMP 50HX mines Pearl at only ~2.6 TH/s per card (the same on drivers 550.78 and 595.91.07): not recommended for Pearl.
* Fix: an engine stopped while it was still loading the model could leave processes behind that held about 7 GB of GPU memory
  each; they are now ended with the engine.

## 1.6.3 — `--auto-uv` for CMFD, lock restore, auto-uv after an OC apply

* **`--auto-uv` for CMFD on NVIDIA.** The miner finds an undervolt for CMFD itself, separate from the Pearl result, before the
  engine starts; every step is checked bit-exactly on the real model and with the official test vector, and the result keeps
  a safety margin. It is re-tuned when the power limit changes by more than 3 % or `--lock-cclock` changes. While mining, an
  engine `INTEGRITY | ERROR` or a CUDA fault on a tuned card steps its offset back (saved) and restarts the engine. Your own
  offset comes back on exit. Tuning took about 16 minutes on an RTX 3070 and about 17 minutes for a 4× CMP 50HX rig (all cards
  at once).
* **`--auto-uv` for CMFD on AMD** (Windows, `AUTO_UV=1` in `start-cmfd.bat`): a separate CMFD result with the profiles `eff`,
  `bal` (default) and `max` (`UV_GOAL`); your clock / voltage / power-limit settings are put back on exit.
* **`--lock-cclock` / `--lock-mclock`:** a lock that was there before is put back on exit instead of removed (HiveOS: from the
  OC profile; elsewhere name it in `FFGP_RESTORE_CCLOCK` / `FFGP_RESTORE_MCLOCK`), also after a crash at the next start.
* **Pearl `--auto-uv` after an OC apply:** when HiveOS (or Afterburner) re-applies the OC profile and resets the offset, the
  verified offset is set again after a quick check (at most 4 times an hour); any other value is treated as your own choice.
* Fixes: a `miner stop` during a check was later treated as a crash at that offset, and a stop during the start check paused
  auto-uv for 24 h; both fixed. A re-check while mining no longer pauses a GPU that is still waiting for RAM.

Measured, NVIDIA (Aria, averages of the stats line):

| card / setting | stock (offset 0) | result | tuned | FW/kWh |
|---|---|---|---|---|
| RTX 3070, PL 135 W | 13.84 FW/s, 134.8 W, 369.6 k | **+195** | 13.97 FW/s, 134.9 W, 372.8 k | +0.8 % (power-bound at ~1050 MHz, voltage already at the floor) |
| RTX 3070, PL 220 W | 19.51 FW/s, 214.2 W, 327.9 k | **+135** | 20.41 FW/s, 212.9 W, 345.1 k | **+5.2 %** |
| CMP 50HX, PL 225 W, core lock 1410 MHz | 13.10 FW/s, 215.6 W, 218.7 k | **+225** (its limit) | 13.21 FW/s, 195.6 W, 243.2 k | **+11.2 %** |
| CMP 50HX (second card) | 13.01 FW/s, 217.0 W, 215.8 k | **+225** | 13.17 FW/s, 203.2 W, 233.3 k | **+8.1 %** |

RTX 3070 at PL 135 W with `--auto-uv` (+195), 20 min on Aria: 14.12 FW/s, 134.3 W, 378.5 k FW/kWh, 1 accepted / 0 rejected.
Pearl, RTX 3070, lock 1500/810 MHz, +195, 5 min on LuckyPool: 67.7 TH/s at 99 W (0.685 TH/W), 13/13 accepted; with the stock
offset 67.67 TH/s at 123 W.

Measured, AMD RX 6800 XT (Adrenalin 25.5.1, chip watts from ADLX):

| setting | FW/s | W | FW/kJ |
|---|---|---|---|
| stock (2324 MHz max, 1150 mV) | 10.1 | 280 | 36 |
| `eff` = `bal`: 2100 MHz / 881 mV | 9.13 | 193 | 47.2 (+32 %) |
| `max`: 2324 MHz / 1075 mV | 10.07 | 248 | 40.6 (+14 %) |

On the Aria pool through the miner: `bal` 15 min 9.06 FW/s at 191 W, `max` 10 min 10.03 FW/s at 247 W, stock 10 min 9.86 FW/s
at 277 W; 0 rejected, engine `INTEGRITY | ok`, your settings put back each time. Not useful on this card: the power limit goes
down only to −6 % (9.9 FW/s at 264 W, +4 % FW/W); a higher memory clock or fast memory timings give nothing (and 2150 MHz memory
gives wrong results).
