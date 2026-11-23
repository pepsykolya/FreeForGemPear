> [!IMPORTANT]
> ➡️ **FreeForGem 1.7.7 is out — the successor of FreeForGemPear** ([releases](https://github.com/pepsykolya/FreeForGem/releases/latest)). The same Pearl mining and auto-undervolt, plus NOID and CMFD in one miner, 1% fee. Your FreeForGemPear `--auto-uv` results are imported on first start, no new tuning needed. HiveOS: new flight sheet, custom miner `freeforgem`, Installation URL `https://github.com/pepsykolya/FreeForGem/releases/download/v1.7.7/freeforgem-1.7.7_hiveos.tar.gz`. Windows: the zip from the release. FreeForGemPear stays here as the Pearl-only miner with bug fixes.
>
> ➡️ **Вышел FreeForGem 1.7.7 — преемник FreeForGemPear** ([релизы](https://github.com/pepsykolya/FreeForGem/releases/latest)). Тот же майнинг Pearl и автоандервольт, плюс NOID и CMFD в одном майнере, комиссия 1%. Результаты `--auto-uv` из FreeForGemPear подхватываются при первом запуске, заново подбирать не нужно. HiveOS: новый полётный лист, кастомный майнер `freeforgem`, Installation URL `https://github.com/pepsykolya/FreeForGem/releases/download/v1.7.7/freeforgem-1.7.7_hiveos.tar.gz`. Windows: архив из релиза. FreeForGemPear остаётся здесь как майнер только для Pearl с исправлениями.

# FreeForGemPear

GPU miner for **Pearl (PRL)** — pearlhash proof-of-useful-work — with a **1% developer fee**. Since 1.6.0 it also mines
**Common Foundry (CMFD)** and runs on **AMD Radeon** cards (Windows).

Our own GPU kernels (closed source). Every share proof is built and checked with the official `zk-pow` verifier before it is
submitted.

## Performance

RTX 3070 locked at 1560 MHz core / 810 MHz memory, LuckyPool, 5-minute run:

| | Hashrate | Power |
|---|---|---|
| FreeForGemPear 1.4.6 | 70.5 TH/s | ~107 W (optional `--auto-uv`: see below) |
| FreeForGemPear 1.4.5 | 70.2 TH/s | ~107 W |
| FreeForGemPear 1.3.1 | 70.2 TH/s | ~107 W |
| FreeForGemPear 1.3.0 | 70.2 TH/s | ~107 W |
| FreeForGemPear 1.2.0 | 69.1 TH/s | ~107 W |
| FreeForGemPear 1.1.0 | 68.3 TH/s | ~114 W |
| FreeForGemPear 1.0.0 | 65.6 TH/s | ~105 W |


## Supported hardware

* **Tuned for:** RTX 30xx (Ampere, sm_86) — 3060 / 3060 Ti / 3070 / 3070 Ti / 3080 / 3090.
* **RTX 40xx (Ada, sm_89):** own kernel since 1.5.4 (RTX 4070 Ti @2445/810: 144.7 TH/s at ~169 W vs 144.3 at ~179 W with the RTX 30xx kernel). Tested on RTX 4070 Ti and RTX 4070 Laptop; other 40xx cards use the same kernel.
* **RTX 50xx (Blackwell, sm_120):** supported since 1.5.0 (not specially tuned yet), needs NVIDIA driver 570+.
* **Turing (RTX 20xx / GTX 16xx / CMP 30HX–50HX, sm_75):** supported since 1.5.3 with an own kernel (~49.9 TH/s on an RTX 2060 SUPER @1605 MHz); lock the memory at 5001, not 810. CMP 50HX gives only ~2.6 TH/s: not recommended for Pearl, but it mines CMFD well (~13 FW/s per card at ~220 W). Unlocking CMP cards is not provided by FreeForGemPear (MANUAL.md section 7, *CMP cards*).
* **AMD Radeon (Windows only, since 1.6.0):** own kernel, tested on RX 6800 XT (RDNA2): 34.4 TH/s at ~260 W stock,
  ~30–32 TH/s at 160–172 W with `--auto-uv`. RX 6000 (RDNA2) supported; RX 7000/9000 should work but are untested; integrated Radeon
  graphics are skipped. No AMD build for Linux/HiveOS yet. The miner picks CUDA or AMD by itself (`--backend auto|cuda|amd`).
  Details: [MANUAL.md](MANUAL.md) section 14.
* **Not supported:** older GPUs (Pascal and earlier), A100 (sm_80), AMD RX 5000 and older, AMD on Linux/HiveOS.
* **System:** Linux x86_64 or Windows 10/11 x64, NVIDIA driver 525 or newer (RTX 50xx: 570 or newer). The Linux binary is built on
  Ubuntu 20.04 (glibc 2.31), so it runs on Ubuntu 20.04+ and HiveOS.

Full manual in English, Russian and Chinese: **[MANUAL.md](MANUAL.md)** (HiveOS, Linux, Windows, pools, overclocking, troubleshooting).

## Usage

```
tar xzf freeforgempear-1.6.3-linux-x86_64.tar.gz
cd freeforgempear-1.6.3
./freeforgempear --self-test                      # optional: checks the GPU kernel against the CPU reference
./freeforgempear --pool pearl-eu2.luckypool.io:3360 --wallet YOUR_PRL_ADDRESS --worker rig1
```

| Option | Meaning |
|---|---|
| `--pool HOST:PORT` | stratum pool (default `pearl-eu2.luckypool.io:3360`) |
| `--wallet ADDRESS` | your PRL address (required) |
| `--worker NAME` | worker name (default `worker1`) |
| `--password TEXT` | stratum password (default `x`); on Kryptex `d=VALUE` sets a custom share difficulty |
| `--devices 0,1,...` | GPUs to use (default: all visible; honours `CUDA_VISIBLE_DEVICES`) |
| `--backend auto\|cuda\|amd` | GPU library: auto (default) = CUDA if it sees an NVIDIA GPU, else AMD (Windows) |
| `--minutes N` | stop after N minutes (default: run until Ctrl-C / SIGTERM) |
| `--self-test` | verify the kernel and exit |
| `--auto-uv` | optional (off by default): find, apply and supervise a safe undervolt of the GPU (V/F offset), needs root/Administrator, restores your settings on exit; see [MANUAL.md](MANUAL.md) section 12 |
| `--uv-offset N` / `--uv-reset` | apply a fixed undervolt offset after a check / forget the saved undervolt results |
| `--uv-max N` / `--uv-min N` | highest / starting undervolt offset; default per card (RTX 20xx / GTX 16xx up to +225, CMP 50HX up to +225, CMP 30HX/40HX up to +150, others +495). One value for all GPUs, or one per GPU in `--devices` order: `--uv-max 225,495,150` |

The miner reconnects to the pool on its own. Every 10 s it logs the hashrate per GPU and in total,
plus the submitted, accepted and rejected shares.

**Tuning.** For best efficiency on a 3070, lock the core clock (for example
`nvidia-smi -lgc 1560,1560`) and lower the memory clock (`nvidia-smi -lmc 810,810`): Pearl does not need a high memory
clock. At 1560 MHz a 3070 draws ~107 W. With the optional `--auto-uv` (new in 1.4.0) the miner also finds the lowest safe core voltage
of each GPU, checking every step bit-for-bit on the GPU and on the CPU, and restores your settings on exit
(roughly 5-10 W less per card at the same hashrate). Mixed rig: `--auto-uv` is enough, the limits are chosen per card
automatically (since 1.6.2); for manual control `--uv-max 225,495,150`. The work size is picked per GPU automatically, so
every card of a mixed rig runs at its own best rate (since 1.4.7).

**CPU.** The miner needs well under 10% of one CPU core per GPU, so multi-GPU rigs with older CPUs run at full speed.

## Windows

Download `freeforgempear-1.6.3-windows-x64.zip`, unpack it to a folder without spaces, pick the ready start file for your pool
(`start-luckypool-eu/ru/us/asia.bat`, `start-kryptex-eu/ru/us/asia.bat`; `start.bat` = LuckyPool Europe), replace the example
`WALLET` (the developer's address) with your own and run it. Output goes to the window and to `logs\`; a miner that
exits is restarted after 15 s. Only the NVIDIA driver is needed (AMD Radeon: the AMD Adrenalin driver). Details: [MANUAL.md](MANUAL.md#5-windows-1011),
`README-WINDOWS.txt`.
For a driver clock lock + power limit through `nvidia-smi` there is `start-nvidia-smi.bat` (run as Administrator).

## HiveOS

FreeForGemPear runs on HiveOS as a Custom miner. Set the flight sheet *Miner* to *Custom* and use
this *Installation URL*:

```
https://github.com/pepsykolya/FreeForGemPear/releases/download/v1.6.3/freeforgempear-1.6.3.tar.gz
```

Wallet template `%WAL%.%WORKER_NAME%`, pool URL `pearl-eu2.luckypool.io:3360`.

**NOCK merge mining (LuckyPool).** Use `prl1...+YOUR_NOCK_ADDRESS` as the wallet and LuckyPool pays NOCK for the same work,
with no hashrate loss (Linux, Windows `.bat` and HiveOS alike; on other pools the miner drops the `+NOCK` part by itself). The step-by-step guide,
in English and Russian, with overclock settings is in **[HIVEOS.md](HIVEOS.md)**.

## Common Foundry (CMFD)

Since 1.6.0 the same binary also mines **Common Foundry (CMFD, ForgeMatrix V4)**:
give it a CMFD pool URL and your CMFD address (`cmfd1...` as the wallet 1.0.16+ shows it, or the older 64-hex form), everything else works as for Pearl
(one process for all GPUs, `--devices`, `--lock-cclock/--lock-mclock/--moffset`, `--minutes`, the stats line, HiveOS, Windows):

```
./freeforgempear --pool 'cmfd+tls://IP:PORT?pin=64HEX' --wallet cmfd1YOUR_ADDRESS --worker rig1
./freeforgempear --algo cmfd --self-test
```

The CMFD work is done by the **FreeForgeMiner** engine (MIT, <https://github.com/pepsykolya/freeforgeminer>), shipped
in the `cmfd/` folder (release 1.2.4; the Linux binaries are built from the same source on Ubuntu 20.04, so CMFD runs on every
HiveOS): same consensus code, same speed (RTX 3070 ~20 FW/s, RTX 4070 Ti ~35 FW/s;
AMD RX 6800 XT ~10 FW/s on Windows). The 6.4 GB
model is downloaded once from the official Common Foundry servers and then only verified (`--cmfd-model DIR`; on HiveOS it is
shared with FreeForgeMiner in `/hive/miners/custom/cmfd-model`). Needs about 7 GB of free GPU memory and an RTX 20xx or newer.
`--cmfd-mode eco` = RTX 30 about 13% fewer watts for 2% less hashrate. A Pearl undervolt (positive core offset) makes CMFD
compute wrong results, so in CMFD mode the miner sets it to 0 for the run and puts it back on exit (needs root/Administrator;
`--uv-offset N` forces a value at your own risk). `--auto-uv` (NVIDIA) tunes a separate CMFD offset, every step checked
bit-for-bit on the real model (MANUAL.md, section 13).
**Use a separate OC profile for CMFD without the memory lock:** the Pearl OC (memory locked at 810 MHz) cuts CMFD about 6×
(RTX 3070 on HiveOS: 3.7 FW/s with *Mem lock* 810, 21.8 FW/s at ~237 W with the memory free); the miner warns about a low
memory clock in the log but never changes your OC.
Pools: **Aria** (default) `cmfd+tls://159.69.194.46:29445?pin=9dfb51083f287726117f689f87bc7a878792efcca58ca8b6f6e7c05ac5d152e9`,
**Midas Mining** `cmfd+tls://24.107.152.151:29745?pin=2ee4e6a028b07b300753dff3c260b79763c5e748d4e78b86cf3855e97eb2e80e`;
more in [MANUAL.md](MANUAL.md), section 13.
HiveOS: the same flight sheet with *Hash algorithm* `forgematrix_v4`, wallet template `%WAL%.%WORKER_NAME%` and the
`cmfd+tls://` pool URL; Windows: `start-cmfd.bat`. Details: [MANUAL.md](MANUAL.md) section 13 and `cmfd/README.md` in the package.

**CMFD dev fee: 1%**, inside the engine: 36 s per hour per GPU (random slot) on your pool, worker `ffm-fee`,
log lines `DEV FEE | ...`. The Pearl fee below is not run in CMFD mode.

## Developer fee

1% of the mining time: one 60-second slot in every 100 minutes, at a random moment of the cycle,
on **the same pool you configured**, under a separate login (worker `ffgp-devfee`; on LuckyPool it also merge-mines NOCK for the developer). The log shows
`devfee: next slot in N min`, `devfee: mining 60s for FreeForGemPear dev (1%)` and
`devfee: back to user`. The first slot comes at a random moment within the first 100 minutes. If the fee login
can't be served, the slot is skipped; your mining is never blocked.

The fee is a condition of the license: removing, reducing or redirecting it is not permitted
(see [LICENSE](LICENSE), clause 3).

## License

FreeForGemPear is distributed **in binary form only** (see [Releases](https://github.com/pepsykolya/FreeForGemPear/releases)) under the
**FreeForGemPear Binary License 1.0**: free to use and to redistribute the release packages unmodified, **as long as the developer
fee stays intact**. See [LICENSE](LICENSE). Versions up to and including 1.6.3 stay under the license they were published
with. Third-party components are listed in [NOTICE](NOTICE).
