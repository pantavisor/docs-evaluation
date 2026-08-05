# Persona 11 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: manual (chat session, RUNBOOK.md followed)

## Task

Read `getting-started/how-to-install/` end to end, including the board pages, then
follow anything that touches identity or claiming. Produce a jargon-audit table of
every term used before it is defined.

## Path taken

`how-to-install/` (topics index) → SD Card → Docker/AppEngine → Toradex → UUU →
Board Guides index → all six board pages (Raspberry Pi, Colibri iMX6ULL, Verdin
iMX8MM, VAR-SOM-MX8M-NANO, DART-MX8M-MINI, i.MX8QXP MEK) → `operate/` (linked from
the Docker page's sidebar) → `operate/device-access/` → `operate/device-access/
remote-pantahub` → `operate/device-access/pvtx-ui` → `operate/device-access/
serial-port` (linked from `remote-pantahub` as the source of the two claim
credentials). I also opened `overview/get-started` (linked from the Docker page as
"build one yourself") and, from there, `overview/glossary` (linked as "see the
glossary if any of that's unfamiliar") purely to check whether any of the terms
below were defined somewhere I hadn't yet been — noted below where relevant.

## Jargon audit

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Line-stopping if I'm wrong? |
|---|---|---|---|---|
| `claim` / "claimed" / "Claim a Device" | [`.../how-to-install/docker`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker) ("...device claiming, and cloud logging.") | Partially — [`remote-pantahub`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/remote-pantahub) documents a manual, one-device-at-a-time web-UI procedure ("Log in to hub.pantacor.com, go to **Claim a Device**... and enter both values"). No mention anywhere I reached of a bulk or API-driven claim path. | That "provisioning" (my sense: writing the image) was the whole job. It isn't — claim is a separate, later, human-driven step on a cloud portal, disconnected from the flash line entirely. | **Yes — S1.** Every claim I found is one device, one login, one form, by hand. Nothing tells me whether hub.pantacor.com has a batch/API claim flow I could script into the station. At one unit per 40 seconds, a manual per-device web form is a line-stopper by itself, and I can't rule it out from here. |
| `device ID` (`/pv/device-id`) | [`remote-pantahub`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/remote-pantahub) ("Read the device ID and challenge token from the serial console debug shell") | No. [`serial-port`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/serial-port) only labels it `# unique device ID` in a comment; nothing says how or when it's generated, whether it's hardware-derived or random, or whether it survives a re-flash. | A per-unit identifier generated automatically, hopefully from something like a SoC/eMMC unique ID, at first boot. | **Yes — S1, automatically.** If two units can ever get the same `device-id`, or if it regenerates on re-flash (which would break a pre-claimed unit), that's a recall. Nothing here confirms either way. |
| `challenge` (token, `/pv/challenge`) | Same page, same sentence, as above | No. `serial-port` calls it a `# one-time claim token` and nothing more — no generation mechanism, no expiry, no statement of whether it's per-boot or fixed for the unit's life. | A one-time secret proving physical possession at claim time, presumably regenerated per boot attempt. | **Yes — S1.** "One-time" with no documented reset path is exactly the kind of thing that strands a unit if the claim window is missed on the line — I can't quote a source that tells me how to recover from that. |
| `provisioning` | [`.../how-to-install/docker`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker) ("a `pvtx.d` directory of `.pvrexport.tgz` bundles to **provision**"; later, "The `pvtx.d` **provisioning** scripts only run once per storage volume") | Not for this sense — `provisioning` has no glossary entry at all. | Going in, I read "provisioning" as *my* meaning: injecting per-unit identity at manufacture. The docs use it for something else entirely — running config/app-state scripts against an already-booted container. | **Yes — S1.** This is the exact mismatch the job briefing warned me about. If a spec sheet says "provisioning is handled" and means the `pvtx.d` sense, not mine, I'd quote a job assuming per-unit identity is covered when it isn't. |
| `Pantahub` | [`.../how-to-install/docker`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker) ("Remote tests validate **Pantahub** connectivity") | Yes — but only in [`overview/glossary`](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) ("The optional cloud backend..."), which I only found by following an unrelated Yocto-jargon link from `overview/get-started`. None of `docker`, `operate/`, `device-access/`, `remote-pantahub`, or `pvtx-ui` link to the glossary or explain the word locally. | Some cloud service the device talks to; not clear on first read whether it's optional or load-bearing. | No — the glossary does resolve it as "optional," so this one doesn't cost units by itself. Flagging as `broken-path`: the definition exists but nothing on the path I actually took gets me to it. |
| `pvtx` / `pvtx.d` | [`.../how-to-install/docker`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker) ("a `pvtx.d` directory of `.pvrexport.tgz` bundles") | Yes — glossary: "Pantavisor's on-device update-transaction tool." Same problem as `Pantahub`: not linked from `docker` or from [`pvtx-ui`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui), which describes the *app* built on it without ever expanding the name. | An on-device update mechanism; functionally guessable from `pvtx-ui`'s description even without the glossary. | No — S3. Didn't cost me anything, just friction. |
| `creds.prn` | [`pvtx-ui`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui) (Device Config table: "`creds.id`, `creds.prn`, `creds.secret`") | No — not in the glossary, not explained inline. | Something like a Pantahub resource name/ARN, scoped to the account rather than the unit — but I can't confirm that from anything I read. | Uncertain, marked S3 not S1: it showed up only as a field name inside a "treat this page as sensitive" warning, not inside an actual task step, so I can't say it blocked a step — but it's exactly the kind of unexplained identity-shaped noun the brief told me to flag. |
| `trail` | [`pvtx-ui`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui) ("the current revision number in the device's **trail**") | Yes — glossary: "The ordered history of a device's revisions." Not linked from `pvtx-ui` itself. | The device's update/revision log. | No — S3, `broken-path`. |
| `pvrexport` / `.pvrexport.tgz` | [`.../how-to-install/docker`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker) | Yes — glossary: "A tarball of one or more state parts... produced by `pvr export`." Not linked from `docker`. | A packaged bundle of app/config state to load onto a device. | No — S3, `broken-path`. |
| `WIC` (`.wic` image) | [`.../how-to-install/sdcard`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/sdcard) ("Most Pantavisor machines produce a `.wic` image") | Yes — glossary: "Yocto's disk-image partitioning tool." Not linked from any of `sdcard`, `toradex`, or `uuu`, which use `.wic` dozens of times between them. | A disk-image file format; didn't need to know more to follow the flashing commands mechanically. | No — S4, polish. Never blocked a step because every flashing page gives the literal filename pattern to use, not just the extension. |

## What worked

The flashing path itself — `how-to-install` topics index → SD Card / Docker / Toradex
/ UUU → Board Guides → every individual board page — was excellent for this
persona: exact partition tables, boot-mode switch positions, DIP-switch tables,
udev rules, and copy-pasteable `uuu`/`pvflasher`/`dd` commands, pitched at exactly
the level of someone who already knows `fastboot`, SDP, and gang programming. Zero
findings there. The gap is concentrated entirely in the identity/claiming domain,
once the path leaves flashing and touches Pantahub.

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding:** S1 — the only documented way to `claim` a device (`remote-pantahub`, https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/remote-pantahub) is a one-at-a-time, human-driven web form on hub.pantacor.com, with `/pv/device-id` and `/pv/challenge` never explained for generation, uniqueness, or persistence across re-flash anywhere reachable in the docs — for a 40-second-takt line, I cannot tell from the docs alone whether per-unit identity can be provisioned at scale or whether every unit needs a person at a keyboard after it leaves my station.
- **What worked:** [`how-to-install/toradex`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/toradex) and the board pages under it — factory-flash-grade detail (partition offsets, boot-mode strapping, udev rules) with nothing assumed.
- **Confidence:** stayed in character throughout — no prior Pantavisor/Pantahub knowledge used. I did follow one link (`overview/get-started` → `overview/glossary`) that doesn't itself touch identity or claiming, purely to double-check whether `Pantahub`/`pvtx`/`trail`/`pvrexport` were defined *anywhere* reachable before calling them undefined; that's a due-diligence check, not something this persona would organically click through mid-task, and it's noted per-row above rather than silently used to soften a finding. `claim`, `challenge`, `device ID`, and `provisioning` (in my sense) stayed undefined even after that check.
