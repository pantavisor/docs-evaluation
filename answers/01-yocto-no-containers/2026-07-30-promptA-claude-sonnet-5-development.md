# Persona 01 — Prompt A — 2026-07-30

Target: https://docs.pantavisor.io/development/
Date: 2026-07-30
Model: claude-sonnet-5
Version: development
Run by: manual

## Narrative

Landed on `meta-pantavisor/overview` as instructed — a colleague sent it saying "this is
the Yocto layer, you'll get it." Read top-down, following links in the order a skeptical
Yocto veteran evaluating whether to add this to an existing build would plausibly click:
overview → "Layer Layout" (first topic listed) → "Starter Image" (jumped ahead of "Build
System" because its blurb — "how `pantavisor-starter` composes core containers with the
BSP" — promised the actual concept explanation I needed, while "Build System" sounded
like pure KAS/multiconfig mechanics).

The BitBake mechanics throughout (recipes, classes, the `??=`/`:append` vs `+=` pitfall,
`LAYERSERIES_COMPAT`) are trivial to me — ten years of this, no complaints, the layer
layout page is a perfectly good reference for someone who already knows Yocto. That part
of the docs works.

What doesn't work is that the entire conceptual pitch — what a "container" is here, and
what a "trail" is — is used from the very first sentence of the very first page and never
defined anywhere I could reach by clicking. By the third page I still could not explain
to my team lead what this layer actually buys us over rebuilding a monolithic image, or
what it costs in flash/RAM/boot time — the specific question I came in skeptical about.
That's the point I'd close the tab: not because any single word stopped me, but because
three pages deep, the docs still hadn't made the case, and the one term the whole model
depends on ("trail") has no visible path to a definition.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/images` | A, page 3 (Starter Image) | The whole page pivots on "trail" as if I already know what it is — a rootfs? a git-like history? a running state? — and I never saw it defined anywhere I clicked from the overview. | "the real payload is an initial signed trail under `/trails/0`" | S1 | `missing-concept` | Define "trail" the first time it's used, or link it to wherever it is defined. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/images` | A, whole page | I came here to decide what containers cost me in flash, RAM, boot time, and build complexity versus my current full-image rebuild. Three pages in, nobody has made that case or given numbers — only what gets assembled, not why it's better. | Page enumerates `PVROOT_CONTAINERS_CORE`, build artifacts, and the assembly steps, but has no section addressing size, RAM, or boot-time cost of the container split. | S1 | `missing-concept` | Add a section (even a short one) stating the concrete cost/benefit versus a monolithic image, since that's the decision this page's own audience is making. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview` | A, page 1 (overview) | The very first sentence uses "container" as a known term. I know what a shipping container is; I don't know what this is made of, how it differs from "an image," or why the layer produces both. | "It provides recipes, BitBake classes, and KAS configurations for producing initramfs images and container pvrexport bundles." | S2 | `missing-concept` | Either link "container" to a definition on first use, or add one sentence contrasting it with a plain rootfs/image before using the term. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/meta-pantavisor` | A, page 2 (Layer Layout) | A one-line table cell names a whole framework I've genuinely never heard of, with zero context on what it does or why a fork of it exists. | `lxc-pv` \| Pantavisor-specific LXC fork | S3 | `undefined-jargon` | One clause explaining what LXC is (or a link) would resolve this instantly. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/meta-pantavisor` | A, page 2 (Layer Layout) | Two undefined ideas stacked in one table cell — I don't know what a container is yet, and now I'm told they need a "service mesh" between each other. | `xconnect` \| Service mesh for container-to-container communication | S3 | `undefined-jargon` | Define "service mesh" in this context or link to the xconnect feature's own page if one exists. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/images` | A, page 3 (Starter Image), "Related" section | The last bullet in a list where every other entry is a working link has no link at all — dead end, and I can't tell if it's a missing page or just missing markup. | "- Install guides — flashing the starter image per board" (no `[]()` link, unlike every sibling bullet) | S4 | `unlinked` | Add the link, or remove the bullet if no such page exists yet. |

## Closing summary

- **Task outcome:** blocked at step 3 (Starter Image page) — I would close the tab here, three pages into a top-down read from the entry point I was sent.
- **Worst finding:** the entire mental model rests on "trail," used repeatedly from page 3 onward, and I found no definition or link to one anywhere in my reading path — https://docs.pantavisor.io/development/meta-pantavisor/overview/images.
- **What worked:** `meta-pantavisor/overview/meta-pantavisor` (Layer Layout) — directory structure, key recipes, and the `PANTAVISOR_FEATURES` `:append`-vs-`+=` pitfall are exactly the kind of concrete, BitBake-literate reference I'd want, and cost me nothing to read.
- **Confidence:** high — I stayed in character throughout. I never used outside knowledge of containers, LXC, OCI, or Pantavisor's revision model; every gap logged above is a term or claim the docs themselves introduced without resolving.
