# Persona 06 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: manual (fresh session, docs-framework repo)

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/pantavisor/overview/containers`, `https://docs.pantavisor.io/development/meta-pantavisor/overview/meta-pantavisor`, `https://docs.pantavisor.io/development/meta-pantavisor/overview/port/*` | A, after locating the porting guide, sizing the container-runtime portion of the estimate | I now know containers run under a fork of LXC, not Docker, and I know how to wire a new machine into KAS. What I never find, anywhere reachable, is a list of kernel config options (namespaces, cgroups, overlayfs, squashfs, device-mapper) my existing defconfig needs for LXC and the config-overlay mechanism to actually work. My kernel already boots — but "boots" and "can host `mount -t overlay` and cgroup-confined processes" are different claims, and nothing tells me which one I have. | "Pantavisor implements a lightweight container run-time with the help of Linux Containers (LXC)." (containers page) — no page anywhere in `/pantavisor/overview/`, `/meta-pantavisor/overview/`, the porting guide, or troubleshooting/FAQ states required kernel config options for LXC/overlayfs support. | S1 | `missing-concept` | Add a "kernel requirements" checklist (CONFIG_NAMESPACES, CONFIG_CGROUPS, CONFIG_OVERLAY_FS, CONFIG_SQUASHFS, device-mapper as applicable) to the porting guide or the BSP/Containers overview page. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/supported-device.md` (and, less critically, `.../getting-started/start/download-and-flash.md`, `.../getting-started/how-to-install.md`) | A, following links from the `start` page toward "how do I port to new hardware" | The single most important link for this whole task — "Porting Pantavisor" — is invisible in the page's own `.md` export: the text renders as `See the porting guides inPorting Pantavisor for how to add new machine support.` with no brackets, no URL, word run together with the link text. Two earlier links ("Install on hardware", "Board Guides") have the identical problem. Since rule 1 tells me to prefer the `.md` export over the rendered page, a strict reading of my own instructions would have dead-ended the task at the very first page with no path to the porting guide at all. I only found it by fetching the rendered HTML instead and reading the actual `<a href>`. | `.md`: `"...seeInstall on hardware for board-specific guides."` / `"...See the porting guides inPorting Pantavisor for how to add new machine support."` — no `[text](url)` markdown present. Rendered HTML: `<a href=/development/meta-pantavisor/overview/port/>Porting Pantavisor</a>` — the link is real and correct, just dropped by the `.md`/"Copy page" export. | S2 | `broken-path` | Fix the Docusaurus "Copy page" Markdown generator so inline links immediately following bold text, at the end of an ordered-list item, or inside a table cell aren't stripped of their `[text](url)` syntax — this is the same export AI agents and the "Copy page" button both produce. |

## Absence of findings

Once the porting guide was actually reached, it worked well for this persona.
`Porting Pantavisor` → `Platform` → `Machine` → `Building`
(`/development/meta-pantavisor/overview/port/platform`,
`/development/meta-pantavisor/overview/port/machine`,
`/development/meta-pantavisor/overview/port/kas`) gives concrete file paths
(`kas/platforms/<family>.yaml`, `kas/machines/<device>.yaml`), a worked example with
real YAML (the Toradex/Verdin platform and machine files), and exactly the kind of
"what does the CI registration entry look like" detail I'd want before quoting a
number. `Boot Flow` (`/development/meta-pantavisor/overview/boot-flow`) is the
best page on the whole site for my bias against boot-flow prose that doesn't show
the actual sequence: it gives the literal `boot.cmd.pvgeneric` walkthrough, the
`root=/dev/ram rootfstype=ramfs rdinit=/usr/bin/pantavisor` command line, the FIT
image paths under `/trails/<rev>/`, and the `PV_BOOT_OEMARGS` / `mtdparts=` handling
for NAND — real variables, in order, exactly what I trust. `BSP`
(`/development/pantavisor/overview/bsp`) cleanly draws the boundary I expected to
need drawn: bootloader/kernel/DT bring-up is on me, Pantavisor owns the trail/revision
side once U-Boot hands off. The FAQ's "How does Pantavisor run Docker images?"
entry (`/development/meta-pantavisor/getting-started/troubleshooting/faq`) is the one
place that explicitly says containers run under LXC, not a Docker daemon — useful,
though I only reached it via the troubleshooting index, not via any link from the
porting guide itself.

## Work estimate

I can give a number, but only as a range with one named unknown, not a single
figure I'd be held to without qualification.

**Baseline (KAS/BSP integration work — everything the docs actually cover), 5–6 days:**
- Write `kas/platforms/<family>.yaml` pointing at my own BSP layer's repo, reusing
  `freescale.yaml` for the base i.MX layers if compatible — 0.5–1 day.
- Write `kas/machines/<device>.yaml` binding my `MACHINE` name, plus any
  `local_conf_header` overrides — 0.5 day.
- Wire the U-Boot boot script: confirm `boot.cmd.pvgeneric` picks up my storage
  backend (MMC/SD in my case), set `PV_BOOT_OEMARGS`/`PV_MACHINE_UBOOT_CONFIGS` as
  needed, get a first FIT image booting through Pantavisor as init — 1.5–2 days,
  most of the risk here is my own u-boot env, not something the docs are missing.
- First `pantavisor-starter` KAS build, iterate on build breakage — 1–1.5 days.
- First boot with Pantavisor as init, confirm the pvr-sdk/starter containers come up
  and I can `pvr clone`/`pvr post` against the device — 1 day.

**Unresolved, +2–5 days contingency:** whether my kernel defconfig already has what
LXC and the config-overlay (`overlayfs`) mechanism need. I have no way to check this
against the docs — it isn't stated anywhere I could find. Best case, my defconfig
already has cgroups/namespaces/overlayfs/squashfs (increasingly likely on a modern
i.MX8M kernel branch) and this contingency collapses to near zero. Worst case, it's
missing something and I'm patching + retesting kernel config, which is exactly the
kind of variable-cost work I don't want to silently fold into a number I'd be held to.

**My estimate: 5–6 days confident baseline, with a stated +2–5 day contingency I
cannot resolve from the docs alone.** I would tell management that number with the
contingency attached, not a single collapsed figure — and I'd flag that resolving it
takes either a docs answer or ten minutes of `zcat /proc/config.gz | grep -E
'CGROUP|NAMESPACE|OVERLAY|SQUASHFS'` on the running board, which I can't do from a
docs read alone.

## Closing summary

- **Task outcome:** completed with detours — I reached a real, usable porting guide
  and can produce a bounded estimate, but only after a routing failure in the `.md`
  export nearly dead-ended the task at the entry page, and only with an explicit
  contingency range rather than a single number, because of the missing kernel-config
  gap.
- **Worst finding:** the docs never state what kernel config (namespaces, cgroups,
  overlayfs, squashfs) my existing BSP kernel needs for LXC and the config-overlay
  mechanism to work — the exact container/kernel boundary this persona's mental model
  says the docs have to draw, and it's the one input I can't get from the docs at all.
- **What worked:** the three-page porting guide at
  `/development/meta-pantavisor/overview/port/{platform,machine,kas}` — concrete file
  paths, real worked-example YAML, and a build/flash walkthrough that matches how a
  BSP engineer actually reads.
- **Confidence:** I stayed in character — I did not assume any LXC/cgroups/overlayfs
  kernel-config knowledge I wasn't shown, which is exactly why that gap became a
  finding instead of something I quietly filled in myself. The one place I went
  beyond a strict `.md`-only read was cross-checking rendered HTML for the three
  dropped links; without that step the task would have dead-ended at the entry page,
  which itself is the S2 finding above, not a break in character.
