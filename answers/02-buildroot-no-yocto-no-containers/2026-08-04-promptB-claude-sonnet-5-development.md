# Persona 02 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Report

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks/vs-buildroot` | B, Q1 | This page finally gives me a straight answer to "can I skip Yocto" — it says Pantavisor is a binary plus an "LXC fork" I can drop into my own rootfs — but I don't know what LXC is or why an LXC fork of it matters for keeping my Buildroot workflow. | "Pantavisor is a small static binary plus an LXC fork — it can be cross-compiled with Buildroot's toolchain and packaged as the init for a Buildroot-built rootfs." | S3 | `undefined-jargon` | Link "LXC" to a short definition (or add it as its own glossary term) the first time it's used outside the Container entry. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview` | B, Q1 | This is the page I land on to find out if I can skip Yocto. Its Topics and Build Guide lists only cover layer layout, build system, starter images, flashing, and boot flow — nothing here points at a build-system comparison. I only found the real answer three hops later, off a "see also" on the Starter Image page's cost section. | "## Topics\n1. Layer Layout ... 6. Flashing NXP devices" / "## Build Guide\n1. Get Started ... 7. Bootchartd" — no comparison/alternatives entry in either list | S3 | `broken-path` | Add a direct link to "Pantavisor vs Buildroot" from the overview page itself, not only from a Starter Image aside three hops deep. |
| `https://docs.pantavisor.io/development/pantavisor/overview/remote-control` | B, Q4 | This page (which I followed into from meta-pantavisor's glossary, since the wire mechanism is the runtime's) tells me an update downloads "object metadata... including object size, sha and download URL" — that's the mechanism — but it never gives me an actual number for how big a typical field update is on the wire. | "a new update (same as a new state JSON) has been received and the device will now download the object metadata information for each of its objects. This includes object size, sha and download URL." | S3 | `no-example` | Add one worked example (bytes actually transferred for a single-container update) next to the object-metadata description. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview` | B, Q1 | The page tells me to skip the build guide and "go straight to Develop applications," but that phrase isn't a link anywhere on the page, so I can't actually take the shortcut it describes. | "grab a ready-made image via Starter Image / Flashing Images and go straight toDevelop applications . The build guide below is for building your own custom image from source." | S4 | `unlinked` | Turn "Develop applications" into an actual link to its target page. |
| `https://docs.pantavisor.io/development/` | B, entry point | The plain version root 404s, so I can't start from the obvious top-level URL for this version at all — I had to enter via the meta-pantavisor overview instead. | "The server returned HTTP 404 Not Found." | S4 | `stale` | Make the bare `/development/` path redirect to a real landing page. |

### What was answered cleanly (no finding)

- **Q1 (can I use this without Yocto?)** — Yes, explicitly, on `.../benchmarks/vs-buildroot`: "Buildroot doesn't have an equivalent upstream layer, but Pantavisor is a small static binary plus an LXC fork — it can be cross-compiled with Buildroot's toolchain and packaged as the init for a Buildroot-built rootfs." It also honestly flags the trade-off: "most production users go via Yocto + meta-pantavisor because the integration work is already done."
- **Q2 (smallest change to my build)** — Same page, concretely: "1. Build a minimal Buildroot rootfs containing only the kernel + Pantavisor binary + LXC. 2. Deliver the initial `/trails/0/` composition (BSP container + your apps) alongside. 3. From then on, every change ships as a Pantavisor state revision via Pantahub — no Buildroot rebuild." That's a real, concrete answer to "smallest thing I'd have to change."
- **Q3 (flash cost)** — `.../getting-started/solutions/firmware-size` gives real numbers: "Pantavisor (PID 1): ~1 MB", "LXC + supporting binaries: ~3–5 MB", "Kernel + initramfs: 5–15 MB", "Minimal BSP container: 8–20 MB", and "squashfs container volumes are typically 50–70% smaller than a plain rootfs" — directly answers "what does it cost me in flash" against my 40 MB baseline.
- **Q4, mechanism half** — `.../pantavisor/overview/remote-control` documents the update trigger and per-object download mechanism (state JSON diff, only changed objects move) even if it stops short of a concrete number (see finding above). The one concrete-ish number I did find, on the meta-pantavisor side, was illustrative rather than measured: "Differential OTA. Updates ship only changed object hashes; a 200 KB config tweak doesn't push a 200 MB rootfs over cellular" (`.../getting-started/solutions/firmware-size`).

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding:** No S1s this run — every one of the four questions has an answer somewhere on `development`. The worst is S3: the sentence that actually answers "can I skip Yocto" (`.../benchmarks/vs-buildroot`) leans on "LXC fork" without ever defining LXC, which is exactly the kind of container jargon I was told I don't know.
- **What worked:** `.../getting-started/benchmarks/vs-buildroot` and `.../getting-started/solutions/firmware-size` — between them they gave a direct yes/no on the Yocto question, a concrete 3-step minimal-change path, and real flash-size numbers against my 40 MB baseline.
- **Confidence:** High — I did not use outside knowledge of LXC, Yocto, or containers to fill gaps; where a term (LXC) was used undefined, I flagged it as a finding instead of silently explaining it myself.
