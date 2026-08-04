# Persona 03 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narrative

Landed on `meta-pantavisor/getting-started/start`, exactly where the prompt puts me.
Skimmed for the code block — there isn't one, just two "Paths": flash a Raspberry Pi, or
run Pantavisor in Docker if I have no hardware. I was handed an actual device product, so
I took the real-hardware path: "Download and flash on a Raspberry Pi."

That page is a long, competent flashing guide (`pvflasher`, `bmaptool`, `dd`, a whole
"finding your SD card's device name" section) — none of it blocked me, it's just not my
job; I'd hand this to whoever racked the device. First Kubernetes assumption stated out
loud: I expected "boot the device" to be the whole story, the way a pod just starts. This
page instead front-loads USB-to-TTL serial adapters and GPIO pin wiring for console
access — new territory, but the page doesn't require me to use it, just recommends it for
debugging, so I kept moving. I followed "Install your first application with `pvr`" under
Next Steps, since that's my actual goal.

That landed me on "Install apps," which opens with the single most important correction
in this whole run, stated as fact: "Every application on a Pantavisor device is an LXC
container added to the device's revision trail." Not a Docker container, not a pod — an
LXC container in something called a "trail." My `docker run my-image` instinct dies right
here, on the page, on purpose. Credit where due. Three install methods are offered — CLI,
web UI, remote Pantahub — and "This is the recommended method for development and
automation" next to the `pvr` CLI option is exactly the sentence a CI-pipeline person
like me is looking for, so I took it.

"Install with the pvr CLI" is a real, runnable five-step walkthrough: clone the device,
`pvr app add <name> --from <image> --platform <arch>`, stage, commit, `pvr post`, verify.
Unlike a "docker run," I'm operating on a git-like checkout of the device's state on my
own workstation, not on the device directly — that's the second correction, and it's
clean and explicit this time (not accidental). The one runnable example is
`pvr app add tailscale --from tailscale/tailscale --platform linux/arm64` — a public
Docker Hub image. By analogy I can write my own command:

```bash
pvr clone http://<device-ip>:12368/cgi-bin mydevice
cd mydevice
pvr app add api --from registry.example.com/team/api:v2.1 --platform linux/arm64
pvr add .
pvr commit -m "add api service"
pvr post http://<device-ip>:12368
```

Two things I wasn't sure of. First: is `--from` even allowed to point at a host other
than Docker Hub? The CLI reference page (linked from this page's Prerequisites as the
`pvr` install guide) states outright, under Best Practices, "Applications are pulled from
Docker Hub by default" — and every `--from` example on both that page and this one is a
bare Docker Hub short name (`nginx:stable-alpine`, `tailscale/tailscale`). I went looking
for a counter-example and found one, but not where I expected it: the sibling "Configure
Application Settings" page — which I only opened because I was still hunting for
registry syntax, not because anything on my install path pointed me there — has
`pvr app update sensor-app --from registry.example.com/sensor-app:v1.2.0`. So a
registry-qualified `--from` does work, syntactically. Nobody on the actual "install a new
app" path shows or says so.

Second, bigger: our image lives in a private company registry. Assuming `--from` accepts
the host (confirmed above, eventually), does the pull authenticate against it at all? I
read the local-pvr walkthrough, the CLI reference, the Configure page, and the Access page
looking for a credentials step — `pvr login`, a config file, an env var, anything. The
only login command mentioned anywhere (`pvr login`) is explicitly scoped to the Pantahub
cloud API for claiming/managing devices, not to arbitrary image registries. Nothing tells
me how — or whether — `pvr app add` authenticates to `registry.example.com`. I know Docker
registries cold; I'm filling this with "surely it behaves like `docker pull` and reuses my
local credentials somehow" purely because that's my own prior, not because the docs said
it.

While checking the Access and Remove pages for rollback (my other standing Kubernetes
question — "rollback is a rollout undo, right?"), I found the answer is close to yes: a
removed or bad revision is "kept in the trail," restorable with
`pvcontrol cmd run <revision>`. "Trail" is the same undefined load-bearing word from the
Install apps page, still never defined by the time I've read five pages that use it.

