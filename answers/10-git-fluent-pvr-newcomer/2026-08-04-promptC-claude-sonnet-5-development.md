# Persona 10 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Scope read

`https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli`,
`.../cli-tools/pvcontrol`, `.../cli-tools/configuration`, `.../cli-tools/workflows` — all four
fetched via their `.md` export.

## Table 1 — Undefined terms

Judged strictly against what I actually know (git's object model, Docker, JSON, REST, the
shell) and explicitly don't (Pantavisor, devices, embedded Linux, LXC, BSPs).

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant |
|---|---|---|---|
| `trail` | `.../cli-tools/pvcontrol` — "List the revisions (steps) in the device trail" | No — not on any of the four pages, and no link out from where it's used. | pvr's word for commit history — the ordered sequence of revisions for a device. |
| `xconnect` | `.../cli-tools/pvcontrol` — "Pantavisor internal daemons (e.g. `pv-xconnect`)" | No — reappears in `.../cli-tools/configuration` ("`services`: xconnect service requirements") but is never explained in either place. | Some kind of inter-container wiring/discovery mechanism, guessed from "service graph" and "service requirements." |
| `BSP` | `.../cli-tools/configuration` — repo layout comment, "the BSP artifacts (squashfs, kernel image, …)" | Not on this page or any of the other three. A definition does exist behind the "Configure applications" link that both `pvcontrol` and `configuration` point to, but that link's anchor text gives no hint it defines BSP, so I'd only find it by accident. | A pre-built firmware/kernel blob you flash, inferred from being grouped with "kernel, initrd, modules, firmware, DTB/FIT paths." |
| `LXC` (as container type) | `.../cli-tools/configuration` — `"type": "lxc"` in the `run.json` example, and "LXC runtime configuration" for `lxc.container.conf` | No — never defined or linked anywhere in the section. | Some container runtime, presumably related to but distinct from Docker (Docker is only mentioned for pulling images, not running them). |
| `Pantahub` | `.../cli-tools/pvr-cli` — "Pantacor Hub (requires `pvr login`) ... talk to the Pantahub cloud API" | No — named as though already known. | A hosted dashboard/API for managing a fleet remotely, inferred from the surrounding `device create/get/logs/ps` commands. |
| "the device must be **claimed**" | `.../cli-tools/pvr-cli` — same Pantahub paragraph | No — no link or explanation of what claiming is or how to do it. | Some registration step that associates a physical device with my Pantahub account, mechanism unknown. |
| `JOSE` | `.../cli-tools/pvr-cli` — "Show signatures with full JOSE serialization" | No — acronym never expanded or linked. | A signing/crypto envelope format (recognized the acronym from general REST background, but nothing here confirms it). |
| "**factory revision**" | `.../cli-tools/pvcontrol` — "`pvcontrol cmd make-factory` # mark the current revision as the factory revision" | No. | A designated fallback/recovery revision, used when normal rollback isn't enough — never confirmed. |

## Table 2 — The false-friend audit

"What I therefore assumed" filled in from the persona card's git model, before reading what
the docs say.

