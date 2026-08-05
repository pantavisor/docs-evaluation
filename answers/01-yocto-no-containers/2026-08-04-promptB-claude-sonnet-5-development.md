# Persona 01 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude (background agent session)

## Narrative

Started at `meta-pantavisor/overview` (the Yocto-layer landing page) and read the
Topics/Build Guide lists top to bottom, then followed the "Getting Started" /
"Develop applications" trail a colleague would point me at for the actual weekly-update
workflow, and finally the "Pantavisor Runtime" section the prompt told me to trace into.
Concretely: `overview` → `overview/meta-pantavisor` (Layer Layout) →
`overview/build-system` → `overview/get-started` → `overview/container-development` →
`overview/glossary` → `getting-started/start` → `getting-started/develop` →
`getting-started/develop/application/install` → `.../install/local-pvr` →
`.../install/local-pvtx` → `.../install/remote-pantahub` →
`getting-started/security/trust-model` → `getting-started/benchmarks` →
`getting-started/benchmarks/vs-yocto` → `getting-started/benchmarks/vs-docker` →
`getting-started/troubleshooting/faq` → `getting-started/migrate/docker-compose` →
`pantavisor/overview` → `pantavisor/overview/revisions` → `pantavisor/overview/updates` →
`pantavisor/overview/containers` → `pantavisor/reference/pantavisor-state-format-v2`.

Answering the five questions as I'd have to for my team lead, strictly from what these
pages say:

**1. Do my existing recipes/layers survive?** Mostly yes, but with a real gap. The
`vs-yocto` page lays out a two-phase model: Yocto still builds the kernel, BSP, and
initial container set — "build once, rarely changes" — and `meta-pantavisor` is added as
another layer via standard BitBake (`bitbake pantavisor-bsp`) or KAS. My existing BSP
recipes and layers keep working. What's unclear is what happens to *this specific app's*
recipe — the one that currently does `IMAGE_INSTALL` into the main rootfs. The Container
Development guide shows how to build a *new* container from a BitBake recipe
(`inherit core-image container-pvrexport`), which in principle could just wrap my
existing package. But nothing on the page that actually walks me through installing an
app (`develop/application/install`) mentions this recipe-built path at all — it only
offers "a Docker Hub image or a pvrexport bundle," and never explains that a
pvrexport bundle is exactly what the Container Development recipe produces. I'd have to
guess that connection myself. See finding #2.

**2. What does the weekly update consist of?** This one's answered cleanly. `pvr app add
<name> --from <image> --platform <arch>` pulls the image, converts it to a SquashFS
rootfs, and writes `root.squashfs`, `run.json`, `lxc.container.conf`, and `src.json` into
a pvr checkout. Then `pvr add .`, `pvr commit -m "..."`, `pvr post http://<device-ip>:12368`
pushes it. On the device this downloads only the changed objects and, per the runtime
docs, does *not* reboot unless the change touches the BSP or a container with `system`
restart policy — and the default `app` group (where `pvr app add` containers land) has
restart policy `container`, i.e. non-reboot. So the weekly update is: one `pvr app add`
+ commit + `pvr post`, no rebuild, no reflash, no reboot.

