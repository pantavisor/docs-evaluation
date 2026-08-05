# Persona 04 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narrative

I have a Pi 4, an SD card, and a Docker Hub image. I want a runnable script and honest
answers to four questions, not a tour. Landed on `meta-pantavisor/getting-started/start`
via the top-nav "Getting Started" link. Three paths; "Download and flash on a Raspberry
Pi" matches my hardware, so I take it.

`download-and-flash` immediately hands me a real gap: step one is "Head over to our
[Downloads page](https://pantavisor.io/downloads/)" — that's off `docs.pantavisor.io`
entirely, a different domain, and I'm not supposed to follow off-domain links while
evaluating this site. So my script's first line can't actually come from the docs I was
sent to; I have to note the hop and assume the file lands with the name the rest of the
page already uses as a placeholder. Everything after that is clean and familiar:
`pvflasher` is a `curl | bash` installer exactly like a dozen tools I already use,
`pvflasher list` / `pvflasher copy ... /dev/sdX` reads like `dd` with guardrails, and
`bmaptool`/`dd` are offered as fallbacks I don't need. No build step anywhere on this
page. Good start, patience still full.

For the second half — get my own app on, not just flash the starter image — the "Next
steps" bullet says "Install your first application with `pvr`" but it's plain text, no
link. I fall back to the sidebar (visible on the rendered page, not the article-only
`.md` export) and find "Develop applications" listed as a sibling section to "Start."
While I'm there I also check the persistent top-nav's "meta-pantavisor" link out of
curiosity about what "meta-pantavisor" even is, and land on its overview page — which is
where I get the reassurance I actually care about: "Just want to deploy your own
container onto an existing device? Skip the build guide — grab a ready-made image...
and go straight to Develop applications." That sentence is exactly aimed at me. Small
paper cut: in the raw markdown, "Develop applications" at the end of that sentence isn't
a link at all — `...and go straight toDevelop applications .` — no href, no space. Didn't
stop me, because the sidebar already got me to the same place, but it's the one sentence
built to catch someone exactly like me, and it's broken.

"Develop applications" (`getting-started/develop`) links to an "Install apps" page
listing three methods. Method 1, "pvr CLI over the Local Network," is explicitly "the
recommended method for development and automation" and matches my workstation-over-network
mental model — but its own "→Install with pvr CLI" pointer is also plain text with no
href, a second dead arrow-link on the same page type. I only reach the actual walkthrough
(`application/install/local-pvr`) because I'd already seen it referenced from a different
page (`application/configure`, which I found while cross-checking the bugfix flow) and
because it's in the sidebar. A reader who takes only the broken arrow at face value is
stuck here.

`install/local-pvr` is the best single page on this whole path. It gives me the exact
install sequence, and — this answers question 3 outright — it states the mechanism:
"The `pvr app add` command pulls the image, converts it to SquashFS format... Docker
image configuration is compiled directly into `lxc.container.conf`: `ENTRYPOINT`/`CMD`
become init commands, `ENV` becomes environment variables, `WORKDIR` becomes the working
directory, and `VOLUME`s become mount entries." That's concrete and specific — I know
what happens to my image, and I know it isn't kept as-is (Docker layers become one flat
SquashFS), but the parts that matter functionally (entrypoint, env, workdir, volumes)
are explicitly preserved. This page never says "Yocto," "BitBake," or "kernel" once.

One real hole in `local-pvr`: step 1 is `pvr clone http://<device-ip>:12368/cgi-bin
mydevice` — nothing on this page says how I get `<device-ip>`. I only have a lead
because I separately skimmed the CLI reference page (`getting-started/develop/cli-tools/pvr-cli`)
for its install command and noticed `pvr device scan` described as "Scan for Pantavisor
devices on the local network (mDNS)" — no example run, no sample output, no confirmation
it's the address I plug into `clone`. I'd run it and hope, not because the docs told me
to.

