# Persona 02 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Task

Read `https://docs.pantavisor.io/development/meta-pantavisor/overview` and follow the
docs' own reading order for as long as a real Buildroot engineer evaluating whether they
can adopt this without Yocto would plausibly read, producing a jargon audit judged
strictly from eight years of Buildroot experience and zero Yocto/BitBake/container
exposure.

Path actually followed (in-character, the way someone scoping out a new build system
would): `overview` → `overview/glossary` (via the "container" link — the first unknown
word on the page happened to be a live link, so I clicked it immediately) → the six
"Topics" pages in listed order except I skipped "Flashing Images" and "Flashing NXP
devices" as clearly board-bring-up detail, not core to the Yocto question →
`overview/get-started` (Build Guide #1) → `overview/pantavisor-development` (Build Guide
#3) → `overview/container-development` (Build Guide #4). I stopped before "Supported
Devices," "Manifest Audit," "Component Docs," "Bootchartd," and the CI overview — by that
point the pattern (deep, unexplained BitBake internals throughout) was already clear, and
a real engineer sizing up a build system would have seen enough to form a judgment.

## Jargon audit

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? | Evidence | Severity | Type |
|---|---|---|---|---|---|---|---|
| `image` (Docker sense) | [`.../overview/images`](https://docs.pantavisor.io/development/meta-pantavisor/overview/images) | No | Same as "deployable image" two paragraphs above on this exact page — a flashable rootfs/disk image, the only sense of "image" I know from Buildroot | No — it's a Docker container image, a technology I've never touched, distinguished from the disk-image sense only by a negative parenthetical | "`pantavisor-appengine` \| Docker-based image for local appengine testing (not a flashable device image)" | S1 | `undefined-jargon` |
| `sstate` | [`.../overview/get-started`](https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started) | No | Some kind of build-output cache directory, similar to Buildroot's `dl/` download cache — guessed purely from `SSTATE_DIR`/`DL_DIR` appearing side by side | Can't confirm — plausible from naming alone, but nothing ever says what's actually cached (fetched sources? compiled artifacts? both?) or why the whole worktree-sharing workflow hinges on it | "Yocto's `SSTATE_DIR` and `DL_DIR` are designed for concurrent access — sstate artifacts use per-task hashes and atomic rename." | S1 | `missing-concept` |
| `layer` | [`.../overview`](https://docs.pantavisor.io/development/meta-pantavisor/overview) | No | A self-contained chunk of build configuration sitting on top of a base — my closest Buildroot analogy is an external tree | Can't confirm — no page anywhere in my path explains how layers combine, override each other, or resolve priority; I don't actually know if my analogy holds | "meta-pantavisor is the Yocto/OpenEmbedded layer that builds Pantavisor-based BSP images for embedded Linux." | S1 | `missing-concept` |
| `machine` | [`.../overview`](https://docs.pantavisor.io/development/meta-pantavisor/overview) (Build Guide list) | No | The target hardware board — the same idea as picking a Buildroot `defconfig` | Apparently yes, by luck — every later usage (`kas/machines/`, `images/{machine}/`, `PV_MACHINE_UBOOT_CONFIGS`) is consistent with that guess, but nothing ever states it; I'd have had no way to know if I'd guessed wrong | "2. Supported Devices — machines supported and built by CI" | S2 | `undefined-jargon` |
| `BitBake` | [`.../overview`](https://docs.pantavisor.io/development/meta-pantavisor/overview) | Partially, 2 pages later | A lower-level CLI build engine that KAS wraps, similar to `make` sitting under a higher-level wrapper | Roughly yes, confirmed indirectly by Get Started's "KAS vs BitBake Command Reference" table pairing `kas build <config> --target <recipe>` with `bitbake <recipe>` as equivalents — never stated outright, only shown side by side | "It provides recipes, BitBake classes, and KAS configurations for producing initramfs images and container pvrexport bundles." | S2 | `undefined-jargon` |
| `KAS` | [`.../overview`](https://docs.pantavisor.io/development/meta-pantavisor/overview) | Partially, 2 pages later | Some kind of manifest/config-aggregation tool, like a repo-manifest tool | Roughly yes — Build System's "KAS is the primary build system. Configuration is composed by layering YAML fragments" matches, once I reached it two pages later | "...and KAS configurations for producing initramfs images and container pvrexport bundles." | S2 | `undefined-jargon` |
| `recipe` / `.bb` | [`.../overview`](https://docs.pantavisor.io/development/meta-pantavisor/overview) | No explicit statement, inferable from repeated table pattern | A per-component build/packaging instruction file — the rough equivalent of a Buildroot package `.mk` | Probably, based on the pattern in every "Key Recipes" table (one `.bb` per component), but no page ever states this plainly | "It provides recipes, BitBake classes, and KAS configurations..." | S3 | `undefined-jargon` |
| `OpenEmbedded` | [`.../overview`](https://docs.pantavisor.io/development/meta-pantavisor/overview) | No | Just another name for Yocto, used interchangeably | Can't tell — never distinguished from "Yocto" anywhere I reached, so I don't know if they're the same thing or if OpenEmbedded is something narrower underneath it | "meta-pantavisor is the Yocto/OpenEmbedded layer..." | S3 | `undefined-jargon` |
| `LXC` (bare, before expansion) | [`.../overview/glossary`](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) (Container entry) | Yes, 2 pages later | Presumably a Pantavisor-proprietary container format — I had no other referent | No — it turned out to be an independent, real project (Linux Containers) that Pantavisor forks, not something Pantavisor invented; I had no way to know that from the glossary entry alone | "An LXC container defined as a part of the device state: a squashfs root filesystem plus a `run.json` runtime manifest..." | S3 | `undefined-jargon` |
| `OCI` / `runc` | [`.../overview/meta-pantavisor`](https://docs.pantavisor.io/development/meta-pantavisor/overview/meta-pantavisor) | No | Some alternate container-runtime standard, distinct from LXC, guessed from the initials | Can't confirm — the page says containers are "LXC-based regardless of this flag," implying `runc`/OCI is a seldom-used alternate path, but never explains when or why it'd matter | "`runc` \| OCI runtime support (a Pantavisor runtime build option...). Containers created via `pvr app add` are LXC-based regardless of this flag" | S3 | `undefined-jargon` |
| `pseudo` (Pseudo Path Mismatch) | [`.../overview/container-development`](https://docs.pantavisor.io/development/meta-pantavisor/overview/container-development) | No | Some kind of build sandboxing/permission-shim tool, guessed purely from the error format looking like a filesystem-metadata database | Can't confirm — never explained at all, even in the "Common Issues" section built entirely around it | "Errors like `path mismatch [1 link]: ino XXXXX db '...' req '...'` during image builds indicate pseudo database corruption..." | S3 | `undefined-jargon` |
| `WIC` / `WKS` | [`.../overview/meta-pantavisor`](https://docs.pantavisor.io/development/meta-pantavisor/overview/meta-pantavisor) | No | A Yocto-specific disk-partitioning/image-layout tool and config format, roughly analogous to Buildroot's `genimage` plus a partition table | Can't confirm — never explained beyond the one-line directory comment | "└── wic/ # WIC disk image layout files" | S4 | `undefined-jargon` |

## What worked without issue

The [Glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary)
is genuinely good for Pantavisor's own vocabulary — alphabetized, one clean paragraph per
term, and linked directly from the first use of "container" on the overview page. Every
Pantavisor-native concept I needed (`trail`, `container`, `BSP`, `revision`, `pvrexport`,
`state`, `xconnect`) was there and consistent with how it's used later. `trail`
specifically — a term that blocked other personas on other prompts — was a non-issue for
me: it's hyperlinked to the glossary at its very first use on the Starter Image page.

The gap is precisely where the prompt said to look: the glossary defines *Pantavisor's*
words but not one entry for `layer`, `recipe`, `BitBake`, `KAS`, `sstate`, or
`OpenEmbedded` — the Yocto vocabulary the whole meta-pantavisor section is written in. The
docs authors clearly built a glossary for their own product but never noticed the build
system underneath it needed one too.

## Closing summary

- **Task outcome**: completed.
- **Worst finding**: S1 — on the Starter Image page, "image" means a flashable disk image
  in one paragraph and a Docker container image two paragraphs later in the same table,
  distinguished only by the negative parenthetical "(not a flashable device image)" —
  never a positive explanation of what a Docker image actually is, which is exactly the
  kind of silent meaning-swap that would send a Buildroot reader down the wrong path
  without ever knowing they'd gone wrong.
- **What worked**: [`.../overview/glossary`](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) —
  thorough, alphabetized, and linked from first use of "container"; it cleanly defines
  every Pantavisor-native term I needed and never once left me guessing about `trail`,
  `revision`, or `BSP`.
- **Confidence**: high — I held the boundary. Embedded-Linux fundamentals I used to read
  MMC/NAND/UBIFS, squashfs, device trees, and U-Boot came from the persona's own declared
  "know cold" list, not outside knowledge. I evaluated every Yocto term (`layer`,
  `recipe`, `BitBake`, `KAS`, `sstate`, `OpenEmbedded`) and every container term (`LXC`,
  `OCI`, Docker `image`) cold, as instructed, and logged a gap rather than silently
  filling it whenever a page assumed either.
