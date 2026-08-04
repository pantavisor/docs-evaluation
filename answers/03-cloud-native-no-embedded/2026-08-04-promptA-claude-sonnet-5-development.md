# Persona 03 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: sub-agent

## Narrative

Landed on `meta-pantavisor/getting-started/develop` as instructed. No code block on
the page at all — first surprise, since I skim for one — just two bulleted lists under
"Manage applications" and "Tooling and deeper development." The opening paragraph
already assumes I know what "the base" and "a full-image flash" are ("never touches the
certified base"); I let that one slide since it didn't block anything downstream.

My task is literally the first bullet: "Install apps — add containers with the `pvr`
CLI, the pvtx web UI, or remotely via Pantahub." Kubernetes assumption #1, stated out
loud: I expect this to be one `kubectl apply`-shaped command. But that bullet — unlike
every other bullet on the page — isn't a link. "CLI tools" right below it, which names
`pvr`/`pvtx`/`pvcontrol` directly, isn't linked either. The only clickable things are
Configure / View / Access / Remove (all about apps that already exist) and two
Yocto/BSP-flavored links I have no reason to click for this task.

I clicked "Configure" as the closest match — "overlay files and runtime manifests
through the revision workflow" sounded like it might mean "add." It's a well-built
five-step page (clone → edit → commit → deploy), and it's the first real correction to
my Kubernetes mental model: there's no control plane and no scheduler here, I `pvr
clone` the device's state to my own workstation like a git checkout, edit it locally,
and `pvr post` it back. That's a genuine, well-explained paradigm shift and the page
gets credit for it. But every command on it operates on a container that's already in
the checkout (`sensor-app`) — the one runnable example is `pvr app update`, not `pvr
app add`. `pvr app add` is only *named*, once, in a sentence about environment
variables, with no example.

I kept going — View (read-only, no add), then Access, which is nominally about
reaching a running app but which happens to contain, inside a Tailscale walkthrough, the
only full `pvr app add` invocation I found anywhere in this path:
`pvr app add tailscale --from tailscale/tailscale --platform linux/arm64`. That's how I
actually learned the syntax — by accident, in a section about something else entirely.
By analogy I can now write my own command:

```bash
pvr clone http://<device-ip>:12368/cgi-bin my-device
cd my-device
pvr app add api --from registry.example.com/team/api:v2.1
pvr add .
pvr commit -m "add api service"
pvr post http://<device-ip>:12368
```

Two things stopped me from being confident about actually running this. First,
`--platform`: the Tailscale example uses `--platform linux/arm64`, the Configure page's
`app update` example uses no platform flag at all — I don't know if it's required for my
image or how I'd find the right value for my device. Second, and bigger: our image lives
in `registry.example.com/team/api`, a private company registry. Nothing on any page I
reached — Configure, View, Access, Remove — says one word about registry
authentication. Access does clarify these are commands "run on your workstation," which
at least tells me the pull happens locally via `pvr`, not on the device itself — so my
Kubernetes assumption of an `imagePullSecret` on the cluster side is wrong, and the docs
do correct it, just not on purpose. But whether `pvr` reuses my local Docker credentials,
needs its own login step, or something else — silence. I know Docker registries cold;
I'm filling this gap with "surely it behaves like `docker pull`" purely because that's
my own prior, not because the docs told me so.

Rollback is the other Kubernetes-shaped question I went looking for
("rollback is a rollout undo," right?). The Remove page answers it, sort of: a failed or
unwanted revision is "kept in the trail," restorable with `pvcontrol cmd run
<revision>`. "Trail" is used as a load-bearing term there and nowhere in anything I read
is it defined — is it a git-style history, a fixed-depth ring buffer, something else? I
can guess (git background helps here) but the docs never say.

I did not literally hit a wall — I have a command I believe would work for the
happy-path (non-private-registry) case, and a rollback mechanism I can name even if I
can't fully explain it. But I would not run this against our actual private image without
first confirming the registry-auth question with a colleague; that's the one thing here I
would escalate rather than guess through, since guessing wrong risks a broken device with
nobody local to fix it in person — a cost my Kubernetes-trained instincts don't
naturally price in.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/access-applications` | A — constructing the `pvr app add` command for my own image | I'm about to run `pvr app add api --from registry.example.com/team/api:v2.1` against our private company registry, but nothing on any page I reached says how the pull authenticates — I'm assuming it reuses my local Docker credentials purely because that's how `docker pull` works, not because the docs said so. | `pvr app add tailscale --from tailscale/tailscale --platform linux/arm64` — the only full `pvr app add` example anywhere in this path, using a public image, with no credential step before or after. | S1 | `missing-concept` | State explicitly how `pvr app add`/`update` authenticates against a private registry. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop` | A — landing page, choosing where to click first | The very first bullet, "Install apps," is exactly my task, but it's the only bullet under "Manage applications" with no link — I had to guess and click "Configure" instead, which only shows how to update an app that already exists. | "- Install apps — add containers with the `pvr` CLI, the pvtx web UI, or remotely via Pantahub" — no `[]()` markup, unlike the four bullets immediately below it. | S2 | `broken-path` | Link "Install apps" to wherever `pvr app add` is actually demonstrated. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop` | A — evaluating my two install options | As someone who thinks in fleets, not single boxes, "remotely via Pantahub" is the option I'd actually want for a real deployment, but it's never a link anywhere I could click, so I fell back to the local device-IP workflow instead. | "add containers with the `pvr` CLI, the pvtx web UI, or remotely via Pantahub" — plain text, not linked. | S2 | `missing-concept` | Link "Pantahub" here to whatever page explains the fleet/remote deploy path. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/access-applications` | A — constructing the `pvr app add` command for my own image | One example uses `--platform linux/arm64` and another (`app update`) uses no platform flag at all — I don't know if I need it for my image, or what value to use if I do. | `pvr app add tailscale --from tailscale/tailscale --platform linux/arm64` vs. `pvr app update sensor-app --from registry.example.com/sensor-app:v1.2.0` (no `--platform`). | S3 | `missing-concept` | State when `--platform` is required and how to determine the value for a given device. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/remove` | A — checking whether I could recover from a bad deploy | Rollback is described in terms of "the trail," a word I've now seen twice with no definition — I'm guessing it works like git history, but the docs never say. | "The previous revision (with the container) is kept in the trail: to restore it, roll back by running that revision on the device (`pvcontrol cmd run <revision>`)" | S3 | `undefined-jargon` | Define "trail" on first use, or link it to wherever it's defined. |

## Closing summary

- **Task outcome:** completed with detours — I have a command sequence I believe gets `registry.example.com/team/api:v2.1` running, assembled by combining an unrelated Tailscale example with the Configure page's update workflow, not from anything written for "install a new app."
- **Worst finding:** nothing anywhere in this path explains how `pvr app add`/`update` authenticates against a private registry, forcing me to guess it reuses local Docker credentials — https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/access-applications.
- **What worked:** the Configure page's five-step clone → edit → commit → deploy workflow (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/configure) is the clearest single correction to my Kubernetes mental model — no control plane, a local git-like checkout of device state instead — even though it never shows the one command (`pvr app add`) I actually needed.
- **Confidence:** high that I stayed in character on Pantavisor-specific concepts (never claimed to know "trail," "base," or the registry-auth mechanism). I did lean on real prior knowledge of Docker registries/credentials to fill the auth gap — that's in-bounds per the persona card ("Docker... registries, tags, digests" is knowledge I have cold), and I flagged every place I did it rather than passing it off as something the docs told me.