For question 3's other half — does the crossing into the runtime's own docs happen — it
does, but not where I expected. The mechanism (image → SquashFS → LXC) is explained on
the meta-pantavisor side (`local-pvr`), not the runtime side. What I had to go to
`pantavisor/overview` and `pantavisor/overview/containers` for for was the *device-side*
behavior of that container once it exists: the default group table (`app` group =
`STARTED` status goal, `container` restart policy), and `pantavisor/overview/updates`
for what "container restart policy" actually triggers. That's where question 4 gets
answered solidly, and it's also where `getting-started/develop/application/configure`
sends me directly for the bugfix command: `pvr app update sensor-app --from
registry.example.com/sensor-app:v1.2.0`, followed by the same `pvr add .` /
`pvr commit` / `pvr post` I already used for install. `updates.md` and `containers.md`
both independently state the same thing in almost the same words: a full reboot happens
"only when a `system` restart-policy container or the BSP changed" — my app is in the
default `app` group (`container` restart policy), so the bugfix is a **non-reboot
transition**: only that one container stops and restarts, and if it fails to reach its
status goal, "the previous revision is restored automatically." That's a clean,
well-evidenced answer, cited twice independently.

### The script, as the docs would have me write it

```bash
# 1. Get the starter image (docs point off-site for this — see finding below)
#    -> https://pantavisor.io/downloads/  (raspberrypi-armv8, Pi 3B/3B+/4)
#    file: pantavisor-starter-raspberrypi-armv8.rootfs.wic.bz2

# 2. Install pvflasher and flash the SD card
curl -fsSL https://raw.githubusercontent.com/pantavisor/pvflasher/main/scripts/install.sh | bash
pvflasher list
sudo pvflasher copy pantavisor-starter-raspberrypi-armv8.rootfs.wic.bz2 /dev/sdX

# 3. Insert the SD card into the Pi, attach ethernet, power on.

# 4. Install pvr on the workstation
curl -sL https://gitlab.com/pantacor/pvr/-/raw/master/install.sh | bash

# 5. Find the device on the local network (output/format not documented — best effort)
pvr device scan

# 6. Install the app
pvr clone http://<device-ip>:12368/cgi-bin myapp
cd myapp
pvr app add myapp --from <dockerhub-user>/<image>:<tag> --platform linux/arm64
pvr add .
pvr commit -m "add myapp container"
pvr post http://<device-ip>:12368

# 7. Verify
pvcontrol container ls   # or: lxc-ls -f from the device console

# --- next week, the bugfix ---
cd myapp
pvr app update myapp --from <dockerhub-user>/<image>:<newtag>
pvr add .
pvr commit -m "bugfix: <what changed>"
pvr post http://<device-ip>:12368
```

### The four questions

1. **Could I actually run this?** Almost all of it, yes — `pvflasher`, `pvr install`,
   `pvr add`/`commit`/`post` all map cleanly onto Docker/git muscle memory I already
   have. The first command I genuinely can't type from the docs alone is the image
   download itself (step 1) — the docs never give a `curl`/`wget` for the real file,
   only a placeholder filename and an off-domain "Downloads page" link. The second soft
   spot is `<device-ip>` in step 6 — `pvr device scan` is named but never demonstrated.
2. **Did I have to build an OS?** No. Not once, on this path. `meta-pantavisor/overview`
   explicitly told me I could skip the build guide, and independently, the
   `getting-started/start` → `download-and-flash` → `develop` → `install/local-pvr` path
   never mentions Yocto, BitBake, KAS, or a kernel build at any step. I never got close
   to my hard stop.
3. **Where does my image go?** Docker Hub → `pvr app add --from` pulls it and converts
   it to a SquashFS rootfs; `ENTRYPOINT`/`CMD`/`ENV`/`WORKDIR`/`VOLUME` are compiled into
   `lxc.container.conf`, explicitly stated on `getting-started/develop/application/install/local-pvr`.
   It does cross into the runtime's own docs (`pantavisor/overview` and
   `pantavisor/overview/containers`) — but for the *device-side* lifecycle (groups,
   restart policy, status goals) once the container exists, not for the conversion
   itself, which stays entirely on the meta-pantavisor side.
