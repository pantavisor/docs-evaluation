# Persona 04 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narrative

Landed on `meta-pantavisor/getting-started/start` cold. Three paths listed up top; I
have a Pi in a drawer, so "Download and flash on a Raspberry Pi" is the obvious click —
skip the no-hardware Docker path, skip reading the `pvr`-install bullet for now since I
don't need it yet. Patience: full, this looks like a normal getting-started page.

`download-and-flash` is genuinely reassuring at first read: download a **pre-built**
image, flash it with a tool called `pvflasher` that's just a `curl | bash` one-liner
like a hundred other CLI tools I've installed, then `pvflasher copy` to the SD card. No
mention of building anything. The explicit note lower down — "Pantavisor manages the
entire device update itself. There is no separate A/B image updater to install or
configure underneath it" — is exactly the kind of reassurance I want, even if I don't
yet know what it's contrasting itself with. Boot steps are plug-and-play; the serial
adapter is flagged "optional but recommended," and since I don't own one, I'm noting
that as a risk (no serial to debug a bad boot) but not stopping for it — I have an
ethernet cable and that's what the guide says is recommended for first boot anyway.

The page's own "Next Steps" points at "Install your first application" — that's my
goal, so I click it. First sentence on the landing page: "Every application on a
Pantavisor device is an **LXC container** added to the device's **revision trail**."
Neither term is one I know. I write Docker containers; I don't know what LXC is, and
"revision trail" sounds like it might be this platform's answer to a git history but
nothing here says so. Small patience dip here — not a stop, just a flag that the
platform is quietly not what I assumed it was.

Three install options; "pvr CLI over the Local Network" is explicitly called "the
recommended method for development" and matches my workstation-over-network mental
model exactly, so I take it. The page requires `pvr` installed first, so I follow the
prerequisite link to the CLI reference/install page — another `curl | bash` one-liner,
comfortable, done in ten seconds. While I'm on that page anyway (I'm here for the
install command, not browsing), I keep half-scanning and notice a "Device Operations"
section further down with `pvr device scan` for finding devices on the local network via
mDNS. I file that away, because I have a feeling I'm about to need a device IP and
nothing on the page I came from has told me how to get one yet.

Back on the install walkthrough: step 1 is `pvr clone http://<device-ip>:12368/cgi-bin
mydevice`. Exactly the case I just flagged — this page never says how to find
`<device-ip>`. If I hadn't happened to skim the CLI reference page for an unrelated
reason a minute earlier, I'd be stuck here typing something into a search engine. Steps
2–3 (`pvr app add`, `pvr add .`, `pvr commit`) feel exactly like Docker build + git
commit stapled together — completely legible to me, no complaints. Step 4, `pvr post`,
is the actual deploy, and its explanation mentions a reboot happens "when a `system`
restart-policy container or the **BSP** changed." Another undefined term, and this one
lands closer to my hard-stop nerve — BSP sounds like exactly the embedded/kernel
territory I refuse to enter — but the sentence's own logic is that BSP changing is the
exception, not what I'm doing, so I read past it rather than stopping. Step 5 (verify)
offers `pvcontrol` over the network as an alternative to serial/SSH, which matters to
me specifically because I don't have a serial adapter — good, this route doesn't
require touching the device.

That's install done. For the second half of my goal — update later, without touching
the device — I go back to the CLI reference page I already have open, since that's
where I saw command references before. There's an "Update Applications" section:
`pvr app update nginx-app`. One line, no walkthrough. Compare that to the three
explicit, separate steps the install page walked me through (add → stage/commit →
post) — this just shows the update command in isolation, with nothing confirming
whether I still need `pvr add . && pvr commit` and `pvr post` afterward the same way.
I'm fairly confident the same pattern applies by analogy, but "fairly confident" isn't
the same as documented, and this is the actual second half of what I came here to do.

End state: I have a working command sequence for both install and (probably) update,
and — this is the headline result — **at no point on this path did the docs mention
Yocto, BitBake, KAS, building an image, or a kernel.** The pre-built-image route the
start page offered stayed pre-built the entire way through. I never got near my hard
stop. Patience dipped three times (LXC/trail, device-ip, BSP) but never ran out; I'd
put this at roughly 15 minutes of my 30-minute budget, comfortably inside it.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | A, "Clone the Device" step | I'm told to run `pvr clone http://<device-ip>:12368/cgi-bin mydevice` but nothing on this page says how to find my Pi's IP — I only knew to look for `pvr device scan` because I happened to skim the CLI reference page moments earlier for an unrelated reason (installing `pvr` itself). | "`pvr clone http://<device-ip>:12368/cgi-bin mydevice`" — no mention of how to obtain `<device-ip>` anywhere on the page | S2 | `broken-path` | Link `<device-ip>` in the clone command to the `pvr device scan` (mDNS) section of the CLI reference. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install` | A, "Install your first application" landing page | The very first sentence on the page tells me every app is "an LXC container added to the device's revision trail" — I know Docker containers, not LXC, and "revision trail" is never explained here either. | "Every application on a Pantavisor device is an LXC container added to the device's revision trail." | S3 | `undefined-jargon` | Link "LXC container" and "revision trail" to short definitions or the glossary on first use. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | A, "Deploy to the Device" step | It says a full reboot happens "when... the BSP changed" — I don't know what a BSP is, and given my one hard rule is never touching OS/kernel-level stuff, an unexplained term like this right next to my deploy command makes me nervous I'm closer to that line than I think. | "a full reboot only happens when a `system` restart-policy container or the BSP changed" | S3 | `undefined-jargon` | Add a short parenthetical or link explaining BSP, or explicitly reassure the reader that ordinary app updates never touch it. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | A, second half of goal — "update it once afterward" | Install got a full three-step walkthrough (add → stage/commit → post); "Update Applications" is a single isolated command with no matching sequence, so I have to guess that the same commit-and-post steps still apply afterward. | "#### Update Applications\n\n```bash\n# Update existing application\npvr app update nginx-app\n```" — no follow-up commit/post steps shown, unlike Install | S3 | `no-example` | Add a short "Update an application" walkthrough mirroring the install steps (update → stage/commit → post). |

## Closing summary

- **Task outcome:** completed — container installed via `pvr app add`/`add`/`commit`/`post`, and I have a command I'm reasonably confident updates it later (`pvr app update`) without touching the device.
- **Worst finding:** the install walkthrough's clone step requires a device IP it never explains how to find — https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr — I only had it because I'd stumbled onto `pvr device scan` on a different page moments earlier for an unrelated reason.
- **What worked:** `getting-started/start/download-and-flash` followed by `develop/application/install/local-pvr` — pre-built image, `pvflasher` as a familiar one-liner, and an install sequence (`pvr app add` → `add`/`commit` → `post`) that reads exactly like Docker-build-plus-git-commit; at no point did this path mention Yocto, BitBake, KAS, or building a kernel, which is exactly the trap I was watching for.
- **Confidence:** high — stayed in character throughout. I flagged LXC, "revision trail," and BSP as undefined rather than silently resolving them with outside knowledge, and inferred the update command sequence explicitly as a guess rather than presenting it as documented fact.
