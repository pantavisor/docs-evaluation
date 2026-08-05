# Persona 06 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude-in-chat (background session, RUNBOOK.md invocation)

## Porting checklist reconstructed from the docs

Starting at `meta-pantavisor/overview`, then `supported-device` → (HTML fallback) →
`overview/port` → `port/platform` → `port/machine` → `port/kas`, the docs give a
concrete, file-by-file checklist for a new board:

1. Check `kas/platforms/` for a platform file matching your SoC family. If none
   exists, create `kas/platforms/<family>.yaml` — `header.version` (16),
   `header.includes` (reuse an existing platform, e.g. `freescale.yaml`), `repos:`
   (vendor BSP layer git URLs/branches), optionally `local_conf_header` and
   `bblayers_conf_header`.
2. Create `kas/machines/<device>.yaml` — `header.version`, `header.includes:
   kas/platforms/<family>.yaml`, `machine: <yocto-MACHINE-name>` (must match a
   `.conf` in the vendor layer's `conf/machine/`), and a `local_conf_header` for
   device-specific vars (`PV_BOOT_OEMARGS`, `UBOOT_DTB_NAME`, `PV_FLASH_README`,
   etc.).
3. Register the machine for CI: add an entry to `.github/machines.json`, run
   `.github/scripts/makeworkflows`, run `.github/scripts/makemachines` (generates
   the pinned lockfile `kas/build-configs/release/<machine>-scarthgap.yaml`), commit
   all three together.
4. Build: `./kas-container menu Kconfig` (or a direct
   `kas build kas/machines/<device>.yaml:kas/scarthgap.yaml:kas/bsp-base.yaml:...`),
   pick `pantavisor-starter` / `-remix` / `-bsp`, get artifacts from
   `build/tmp-scarthgap/deploy/images/<machine>/`.
5. Flash with `pvflasher copy <image>.wic.bz2 /dev/sdX` (or `dd`, or a board-specific
   method — UUU for NXP/Toradex).
6. NAND-specific: set `PV_BOOT_OEMARGS` to include `mtdparts=...` (if the DT doesn't
   already declare MTD partitions) plus `ubi.mtd=ubi pv_storage.device=ubi0:boot
   pv_storage.fstype=ubifs`, and make sure the machine's U-Boot `.cfg` (via
   `PV_MACHINE_UBOOT_CONFIGS`) sets `devtype=ubi` in `bootcmd`.

