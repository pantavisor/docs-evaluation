# Persona 12 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Per-task: what I'd have told the developer vs. what's actually documented

**1 — "Add nginx to my device with the right settings."** The real add-container command
is `pvr app add` (`.../develop/cli-tools/pvr-cli` and
`.../develop/application/install/local-pvr`). Its only documented flags anywhere on the
site are `--from`, `--platform`, and the templating flag `--arg KEY=VALUE`. **Neither
`--group` nor `--status-goal` is a flag of `pvr app add`, or of any command, anywhere.**
They're fields of the container's `run.json` manifest file — set by hand-editing that
file after `pvr app add`, not by a CLI flag at add-time. If I'd been asked to "use the
right `--group` and `--status-goal`," I would have hallucinated a command like
`pvr app add --group app --status-goal STARTED nginx --from nginx:stable-alpine` — that
command would fail outright; `pvr app add` doesn't parse those flags. Once I know they're
`run.json` fields instead: `status_goal`'s permitted values (`MOUNTED`, `STARTED`,
`READY`) are given inline on the config page. `group`'s permitted *default* values
(`data`, `root`, `platform`, `app`) are real but I could not reach them from any
relevantly-labelled link inside `cli-tools/` — see finding below. Had I guessed anyway
for an nginx web app, I'd have picked `group: app`, `status_goal: STARTED` — which
happens to be exactly the `app` group's own defaults, so the guess would have been right,
but only by luck, not because the docs told me so at the point of decision.

**2 — "Claim my new device."** This one traces cleanly, no invention required:
`cat /pv/device-id` and `cat /pv/challenge` on the device's serial console, then
"Claim a Device" on hub.pantacor.com, entering both values
(`.../develop/application/install/remote-pantahub`, confirmed by the canonical
`.../operate/device-access/remote-pantahub`). The trail ends at a manual step on an
external dashboard — the docs are upfront that this can't be scripted from `pvr`, so
that's not a gap, just a real constraint I'd pass on to the user. The one loose end:
`pvr-cli`'s Pantacor Hub section separately documents `pvr device create mydevice1`
("Create a new device") right next to the claim-relevant `pvr device get`/`ps`/`logs`
commands, with nothing distinguishing it from the challenge-token claim flow. Under time
pressure I could see myself running `pvr device create` and telling the user "your device
is claimed" — it isn't; that command doesn't consume a challenge token at all as
documented.

**3 — "What does `--runlevel` do? It's in our CI script."** I checked every plausible
page: the `pvr` CLI reference, the `pvcontrol` reference, the `pantavisor.json`
configuration reference, the control-socket (`pantavisor-commands`) reference, Boot Flow,
Init Mode, Hooks, the glossary, and the Architecture overview. `runlevel` in any spelling
appears on none of them. What I'd tell the user: I cannot find `--runlevel` documented
for `pvr`, `pvcontrol`, `pvtx`, or the Pantavisor boot/kernel command line anywhere on
this site — I don't know what it does, and I would **not** advise removing it from a CI
script on that basis. Recommending removal would be exactly the confident-but-baseless
move this persona exists to catch; the honest answer is "unconfirmed, please check the
tool that actually consumes it."

