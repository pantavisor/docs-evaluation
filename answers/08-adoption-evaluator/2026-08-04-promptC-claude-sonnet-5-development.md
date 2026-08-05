# Persona 08 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: manual (Claude Code session)

## Jargon audit — every product, service, or component name met

Read path: `meta-pantavisor/getting-started/start` → `pantavisor/overview` (index) →
`meta-pantavisor/overview` (index) → `meta-pantavisor/overview/glossary` →
`pantavisor/overview/remote-control` → `pantavisor/overview/local-control` →
`pantavisor/overview/init-mode` → `meta-pantavisor/getting-started/benchmarks` →
`meta-pantavisor/overview/images` → `meta-pantavisor/getting-started/develop/cli-tools/pvr-cli`
→ `meta-pantavisor/getting-started/how-to-install/docker` →
`meta-pantavisor/overview/get-started` (link inventory) →
`meta-pantavisor/overview/flashing-images` → `meta-pantavisor/getting-started/migrate-to-pantavisor` (404).

| Name | First use (page URL) | What is it? | Open source or commercial? | Optional or required? | Where do the docs say? |
|---|---|---|---|---|---|
| **Pantavisor** | [.../meta-pantavisor/getting-started/start](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) | The core runtime (PID 1) — the product itself. | Open source — the only explicit statement I found. | Required — it's the thing being adopted. | [.../getting-started/benchmarks](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks): comparison table row "Open source \| ✅ 100%" under the "Pantavisor" column. |
| **meta-pantavisor** | [.../meta-pantavisor/overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) | The Yocto/OpenEmbedded layer that builds Pantavisor BSP images. | NOT STATED | Optional if you use a prebuilt starter image; required only to build your own. | "Just want to deploy your own container onto an existing device? Skip the build guide... grab a ready-made image." |
| **pvr** | [.../getting-started/start](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) | Workstation CLI, git-like semantics for device state. | NOT STATED | Required — "your primary interface for managing Pantavisor repositories and devices." | [.../develop/cli-tools/pvr-cli](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli) |
| **Pantahub / Pantacor Hub** | [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | Optional hosted cloud backend: device claiming, fleet management, remote updates, log streaming. | NOT STATED — no price, license, or source-availability statement found anywhere reachable. | Optional — "Devices work fully without it." | glossary.md "Pantahub" entry |
| **Container** (Pantavisor's own) | [.../meta-pantavisor/overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) | An LXC container that's part of device state (squashfs rootfs + `run.json`). | NOT STATED for the mechanism itself (see LXC below). | Required — the unit everything ships as. | glossary.md "Container" entry |
| **LXC / lxc-pv** | [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | Upstream Linux Containers project; Pantavisor runs a fork of it (`lxc-pv`). | Upstream LXC: open source, stated. The fork (`lxc-pv`) itself: NOT STATED. | Required — core runtime mechanism, not swappable per docs. | "the upstream OS-level container project... Not a Pantavisor invention... via a Pantavisor-specific fork (`lxc-pv`)." |
| **BSP** | [.../pantavisor/overview](https://docs.pantavisor.io/development/pantavisor/overview) | Board Support Package — kernel, modules, firmware, versioned with every revision. | NOT STATED | Required — every device/revision needs one. | pantavisor/overview index item 3 |
| **AppEngine** | [.../getting-started/start](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) | Pantavisor packaged to run in Docker on a workstation — dev/CI only, no hardware. | NOT STATED | Optional, explicitly non-production — "designed for container engine prototyping, development, and testing rather than production deployments." | [.../how-to-install/docker](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker) |
| **Pantabox** | [.../pantavisor/overview/local-control](https://docs.pantavisor.io/development/pantavisor/overview/local-control) | Top-level ncurses control UI for a device, run inside a container. | NOT STATED — GitLab repo link only, no license shown. | Optional — one of several ways to talk to the local control socket. | local-control.md "Pantabox" section |
| **pvcontrol** | [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | On-device CLI for the pv-ctrl REST API. | NOT STATED | Optional — "you can always directly use cURL or any other HTTP client" instead. | glossary.md "pvcontrol" entry; local-control.md |
| **pvtx** | [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | On-device update-transaction tool; also serves the device web UI on port 12368. | NOT STATED | Presented as one of three interchangeable options. | images.md: "`pvtx`, the device web UI, or `pvr` can post a new pvrexport to an already-running device." |
| **pvr-sdk** | [.../pantavisor/overview/local-control](https://docs.pantavisor.io/development/pantavisor/overview/local-control) | Development-platform container bundling Pantabox + pvcontrol; shipped by default. | NOT STATED — GitLab-hosted, no license shown. | Optional — shipped by default but removable. | images.md "Customizing the mix" — `PVROOT_CONTAINERS_CORE` can drop `pv-pvr-sdk`. |
| **pvflasher** | [.../meta-pantavisor/overview/flashing-images](https://docs.pantavisor.io/development/meta-pantavisor/overview/flashing-images) | GUI+CLI flashing tool, distributed off-site via `pantavisor.io/downloads`. | Open source, explicitly — "Pantacor's open-source flashing tool (GUI and CLI, Linux/Windows/macOS)." | Optional — `dd` is offered as an alternative for most boards. | flashing-images.md |
| **pv-flash-bundle** | [.../meta-pantavisor/overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) | UUU factory-flash archive recipe for NXP/Toradex/Variscite boards. | NOT STATED | Conditionally required — only for boards without native `.wic` flashing. | flashing-images.md flashing-method table |
| **kas** | [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | YAML build-orchestration tool wrapping BitBake. | NOT STATED on this site (links off-domain to `github.com/siemens/kas`, not followed per scope fence). | Required only if building your own image from source. | glossary.md "kas" entry |
| **BitBake** | [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | Build engine `kas` wraps; executes recipes. | NOT STATED | Required only for building from source. | glossary.md "BitBake" entry |
| **Yocto / OpenEmbedded** | [.../meta-pantavisor/overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) | The build-system project/ecosystem meta-pantavisor targets. | NOT STATED — glossary defines OpenEmbedded's role but not either project's license or governance. | Required only if building from source. | glossary.md "OpenEmbedded" entry: "These docs use 'Yocto' and 'OpenEmbedded' interchangeably." |
| **Pantacor** (the company) | [.../meta-pantavisor/overview/flashing-images](https://docs.pantavisor.io/development/meta-pantavisor/overview/flashing-images) | Never directly introduced — inferred to be the vendor behind Pantavisor, Pantahub, and pvflasher purely from possessive mentions. | N/A — it's the vendor, and its relationship to the open-source pieces is never spelled out. | N/A | "Pantacor's open-source flashing tool"; footer link "Pantacor Website" (off-domain, unexplained) on every page. |
| **Docker / Docker Hub** | [.../getting-started/start](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) | Third-party container engine; Docker Hub is the default source for app images added via `pvr app add`. | NOT STATED on-site (well known, but undiscussed). | Docker itself optional (AppEngine/dev only); Docker Hub is the default but not stated as the only image registry. | pvr-cli.md "Tips and Best Practices": "Applications are pulled from Docker Hub by default." |
| **GitHub** (`github.com/pantavisor`) | sidebar link, e.g. on [.../meta-pantavisor/overview/get-started](https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started) | Presumably where the runtime/layer source lives — never introduced as such. | Implied open (public org) but never stated as "this is the source." | N/A | Sidebar "GitHub" link on every meta-pantavisor page. |
| **GitLab** (`gitlab.com/pantacor`) | [.../develop/cli-tools/pvr-cli](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli) | Hosts `pvr`, `pvr-sdk`, and Pantabox source/releases under a `pantacor` namespace distinct from the GitHub org above. | NOT STATED (company-owned namespace; per-repo license not shown on the docs site). | N/A | pvr-cli.md install script link; local-control.md Pantabox/pvr-sdk links. |

**Can I draw the line between what's free-and-mine-forever and what's a vendor
relationship?** No — I have exactly two explicit open-source anchors (Pantavisor
itself, in a comparison table buried three clicks into the build docs, and pvflasher,
named outright) and one explicit optionality anchor (Pantahub: "devices work fully
without it"), but the actual vendor relationship — what Pantahub costs or requires,
what license covers `pvr`/Pantabox/pvr-sdk/meta-pantavisor, and even who or what
"Pantacor" is — is never stated anywhere I could reach, so the line stays blurred
exactly where it matters most: the hosted Hub my fleet would depend on.

## Closing summary

- **Task outcome**: completed.
- **Worst finding**: S1 — Pantahub / Pantacor Hub, the one component this persona's
  "what am I locked into" question is actually about, has its commercial status
  (pricing, license, self-hostability) stated nowhere reachable on the docs site; the
  only thing stated is that it's optional, not what it costs to depend on.
- **What worked**: [.../meta-pantavisor/getting-started/benchmarks](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks) gave the one
  unambiguous open-source claim for Pantavisor itself, and
  [.../meta-pantavisor/overview/glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) was the single most information-dense page,
  defining nearly every proper noun encountered in one place.
- **Confidence**: high — every row above is cited from a page fetched under
  `/development/`; I did not use prior knowledge of Pantavisor, Pantacor, LXC, Yocto,
  or Docker beyond what these pages themselves stated, and I did not follow any
  off-domain link (hub.pantacor.com, github.com, gitlab.com, pantavisor.io/downloads,
  docs.pantahub.com) that appeared in the fetched content.