This part of the task was completed cleanly — see "What worked" below.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting and https://docs.pantavisor.io/development/pantavisor/overview/bsp and https://docs.pantavisor.io/development/pantavisor/overview/pantavisor-architecture | B, Q4 | I need to know what a *healthy* boot looks like on the serial console before I can recognize an unhealthy one, and nothing on either the BSP side or the runtime side shows me one. | Troubleshooting's own heading list is `Device boot-loops after a deploy`, `OTA update appears stuck`, `Claiming the device on Pantahub fails`, `pvr post`/`pvr clone` fails, `Build & layer pitfalls` — every scenario assumes a device that already booted once and is now failing post-deploy, not a first bring-up with no console output at all. `pantavisor-architecture` and `hooks` (checked on the runtime side per this prompt's own instruction) describe states and lifecycle abstractly with no example log. | S1 | `missing-concept` | Add a worked example of a healthy `dmesg`/console transcript from kernel handoff through Pantavisor's own boot log lines, on either the BSP or runtime side, and link it from Troubleshooting. |
| https://docs.pantavisor.io/development/pantavisor/overview/bsp | B, Q2 | I have a working kernel and need to know what to change in my defconfig — nothing tells me which kernel options or subsystems Pantavisor depends on. | "The Linux Kernel, modules and firmware are also part of the revision BSP of the state JSON." — that's the entire treatment of the kernel; no mention of device-mapper (needed for the `dm-crypt`/`dm-verity` `PANTAVISOR_FEATURES` that are on by default per `meta-pantavisor/overview/meta-pantavisor`), squashfs+lz4/zstd, overlayfs (used for the config-overlay mechanism per `pantavisor/overview/containers`), or initrd/cpio support. | S1 | `missing-concept` | Add a "kernel requirements" list (required `CONFIG_*` options/subsystems) to the BSP page or the porting guide, since it's exactly the step that would make a BSP engineer rebuild a defconfig. |
| https://docs.pantavisor.io/development/meta-pantavisor/overview/images | B, Q5 | Nothing states outright where my job (kernel/bootloader/BSP) ends and the container people's job starts — I inferred it only from the fact that `pantavisor-bsp.bb` and container recipes are separate build targets and "Container Development" is a separate page. | "The `pantavisor-bsp` pvrexport (kernel, initramfs, DTBs, modules, firmware) is always mixed in, regardless of the container list — that is what makes the trail bootable." / "Container authoring is covered in Container Development." | S3 | `undefined-jargon` | Add one explicit sentence somewhere in the porting guide stating that the BSP porting job ends once `pantavisor-bsp` boots and everything past that (containers, app deployment) is a separate concern. |
| https://docs.pantavisor.io/development/meta-pantavisor/overview/supported-device.md | B, step 1 | Following the "Don't See Your Board?" pointer, the `.md` export renders "See the porting guides in Porting Pantavisor for how to add new machine support" with the link text present but no hyperlink at all — it reads as plain prose with no way to click through. | `.md` export: "See the porting guides inPorting Pantavisor for how to add new machine support." (no markdown link syntax; rendered HTML confirmed the real target is `/development/meta-pantavisor/overview/port/`) | S4 | `stale` | This is the site's `.md`-export bug (dropped link syntax after bold/at line-end), not a content gap — worth fixing in the export pipeline rather than the page content. |

## What worked

- `meta-pantavisor/overview/boot-flow` fully answered both Q1 (bootloader) and Q3
  (NAND/UBIFS): it gives the exact kernel cmdline
  (`root=/dev/ram rootfstype=ramfs rdinit=/usr/bin/pantavisor`), the FIT/trail
  loading and try-boot logic, and an explicit, board-example
  `PV_BOOT_OEMARGS` for NAND/UBIFS (`mtdparts=... ubi.mtd=ubi
  pv_storage.device=ubi0:boot pv_storage.fstype=ubifs`) with a clear explanation of
  why `mtdparts=` is needed when the device tree doesn't declare MTD partitions.
  This is exactly the level of detail this persona trusts.
- The three-page porting guide (`overview/port` → `port/platform` → `port/machine`
  → `port/kas`) gave a complete, concrete checklist with a real worked example
  (the Toradex/Verdin platform and machine files), which is what produced the
  checklist above.

## Closing summary

- **Task outcome:** completed with detours — the porting checklist, U-Boot
  requirements, and NAND/UBIFS support (Q1, Q3) were fully answered from the docs;
  Q2 (kernel config requirements) and Q4 (healthy-boot picture) were not answered
  anywhere I could find on either side of the fence, and Q5 (job boundary) required
  inference rather than an explicit statement.
- **Worst finding:** S1 — nothing on the runtime (`pantavisor/`) or BSP
  (`meta-pantavisor/`) side of the docs shows what a healthy boot looks like on the
  serial console, so when a new-board bring-up fails (as this persona expects it
  will, "the first six times"), there is no reference-good log to diff against.
- **What worked:** `https://docs.pantavisor.io/development/meta-pantavisor/overview/boot-flow`
  — precise, example-driven, and answered the bootloader and NAND/UBIFS questions
  completely on its own.
- **Confidence:** high — I did not use any knowledge of Pantavisor/meta-pantavisor
  outside what these pages stated, and I did visit the `pantavisor/` runtime section
  (not just `meta-pantavisor/`) as this prompt's own text directs, specifically to
  check Q4 on both sides of the fence.