**4 — "Explain how my Docker image ends up running on the device."** meta-pantavisor's
side is concrete: `pvr app add` pulls the image, flattens it to a single SquashFS rootfs,
and compiles the Docker config directly into `lxc.container.conf`
(`ENTRYPOINT`/`CMD`→init command, `ENV`→`lxc.environment`, `WORKDIR`→`lxc.init.cwd`,
`VOLUME`→mounts) — `.../develop/application/install/local-pvr`. If the docs had been
silent here, my general container knowledge would have led me to assume the device runs
an OCI-compliant runtime (containerd/runc) preserving the image's layers — that
assumption would have been **wrong**: there's no Docker daemon and no layered image on
the device, just a flattened rootfs run under LXC. Good thing the docs say so explicitly.
Checking `pantavisor/` per this prompt's own instruction: `.../pantavisor/overview/containers`
is where that LXC execution is actually specified — rootfs + LXC config run in a
namespace, group-gated startup order, status-goal state machine, restart policy. That's
real and complete. But nothing in the meta-pantavisor "add an app" trail (`local-pvr`,
`pvr-cli`, `cli-tools/configuration`) links to it directly — the only bridge is a generic
"Reference" link at the bottom of `.../getting-started/develop` pointing at the
`pantavisor/overview/` index, not at Containers specifically. I found it by opening that
index and picking item 4 myself.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` and `.../develop/cli-tools/configuration` | B1, step "read every flag `pvr app add` documents" | Nothing on either page states that `group`/`status_goal` are *not* `pvr app add` CLI flags — they're `run.json` fields I'd only discover by separately reading the configuration page. A reader (or agent) working purely off the command reference has every reason to assume flags named after these fields exist. | `pvr-cli`'s only documented flags for `pvr app add` are `--from` and `--platform`; `configuration` says `status_goal` and `group` are keys inside `<container>/run.json`, with no cross-note on the command page that these aren't add-time flags. | S1 | `missing-concept` | On the `pvr app add` section, add one line stating group/status-goal are set via `run.json` after add, not as CLI flags. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` and `.../pantavisor/reference/pantavisor-commands` (control-socket reference), `.../overview/init-mode`, `.../overview/boot-flow`, `.../overview/hooks`, `.../overview/glossary`, `.../overview/pantavisor-architecture` | B3, "search the docs for `--runlevel`" | I searched every CLI/reference/boot page I could find and `runlevel` (in any spelling) appears on none of them. | Absence confirmed across `pvr-cli`, `pvcontrol`, `pantavisor-configuration`, `pantavisor-commands`, `init-mode`, `boot-flow`, `hooks`, `glossary`, `pantavisor-architecture` — none mention "runlevel". | S1 | `missing-concept` | If `--runlevel` is a real, current flag of any Pantavisor tool, document it; if it's obsolete, say so explicitly somewhere discoverable so agents/readers stop guessing. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/configuration` | B1, "can you reach a definition of `group`'s permitted values by following links?" | This page names `group` but gives no enumeration of valid group names, only "defined in `device.json`." Its one onward link (`pantavisor-state-format-v2`) gives the `groups.json` *schema* but explicitly does not list default group names either. The actual default names (`data`/`root`/`platform`/`app`) live only on `pantavisor/overview/containers`, reached only by clicking an unrelated fragment link (`#loggers` or `#auto-recovery`) on the state-format page and then scrolling past that anchor to an unrelated section of the same page. | `configuration`: "`group` — Orchestration group (defined in `device.json`); inherits the group's defaults." No group names given. `pantavisor-state-format-v2`: "specific predefined names are not enumerated here." | S2 | `broken-path` | Link `group` on the configuration page directly to `pantavisor/overview/containers#groups`, which already has the default-groups table. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr`, `.../cli-tools/pvr-cli`, `.../cli-tools/configuration` | B4, "check `pantavisor/` for its side of that mechanism" | None of the three meta-pantavisor pages that explain adding/configuring a container link to `pantavisor/overview/containers`, the page that actually explains how the rootfs+LXC-config pair gets executed on-device. The only path across is a generically-labelled "Reference" link at the bottom of `.../getting-started/develop`, pointing at the section index rather than the specific page. | `.../getting-started/develop`: "For the state, revision, and BSP data model this CLI operates on, see the Reference." — links to `/development/pantavisor/overview/`, not to `overview/containers` specifically. | S2 | `broken-path` | Add a direct link from `local-pvr`'s "Pantavisor runtime manifest" mention to `pantavisor/overview/containers`. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | B2, "claim my new device" | `pvr device create mydevice1` is documented one line above `pvr device get`/`logs`/`ps` under "Pantacor Hub," with nothing distinguishing it from the challenge-token claim flow described on a different page — an agent moving fast could run this thinking it claims the device. | "`pvr device create mydevice1` — Create a new device" sits in the same code block as `pvr device get`, `logs`, `ps`, all under one "Pantacor Hub (requires `pvr login`)" heading, with no claim-flow cross-reference. | S3 | `undefined-jargon` | Add a one-line note by `pvr device create` clarifying it's not the challenge-token claim step, with a link to the claim walkthrough. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/configuration` | B1, researching `group`/`status_goal` | The page's link to "on-device tools (pvtx, pvcontrol)" 404s under both the version-prefixed and un-prefixed path — the actual `pvcontrol` reference exists elsewhere on the site, just not at this URL. | Link text "on-device tools (pvtx, pvcontrol)" → `/pantavisor/reference/pantavisor-tools`; both `https://docs.pantavisor.io/pantavisor/reference/pantavisor-tools` and `.../development/pantavisor/reference/pantavisor-tools` return 404. | S4 | `stale` | Fix the link to point at `.../develop/cli-tools/pvcontrol` (and/or `pantavisor/reference/pantavisor-commands`). |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview` and `.../getting-started/develop` and `.../getting-started/develop/cli-tools` | B1, general navigation | Several inline links lose their markdown syntax in the `.md` export specifically right after bold text or at a sentence's end ("go straight toDevelop applications", "CLI tools — `pvr`, `pvtx`..." with no link, a trailing "Reference" with no link) — the rendered HTML confirms all of these are real, working links, so this is an export-tooling bug, not a missing page. | `.md` export of `overview`: "...and go straight toDevelop applications ." (link syntax dropped); rendered HTML confirms `/development/meta-pantavisor/getting-started/develop/` is the real target. | S4 | `stale` | Fix the `.md`-export generator to preserve link syntax on bold-led list items and links immediately following inline formatting. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools` (footer nav) | B1, structural check (this persona counts links) | The persistent footer nav's "PVR CLI" link points at an unversioned `/pvr` page (a separate, out-of-fence legacy reference) instead of the version-scoped `.../cli-tools/pvr-cli` page it sits right next to in the same footer. | Footer: "PVR CLI: `/pvr`" alongside "Pantavisor Runtime: `/pantavisor/overview`" and "meta-pantavisor: `/meta-pantavisor/overview`" (also unversioned) — `/pvr` resolves to a separate, differently-structured command reference outside any `{VERSION}` path. | S4 | `stale` | Point the footer's "PVR CLI" link at the versioned `.../develop/cli-tools/pvr-cli` page, or retire the unversioned `/pvr` page. |