I did not hit a hard wall — I have a command sequence I believe is syntactically correct.
But I would not run it against our real private-registry image without first confirming
the auth question with a colleague; guessing wrong here risks a bad revision on a device
nobody local can fix in person, a cost my Kubernetes instincts don't naturally price in.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | A — constructing `pvr app add` for my own private-registry image | I'm about to run `pvr app add api --from registry.example.com/team/api:v2.1` against our company's private registry, but nothing on this page, the CLI reference, or the Configure page says how (or whether) the pull authenticates — I'm assuming it reuses local Docker credentials purely because that's my own prior. | "The `pvr` CLI lets you add a Docker Hub image as a Pantavisor container... all without touching the device directly." No credential step anywhere in the five-step walkthrough. | S1 | `missing-concept` | State explicitly how `pvr app add`/`update` authenticates against a private/protected registry. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | A — checking whether `--from` accepts a non-Docker-Hub registry host before I run it | This page and the install walkthrough both imply Docker Hub is the only source — every `--from` example is a bare short name, and I only confirmed a registry-qualified `--from` works by finding an unlinked example on a different page (Configure) meant for updating an existing app, not installing a new one. | "Applications are pulled from Docker Hub by default." — only `--from` examples here: `nginx:stable-alpine`, `nginx:latest`. The registry-qualified example, `pvr app update sensor-app --from registry.example.com/sensor-app:v1.2.0`, lives only on the Configure page and is never cross-linked from here or from local-pvr. | S2 | `broken-path` | Add or link a `--from <registry-host>/<image>:<tag>` example directly on the install walkthrough / CLI reference. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/remove` | A — checking whether a bad deploy is recoverable (my "rollback is a rollout undo" assumption) | Rollback is described in terms of "the trail," a term I've now hit on two different pages with no definition — I'm guessing it behaves like git history, but the docs never say. | "The previous revision (with the container) is kept in the trail: to restore it, roll back by running that revision on the device (`pvcontrol cmd run <revision>`)." | S3 | `undefined-jargon` | Define "trail" on first use, or link it to wherever it's defined. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | A — looking for the complete `pvr app add` flag reference | I wanted the full flag list for `pvr app add` and the page's own "Official Documentation" pointer sends me off `docs.pantavisor.io` entirely, to a domain I'm not allowed to follow and that's flagged as legacy. | "For complete command reference and advanced usage: ... [PVR CLI Reference](https://docs.pantahub.com/pvr/) - Legacy Pantahub documentation" | S4 | `outside-docs` | Host the complete `pvr app add` flag reference on `docs.pantavisor.io`, or drop the stale off-domain pointer. |

## Closing summary

- **Task outcome:** completed with detours — I have a command sequence I believe is syntactically correct for `registry.example.com/team/api:v2.1`, assembled by combining the official install walkthrough's worked example with a registry-host syntax I had to confirm on an unrelated page.
- **Worst finding:** nothing anywhere in this path — including the CLI reference and the Configure page — explains how `pvr app add`/`update` authenticates against a private registry, forcing a guess that it reuses local Docker credentials (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr).
- **What worked:** the "Install apps" page's opening line ("Every application on a Pantavisor device is an LXC container added to the device's revision trail") and the "Install with the pvr CLI" five-step walkthrough (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr) are the clearest, most deliberate corrections to my Kubernetes mental model in the whole run — no control plane, a local git-like checkout instead — stated on purpose, not discovered by accident.
- **Confidence:** high that I stayed in character on Pantavisor-specific concepts (never claimed to know "trail," LXC, or the registry-auth mechanism going in). I leaned on real prior knowledge of Docker registries/credentials only to name the gap ("surely it behaves like `docker pull`"), which is in-bounds per the persona card, and flagged it explicitly rather than passing it off as something the docs told me.