4. **The bugfix:** `pvr app update <name> --from <image>:<new-tag>`, then the same
   `pvr add .` / `pvr commit` / `pvr post` sequence. The device does a **non-reboot
   transition**: only the affected container is stopped and restarted (my app is in the
   default `app` group, `container` restart policy), everything else keeps running, and
   if the new revision fails to reach its status goal the previous one is restored
   automatically. Stated independently and consistently on `pantavisor/overview/updates`,
   `pantavisor/overview/containers`, and `getting-started/develop/application/configure`.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start/download-and-flash` | B, script step 1 — download the image | I can't write a real command for the very first step: the docs send me off `docs.pantavisor.io` entirely to get the file, so my "runnable script" starts with a manual browser hop instead of a command. | "Head over to our [Downloads page](https://pantavisor.io/downloads/) to see all available platforms and images." | S2 | `outside-docs` | Host the current image filenames/checksums (or a direct, versioned download URL) on the docs page itself, or give a `curl`/`wget` one-liner alongside the browser link. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | B, script step 6 — `pvr clone http://<device-ip>:...` | I'm told to plug in `<device-ip>` with nothing on this page, or anywhere I reached from it, telling me how to get it. `pvr device scan` exists on a different page (found for an unrelated reason) but is never demonstrated — no example run, no sample output, no confirmation its result is what `clone` wants. | "`pvr clone http://<device-ip>:12368/cgi-bin mydevice`" alongside, on the CLI reference page, "Scan for Pantavisor devices on the local network (mDNS)" with no example output shown | S3 | `no-example` | Link the `<device-ip>` placeholder directly to a worked `pvr device scan` example showing sample output. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview` | B, step "just want to deploy an existing container" reassurance | The one sentence written specifically to reassure a reader like me that I can skip the build system has a dead link at the exact word I'd click. | "...grab a ready-made image via Starter Image / Flashing Images and go straight toDevelop applications . The build guide below is for building your own custom image from source." (no href on "Develop applications", missing space) | S4 | `broken-path` | Restore the markdown link on "Develop applications" to `getting-started/develop`. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install` | B, choosing an install method | The arrow link under the method explicitly called "the recommended method for development and automation" — the one I'd take — has no destination. | "→Install with pvr CLI" (plain text, no markdown link syntax) | S4 | `broken-path` | Restore the link on "Install with pvr CLI" to `application/install/local-pvr`. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | B, background reading while installing `pvr` | A "further reading" link on a page otherwise entirely inside `docs.pantavisor.io` points off-domain, to a different host than the one I was told to stay inside. | "PVR CLI Reference" → `https://docs.pantahub.com/pvr/` | S4 | `outside-docs` | Either mirror the reference under `docs.pantavisor.io` or drop the link if it's redundant with this page. |

## Closing summary

- **Task outcome:** completed with detours — I have a runnable script for install and
  bugfix-update, and cited answers to all four questions, but step 1 of the script isn't
  actually a docs-sourced command (off-domain download) and step 6 required a command
  I found by luck rather than by being told.
- **Worst finding:** the actual starter-image file for Prompt B's very first step is not
  hosted on `docs.pantavisor.io` at all — `getting-started/start/download-and-flash`
  sends the reader to `https://pantavisor.io/downloads/`, a different domain, with no
  `curl`/`wget` alternative given on the docs page itself.
- **What worked:** `getting-started/develop/application/install/local-pvr` — it gave me
  the exact install and (via `configure`) update commands, and it's the one page that
  fully answered "does my image survive intact," stating plainly that Docker
  `ENTRYPOINT`/`CMD`/`ENV`/`WORKDIR`/`VOLUME` are compiled into `lxc.container.conf`
  rather than silently dropped.
- **Confidence:** high — stayed in character throughout. Where the mechanism crosses
  into `pantavisor/overview/containers` and `pantavisor/overview/updates` I only used
  what those pages stated, not outside knowledge of LXC or container runtimes; I flagged
  the device-IP step as a guess rather than presenting it as documented.
