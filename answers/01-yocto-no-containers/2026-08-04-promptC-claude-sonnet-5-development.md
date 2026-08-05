# Persona 01 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Jargon audit

Read order followed: [overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) →
[glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) (via the
inline "container" link on the overview page — the natural first move for a reader who's just
told they've never touched containers) → [get-started](https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started)
→ [supported-device](https://docs.pantavisor.io/development/meta-pantavisor/overview/supported-device)
→ [pantavisor-development](https://docs.pantavisor.io/development/meta-pantavisor/overview/pantavisor-development)
→ [container-development](https://docs.pantavisor.io/development/meta-pantavisor/overview/container-development).
Stopped there — by that point every remaining page (manifest-audit, component-docs, bootchartd)
is further from the question that actually matters to me (what does a container cost me, and
what *is* one), and I'd already hit the one term I couldn't get past: LXC is used to define
"Container" itself and is never once defined.

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
|---|---|---|---|---|
| Trail / "device trail" | [overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) — "...composes core containers with the BSP into the initial device trail" | Yes — [glossary#trail](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary): "The ordered history of a device's revisions — `trails/` on the device's storage, mirrored per device on Pantahub." | I guessed "trail" meant some kind of build-output directory or log path — a trace left behind by the build, like a Yocto `tmp/` dir. | No. It's a revision-history concept, closer to a git log of device states, not a filesystem artifact of the build. Without hunting down the glossary I'd have kept treating it as a build byproduct. |
| AppEngine | [get-started](https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started) — Common Build Targets table: "`pantavisor-appengine` \| Docker-based appengine image" | Yes — [glossary#appengine](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary): "Pantavisor packaged to run inside a Docker container on an x86 workstation — a full device without hardware, used for development and CI." | I assumed this was some kind of cloud hosting/PaaS thing — like Google App Engine, somewhere you deploy *to*, not something you run locally. | No. It's a local dev/CI stand-in for real hardware, run in Docker on my own workstation. Nothing on the get-started page itself hints at that — without the glossary the name alone would have actively misled me. |
| "image" (Docker sense) | [get-started](https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started) — `docker load < build/tmp-scarthgap/deploy/images/docker-x86_64/pantavisor-appengine-docker.tar` | No. "Image" is used constantly (BSP image, starter image, Docker image) but nothing on any page I reached ever flags that a Docker image is a structurally different thing from a Yocto/BSP image. | I assumed it was the same kind of object as the BSP image discussed two paragraphs earlier — a flashable rootfs bundle, just built for x86 instead of my target board. | No, and I still don't fully know from these docs alone. A `docker load` image is a layered thing loaded into a running Docker daemon, not something you flash — but that distinction is never called out anywhere I read, so the correction had to come from outside knowledge, not the page. |
| LXC | [glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary), inside the **Container** entry: "An LXC container defined as a part of the device state..." | No. It's the load-bearing word inside the definition of "Container" itself, and it reappears later (`lxc-ls -f` in the smoke test, `lxc-pv/ # LXC with pantavisor patches` in the workspace layout) but is never once expanded, linked, or explained anywhere I reached. | Since it's used to *define* "container" rather than the other way round, I assumed LXC must be some Pantavisor-internal packaging format — I had no way to know it's a general-purpose, vendor-neutral Linux container runtime that predates Docker. | No, and the docs never correct it. As far as anything I read is concerned, LXC could be proprietary to Pantavisor. This is the term I'd most embarrass myself on in a conversation with someone who assumed I already knew it. |
| Container | [overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) — "...producing initramfs images and [container](.../glossary#container) pvrexport bundles" | Yes — [glossary#container](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary): "An LXC container defined as a part of the device state: a squashfs root filesystem plus a `run.json` runtime manifest..." | Literally pictured a shipping container and waited for it to be explained. Once I saw "squashfs" and "manifest" my working guess became: a packaged filesystem bundle, not a running process. | Half right. It is a filesystem-plus-manifest bundle, which matches my guess — but the definition leans entirely on the undefined "LXC" to cover the part where it's also a *running, isolated process*. I got the storage shape right and the runtime behavior is still a blank. |
| pvrexport | [overview](https://docs.pantavisor.io/development/meta-pantavisor/overview) — "...producing initramfs images and container pvrexport bundles" (plain text, not linked at first use) | Yes — [glossary#pvrexport](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary): "A tarball of one or more state parts (an app or a BSP) produced by `pvr export`... deployable onto any device via pvtx, the web UI, or `pvr`." | Guessed it was some kind of exported build artifact tarball — the container equivalent of a Yocto deploy image. | Yes, close enough. My Yocto instinct ("export = packaged build output") carried over cleanly here. |
| `--privileged` (Docker flag) | [get-started](https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started) — `docker run --name pva-test -d --privileged ...` | No — it appears in every smoke-test / dev-workflow snippet across three pages (get-started, pantavisor-development) and is never explained anywhere. | Guessed from the word alone: "runs with elevated permissions inside the container." Pure guess — I've never typed `docker run` before. | Unverifiable from the docs. Nothing here confirms or corrects the guess, and since Pantavisor containers turn out to be LXC underneath, I genuinely don't know whether this flag is doing something specific to nested containers or is just boilerplate copy-pasted into every example. |

### Absence of a finding

Everything Pantavisor-native that has a glossary entry — BSP, Revision, State, Status goal,
pv-ctrl, pvcontrol, pvr, pvtx, xconnect, Auto-recovery — checked out fine once I found the
glossary: clear, short, no circular jargon inside the definitions themselves (unlike Container).
The glossary page is doing real work; the gap is entirely that the container-specific vocabulary
it borrows from the wider container ecosystem (LXC, Docker image semantics, `--privileged`) is
treated as already known rather than defined even once.

## Closing summary

- **Task outcome:** completed.
- **Worst finding:** LXC is the term used to define Pantavisor's core "Container" concept — the
  single most important idea for this persona — and it reappears in commands (`lxc-ls`) and
  source-tree names (`lxc-pv/`), but it is never defined, linked, or expanded anywhere reachable
  under `development/`; a reader with the stated zero container background has no way to learn
  what runtime substrate their apps actually run on. This is S1 by the rubric's own test: the
  answer isn't hard to find, it isn't on the site at all.
- **What worked:** the [glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary)
  page — once located via the inline "container" link on the overview page, it correctly and
  concisely defined the large majority of Pantavisor-specific vocabulary (Trail, AppEngine,
  pvrexport, BSP, Revision, State, Status goal, xconnect, pv-ctrl, pvcontrol, pvr, pvtx) in one
  place.
- **Confidence:** I stayed in character for every "what I assumed it meant" guess — those are
  written as a container-naive Yocto engineer would actually guess, not with hindsight. Grading
  the "Was I right?" column, though, necessarily used my own real knowledge of what LXC and Docker
  images actually are, since that's the only way to score the docs' silence against ground truth;
  I did not use that knowledge to shortcut navigation or invent an explanation the docs don't
  give.