| Command | What git's version does | What I therefore assumed | What the docs say it does | Do they differ? | Would the difference have bitten me? |
|---|---|---|---|---|---|
| `init` | Creates a new, empty local repo (`.git`); no network. | `pvr init` creates an empty local repository on disk, no network call. | "`pvr init`" — "Create a new Pantavisor repository" (pvr-cli); "`pvr init` pvr-izes the current directory" (workflows). Consistent with local-only. | No. | No. |
| `add` | Stages working-tree changes into the index; local. | `pvr add .` stages changes into an index-like area before commit. | "Stage all changes" / "Use `pvr add .` to stage all changes before committing" (pvr-cli Tips). Consistent. | No. | No. |
| `status` | Diffs working tree/index/HEAD; read-only, local. | `pvr status` shows what changed, read-only, no network. | Never explained anywhere in the section — pvr-cli names it only to exclude it: "other day-to-day commands not covered here include `pvr status`, `pvr diff` ... run `pvr help` (or `pvr help <command>`)." | Unknown — can't confirm from this section. | Possibly — I'd have to leave the docs and run `pvr help status` to find out, with no confirmation the docs agree with what the CLI's own help says. |
| `diff` | Shows changes between working tree/index/commits; read-only, local. | Same idea — diff against my last commit or a device's state. | Same "not covered here" sentence as `status`, no link, no definition anywhere in the section. | Unknown. | Possibly, same as `status`. |
| `commit` | Records staged changes as a local commit object; cheap, no network. | `pvr commit` is local and cheap, doesn't touch the device. | Every workflow shows `pvr commit` as a separate local step before the explicit network step `pvr post <endpoint>` (e.g. workflows.md's entire clone→edit→commit→post loop). Tips: "Repository history is maintained like Git." | No — consistent. | No, but this is inferred from workflow ordering, never stated outright as "commit is local-only." |
| `clone` | Downloads the full repo — all history, all refs — into a new local repo tracking the remote. | `pvr clone <device>` pulls the device's entire revision history, so I can inspect/diff past revisions afterward. | configuration.md: "A Pantavisor device revision is described by a single JSON manifest... When you `pvr clone` a device, that manifest is expanded into a working directory." Describes cloning one manifest — the current state — not a history of manifests. | Likely yes — nothing confirms past revisions come down with a clone. | Yes — I'd expect to browse/diff earlier revisions locally the way `git log` lets me, and the docs never say whether I can. |
| `get` | No command literally named `get` in git; the metaphor I'd reach for is `git fetch` — download remote refs/objects without touching my working tree. | `pvr get` downloads the device's latest state without merging it into my working copy. | No top-level `pvr get` appears anywhere in the section. The only "get" is `pvr device get DEVICE_ID` — "Get device information" (pvr-cli) — metadata about a device, not a repo fetch. | Yes — the command I assumed exists isn't there. | Yes — I'd type `pvr get` expecting to sync state and get "unknown command." |
| `merge` | Combines two branch histories, creating a merge commit if needed; local. | `pvr merge` combines two revisions or checkouts the way branches converge in git. | `merge` never appears anywhere across all four pages. Multi-device workflows do clone→edit→commit→post per device with no branching/merging concept named. | Yes — command doesn't exist, and no substitute is named either. | Yes — if two edits diverged I'd look for `pvr merge` and find silence, not even a note that pvr has no such concept. |
| `reset` | Moves a branch pointer (and optionally index/working tree) back; looks destructive but recoverable via reflog. | `pvr reset` rolls my checkout back to an earlier revision, recoverable if I get it wrong. | `reset` never appears anywhere in the section. workflows.md's Troubleshooting > System Recovery instead says: "roll back: check out the last known-good revision (clone it from the device or use your saved checkout) and post it," and separately: "Pantavisor automatically rolls back failed updates that never reach their status goal — manual rollback is for revisions that came up but misbehave." | Yes — there's no `pvr reset`; rollback is either automatic (device-side) or a manual re-clone-and-post. | Yes — I'd go looking for `pvr reset` to undo a bad commit and find nothing under that name. |
| `checkout` | A command — switches the working tree to a different commit/branch, updates HEAD; touches my working copy, not the remote. | `pvr checkout <revision>` switches my local working copy to a different revision without touching the device. | "checkout" is only ever used as a **noun** for the directory `pvr clone` produced — configuration.md: "A `pvr` checkout mirrors the device revision"; workflows.md: "...clone it from the device or use your saved checkout." No `pvr checkout` verb/command appears anywhere in the section. | Yes, fundamentally — it isn't a command in this vocabulary at all, just a noun for "your local clone." | Yes — I would type `pvr checkout <rev>` expecting to switch revisions locally and find no such command; switching revisions apparently means re-cloning or reaching for on-device tools instead. |

## Outbound links from the section

Counting every markdown link across the four pages: **13 link instances, 8 unique
destinations.**

- 2 instances (both in pvr-cli.md, under "Official Documentation") go **off-domain** — the
  GitLab package registry and `docs.pantahub.com/pvr` — which I did not follow, per the scope
  fence.
- 2 instances (one in pvcontrol.md, one in configuration.md) point to
  `/pantavisor/reference/pantavisor-tools` — the **only** internal link in the section missing
  the `/development/` version prefix every other internal link carries. I tested it: it 404s.
- The remaining 9 instances land on 5 unique, working, in-fence destinations: the `pvr-cli`
  page itself (linked back from pvcontrol.md and workflows.md), "Configure applications"
  (linked 3×, from pvcontrol.md, configuration.md, and workflows.md), the state-format
  reference (2×, from configuration.md), "Install applications," and "Remove applications."

So: partially. Three of the ten borrowed-verb names I audited above (`status`, `diff`, and
everything to do with `BSP`/`LXC`/`trail`/`xconnect`) are never linked from anywhere in this
section — I'd have had to stumble onto "Configure applications" for unrelated reasons and
happen to notice it mentions `pvr status` in an example. `merge`, `reset`, and `checkout`
(as a command) have no destination to link to because the docs never introduce them as
concepts needing a link — a reader can't be pointed toward an explanation that doesn't
exist. And one of the two working-looking cross-section links is actually dead.

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding (S1):** `checkout` is never a command in pvr's vocabulary — only ever a
  noun for the directory `pvr clone` produces (configuration.md: "A `pvr` checkout mirrors
  the device revision") — and nothing in the four pages ever says the git metaphor exists,
  let alone that this is where it silently breaks; a git-fluent reader would type
  `pvr checkout <rev>` expecting to switch revisions locally and get nothing, with no warning
  anywhere in the section that this gap exists. `missing-concept`.
- **What worked:** `.../cli-tools/configuration`'s repository-layout tree and `run.json`
  field table — for the parts it does cover (`init`, `add`, `commit`, `status_goal` enum
  values, `auto_recovery` fields), it's precise and gave real field-by-field detail I could
  act on without guessing.
- **Confidence:** stayed in character throughout — I did not use outside knowledge of
  Pantavisor/pvr; every "what I assumed" cell in Table 2 came only from git's actual model,
  filled in before reading the corresponding docs text, and the two off-domain links noted
  above were logged, not followed.
