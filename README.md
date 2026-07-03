# tapas / topaz 5.15 kernel

A GitHub Actions workflow that builds a GKI **android13-5.15** kernel (5.15.194) with
**KageSU** (KernelSU) baked in, packaged as a flashable AnyKernel3 zip. Aimed at topaz /
tapas but, being GKI, it runs on any android13 5.15 device that boots a generic kernel image.

Everything happens in CI — there's no kernel source in this repo, only the build recipe.

## What it builds with

| | |
|---|---|
| Source    | CLO `msm-5.15`, branch `kernel.lnx.5.15.r1-rel` (cloned at build time) |
| Toolchain | topnotchfreaks Clang `r547379` (LLVM), full LLVM/IAS build |
| Root      | KageSU (builtin), with optional SUSFS |
| Output    | `AnyKernel3_KageSU_<ver>_5.15.194_palaziks-ShiftPorts.zip` |

Build speed is kept sane with ccache (public cache pull + ECS backend), a RAM-disk output
dir, and a ThinLTO link cache.

## Building

1. Fork / use this repo, then open the **Actions** tab.
2. Pick **KageSU 5.15 Build** → **Run workflow**.
3. Toggle the options you want (see below) and start it.
4. Grab the AnyKernel3 zip from the run's artifacts (or a GitHub Release if you enabled that).

Flash the zip in a custom recovery, or with `ksud`/AnyKernel3's `Flasher`. Keep your current
boot image around in case you need to revert.

## Options

Defaults are tuned for a daily-driver build; leave them alone if you're unsure.

| Input | Default | What it does |
|---|---|---|
| `SUSFS`            | on  | SUSFS integration (hides mounts/paths) |
| `KPM`              | off | KernelSU KPM support |
| `lz4_enable`       | on  | LZ4 1.10.0 + ZSTD 1.5.7 |
| `lz4kd_enable`     | off | LZ4KD ZRAM path |
| `bbr_enable`       | on  | TCP BBR/Brutal — `false` off, `true` add, `default` make default |
| `adios_enable`     | on  | ADIOS I/O scheduler |
| `rekernel_enable`  | off | Re-Kernel (app freezing) |
| `baseband_guard`   | on  | Anti-brick modem protection |
| `lto_type`         | thin| `none` / `thin` / `full` |
| `bore_enable`      | off | BORE scheduler |
| `zram_writeback`   | off | ZRAM writeback |
| `schedutil_opt`    | off | schedutil governor tweaks |
| `f2fs_opt`         | off | F2FS compression/perf |
| `wireguard`        | on  | WireGuard |
| `netfilter_full`   | off | Full netfilter (conntrack/NAT/hashlimit) |
| `lrng_enable`      | off | LRNG v59 |
| `wakelock_blocker` | on  | Wakelock blocker (idle drain) |
| `droidspaces_enable`| on | SYSVIPC + PID_NS + POSIX_MQUEUE (proot-distro) |
| `ip_set_enable`    | off | IP_SET + IPv6 NAT |
| `ttl_enable`       | off | TTL/HL target |
| `mem_opt_patches`  | on  | WildKernels memory/scheduler patches |
| `github_release`   | off | Publish the zip as a Release |

## Notes

- Built against the **CLO** msm-5.15 tree, so it's meant for ColorOS/OxygenOS/MIUI-style
  bases. It is not an AOSP-clang build.
- Some optional patches (Droidspaces oplus fixes, a few WildKernels ones) are OnePlus/oplus
  specific and simply no-op on this tree — they won't fail the build.
- SUSFS uses the `gki-android13-5.15` patch. If it ever can't apply against a CLO change,
  the build stops on purpose rather than shipping a half-patched kernel; turn `SUSFS` off for
  a clean baseline if that happens.

## Credits

- [KageSU / KernelSU](https://github.com/KageKSU/KageSU)
- [SUSFS](https://gitlab.com/simonpunk/susfs4ksu)
- [topnotchfreaks](https://github.com/topnotchfreaks) — Clang toolchain and base patches
- [WildKernels](https://github.com/TheWildJames/kernel_patches) — optimization patches
- [SukiSU / ReSukiSU](https://github.com/ShirkNeko/SukiSU_patch)
- ccache tooling by [cctv18](https://github.com/cctv18)

## License

The build scripts and workflow in this repo are **GPLv3** (see `LICENSE`). The kernel this
workflow *builds* comes from the CLO msm-5.15 source and remains **GPLv2** — the compiled
image and any kernel-derived files are governed by that, not by this repo's license.