**Absence of a finding, noted:** Task 2 (claiming a device) is a clean pass — both
`install/remote-pantahub` and the canonical `operate/device-access/remote-pantahub` give
a complete, consistent, runnable trail from `cat /pv/challenge` to a claimed device, and
`status_goal`'s permitted values (`MOUNTED`/`STARTED`/`READY`) are stated inline exactly
where a reader would look for them on the configuration page — no navigation needed for
that half of the `group`/`status_goal` question.

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding:** S1 — neither `--group` nor `--status-goal` is a real flag of
  `pvr app add` (the actual add-container command), but nothing on the command reference
  says so, so an agent asked for "the right settings" would confidently emit a command
  that fails outright.
- **What worked:** `.../getting-started/develop/application/install/remote-pantahub` — a
  complete, numbered, self-contained claim-to-deploy walkthrough with real commands and no
  gaps, the strongest page encountered this run.
- **Confidence:** Mostly stayed in character. I fetched one page outside the `{VERSION}`
  scope fence by accident — `https://docs.pantavisor.io/pvr`, reached via a footer nav
  link before I registered it was unversioned — and used only its existence/target as
  evidence for the last finding above, not its content, for any task answer. I did not use
  prior product knowledge beyond one explicitly logged inference in Task 4 (assuming an
  OCI/layered-image runtime absent documentation), which I flagged as wrong once I found
  the actual documented mechanism.