**3. What is a "Docker image" here, and what has to be true of my app?** The FAQ answers
this directly: `pvr app add` "does not use the Docker runtime" — it merges the image's
layers into one SquashFS rootfs, and separately compiles the image's `ENTRYPOINT`/`CMD`,
`ENV`, `WORKDIR`, and `VOLUME` into the LXC config. In terms I know: it's functionally a
pre-built rootfs tarball (like what my recipe already produces) plus a small metadata
manifest telling LXC how to start it — not fundamentally different from what my BitBake
recipe outputs today. For it to work, my app has to already exist as a built Docker/OCI
image sitting in a registry (Docker Hub or private) — a `docker build`/`Dockerfile`
artifact, not just a Yocto recipe. If my app is currently *only* a Yocto recipe with no
Dockerfile anywhere, `pvr app add --from` has nothing to pull, and the docs don't say what
to do in that case (see finding #2 again — the recipe-built pvrexport path is the answer
but it's not connected here).

**4. What does this cost — flash, RAM, boot time, build time?** Partially answered.
`vs-docker` gives one real number: Pantavisor + LXC core footprint is "~1 MB" versus
"~50–100 MB for a typical [Docker] engine install" — but that's the runtime overhead, not
my app's added size. `vs-yocto`'s comparison table gives a build-time number: updating a
container takes "2–5 min" versus a "30–60 min cold" full image rebuild (minutes with
sstate-cache). But the dedicated page for exactly this question — Benchmarks — says
outright that the flash/RAM/boot-time/payload-size numbers aren't published yet. That's
the number I'd actually need to walk into my team lead's office with, and it isn't there.

**5. Tracing into the `pantavisor/` runtime docs — does it change answer (2)?** It
confirms and sharpens it rather than changing it. `pantavisor/overview/updates` documents
the full state machine (`QUEUED` → `DOWNLOADING` → `INPROGRESS` → `TESTING` → `DONE`, with
`WONTGO`/`ERROR` rollback paths) and spells out exactly when a reboot happens: BSP
changes, root-of-state-JSON changes, or any container with `system` restart policy.
`pantavisor/overview/containers` gives the default groups table, showing the `app` group
(where installed apps land by default) has restart policy `container` → non-reboot. So
my answer to (2) holds: a normal weekly app-only update is a live container
restart, not a reboot, and the runtime docs are what actually prove that rather than just
asserting it.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks` | B, step 4 | I need real flash/RAM/boot-time numbers to tell my team lead what this actually costs, and the one page built for exactly that question admits the numbers don't exist yet. | "The reproducible payload-size, update-time, and flash-write benchmarks (same hardware, same app change, image-updater vs Pantavisor) ship with the first public deliverable. Until then, treat the table above as the shape of the comparison, not as published figures." | S1 | `missing-concept` | Publish the promised measured numbers, or pull the comparison table until they exist so it doesn't read as if data is being withheld. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install` | B, steps 1 and 3 | My app is a rootfs directory installed by a recipe, not a Docker image. This page's only non-Docker option is "a pvrexport bundle," but nothing here or on its `local-pvr`/`local-pvtx` children says how to build one from an existing Yocto recipe instead of from a Docker image — the Container Development guide that does exactly that is never linked from here. | "You clone the device, add a container from a Docker Hub image or a pvrexport bundle, commit, and deploy... This is the recommended method for development and automation." (no link to `overview/container-development` anywhere on this page or its children) | S2 | `broken-path` | Link "a pvrexport bundle" on this page to the Container Development guide, which is the only place that shows how to produce one without a Docker image. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop` | B, step 3 | I know "layer" cold as a Yocto/OpenEmbedded layer. The docs also use it for a Docker image layer and for the weekly update payload itself, with no acknowledgment these are three different things. | "A CVE fix in your app ships as a small container layer — not a full-image flash" (vs. FAQ: "The image layers are merged into a single SquashFS rootfs") | S3 | `undefined-jargon` | Add a one-line disambiguation the first time "layer" is used for anything other than a Yocto layer, or use a different word for the update payload. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/meta-pantavisor` | B, step 1 | Didn't block me, but this "Layer Layout" page and the "Build System" page one click away both have a `PANTAVISOR_FEATURES` table and default-value string, and they disagree — I can't tell which is current. | `meta-pantavisor.md`: "**Default**: `dm-crypt dm-verity autogrow runc tailscale debug rngdaemon pvcontrol xconnect`" vs. `build-system.md`: "**Default**: `... xconnect container-mdev`" (plus a caution note about `tailscale`/`rngdaemon` gating that only appears on the second page) | S4 | `stale` | Deduplicate — keep one canonical `PANTAVISOR_FEATURES` table and have the other page link to it instead of repeating it. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | B, step 2 | Didn't block me — the on-page commands were enough — but the page's own "Official Documentation" section sends me off `docs.pantavisor.io` for the complete reference. | "**[PVR CLI Reference](https://docs.pantahub.com/pvr/)** - Legacy Pantahub documentation" | S4 | `outside-docs` | Fold any still-unique content from the legacy reference into this site's pages, or drop the dead-end link. |

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding:** the Benchmarks page — the one place built to answer "what does this cost in flash, RAM, boot time, build time" — states plainly that those numbers aren't published yet, leaving the exact question I came in skeptical about unanswered from the docs alone (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks).
- **What worked:** `getting-started/develop/application/install/local-pvr` for the mechanics of the weekly update (`pvr app add` → `pvr add .` → `pvr commit` → `pvr post`), and `pantavisor/overview/updates` + `pantavisor/overview/containers` together for tracing that update into the actual on-device reboot/no-reboot decision — between them they gave a complete, verifiable answer to steps 2 and 5.
- **Confidence:** high. I never used outside knowledge of Docker, LXC, or OCI — every claim about what a "Docker image" or "layer" means came from the FAQ and vs-docker page, and I logged the one place (recipe-to-container migration) where I'd have had to infer past what the docs actually connect for me.
