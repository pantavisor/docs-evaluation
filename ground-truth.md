# Ground truth — a control set for the prompts

A prompt pack that finds nothing looks exactly like documentation with no gaps. This
file tells the two apart.

Below are gaps confirmed to exist as of 2026-07-16, each mapped to the persona whose
knowledge boundary should make it unmissable. **If a persona run doesn't surface the
gaps in its own lane, the prompt is too weak — fix the prompt, not the docs.** That
inversion is the point of this file.

Do not paste this file into a persona run. It is the answer key.

**Provenance note.** Rows were originally verified on 2026-07-16 against local checkouts
of `pantavisor/docs/`, `meta-pantavisor/docs/`, and `pvr/docs/`. Personas now run against
the live `https://docs.pantavisor.io/development/`, which structurally differs from that
assumption in one important way: **there is no standalone `/pvr/` section on the live
site.** PVR CLI content lives folded into one page,
`meta-pantavisor/getting-started/develop/cli-tools/pvr-cli`, not the rich
`pvr/docs/commands/{app,wifi,sig,lowlevel}.md` tree several gaps below cite.

**Live re-verification (2026-07-23).** Every gap below has now been individually
re-checked against `/development/`: all 130 pages under
`/development/pantavisor/` and `/development/meta-pantavisor/` were fetched and their
`<article>` body text isolated from auto-generated sidebar navigation (so "does anything
*link* here" means a real inbound content link, not every page's nav listing every
sibling). Each row's **Live status** cell gives the result. Headline: **9 of 16
reproduce unchanged, 2 are cleanly resolved (pages fixed or removed), and 5 have
materially changed** — mostly because the live `/development/` docs are visibly newer
and more developed than the local trees were on 2026-07-16 (new pages like
`getting-started/operate/device-access/remote-pantahub` and
`develop/cli-tools/configuration` didn't exist in the local snapshot). A "changed" status
does not always mean "fixed" — in a couple of cases the underlying confusion moved
rather than closed. Don't take a persona's failure to reproduce a "changed" gap as a
broken prompt without reading the note first.

## Confirmed gaps

Originally verified against the local trees on 2026-07-16; **Live status** reflects
re-verification against `https://docs.pantavisor.io/development/` on 2026-07-23.

| # | Gap | Evidence (2026-07-16, local trees) | Lane | Live status (2026-07-23) |
|---|---|---|---|---|
| 1 | **The glossary is unreachable.** `meta-pantavisor/docs/overview/glossary.md` defines ~20 core terms — AppEngine, BSP, Groups, Object, Pantahub, pvtx, Revision, Trail, xconnect — i.e. precisely the vocabulary the other two trees leak. Nothing anywhere links to it. | `grep -rlin "glossary" pantavisor/docs meta-pantavisor/docs pvr/docs` returns **only `glossary.md` itself**. Zero inbound references, case-insensitive. | 1, 2, 12 | **Unchanged.** Fetched all 130 `/development/pantavisor/` + `/development/meta-pantavisor/` pages; "glossary" appears in body content on exactly one page — `meta-pantavisor/overview/glossary` itself. Still zero inbound links. |
| 2 | **`pvr claim` is undocumented, and there's a signposted dead end into it.** The command is registered (`pvr/cmd/claim.go:29`, wired at `pvr/cmd/pvrcli/pvrcli.go:238`) and absent from `pvr/docs/`. Worse: `pvr/docs/commands/wifi.md:129` tells the reader `claim-info` outputs "the device ID and claim challenge needed to claim the device on Pantacor Hub" — then no documented command accepts that challenge. | `grep -rn "claim" pvr/docs/` hits only `wifi.md`'s `claim-info` and the index row pointing at it. | 10, 11 | **Largely resolved.** A new page, `meta-pantavisor/getting-started/operate/device-access/remote-pantahub`, walks the full claim flow: "Log in to hub.pantacor.com, go to Claim a Device on the Devices page, and enter both values" — closing the old dead end. It's a web-UI flow, not a `pvr claim` CLI command (none exists), and `pvr-cli` still says "the device must be claimed" without saying how — but the trail no longer goes cold. |
| 3 | **`--runlevel` is undocumented on `app add` / `app install`.** A live deprecated alias for `--group` (`pvr/cmd/app/appadd.go:34`). Also undocumented: setting both `--group` and `--runlevel` is a hard error (`appadd.go:144`). Anyone hitting an older tutorial or CI job has nothing to search for. | `grep -rn "runlevel" pvr/docs/` → **0 hits**. | 12 (named explicitly); 10 (generically, via prompt B task 4) | **Unchanged.** `runlevel` appears once in the whole corpus, as prose ("startup ordering comes from container groups/runlevels" in `getting-started/solutions/iot-gateway`) — never as the `--runlevel` flag. Still 0 hits for the flag itself. |
| 4 | **Cross-repo enums are unresolvable.** `pvr/docs/commands/app.md` gives `--group {data,root,platform,app}`, `--restart-policy {system,container}`, `--status-goal {MOUNTED,STARTED,READY}` as bare enum lists. All three are *defined* in `pantavisor/docs/overview/containers.md`. Nothing links them. This is the sharpest testable gap in the corpus. | Compare `app.md` flag tables against `containers.md` §Groups / §Restart Policy / §Status Goal. | 10, 12 | **Shape changed, largely resolved.** A new page, `meta-pantavisor/getting-started/develop/cli-tools/configuration`, defines `status_goal`/`restart_policy` directly and links straight to `pantavisor/overview/containers`, which explains Groups/Status Goal/Restart Policy in full. But `--group` no longer exists as a CLI flag anywhere — `pvr app add` on live docs only documents `--from`/`--platform`; `status_goal`/`restart_policy` are shown only as `run.json` fields. The specific old complaint (bare CLI enum, unlinked) no longer applies; a narrower one might: the add-container *command's* docs don't show how to set these from the CLI at all. |
| 5 | **How a Docker/OCI image becomes an LXC container is never explained.** This is the core mechanic of `pvr app add --from`, and it is absent from all three trees. `meta-pantavisor/docs/getting-started/migrate/docker-compose.md:24` states that it happens — "Pulls the Docker/OCI image and converts it to an LXC container" — and stops there. | Absent everywhere; no page explains the conversion. | 1, 3, 4 | **Largely resolved.** `getting-started/develop/application/install/local-pvr` now shows the concrete result: "`pvr app add` pulls the image, converts it to a SquashFS rootfs, and creates the container's directory with" a listed `root.squashfs`, `lxc.container.conf`, `run.json`, etc. Still no explanation of the layer→squashfs mechanics itself, and the migrate/docker-compose page keeps the old one-line stop-short version, but a reader who keeps reading past that page now gets real detail. |
| 6 | **`getting-started/` has no landing page.** The `_category_.json` has no `link:` key, so the most important section in the corpus renders as an auto-generated card list: no narrative, no "which path am I on", no "what is this" before the reader is told to flash an SD card. | `cat meta-pantavisor/docs/getting-started/_category_.json` — keys are `label`, `collapsed`, `position`. No `link`. | 2, 4, 8 | **Resolved — moved to [Resolved](#resolved).** |
| 7 | **Yocto, BitBake and KAS are never defined** (35 and 37 files use them respectively), and none are in the glossary. Defensible inside `overview/`; but the assumption leaks into ~21 `getting-started/` pages. | `overview/index.md:9` opens the section with "Yocto/OpenEmbedded layer", "BitBake classes", "KAS configurations", "initramfs", "pvrexport" — five undefined terms in one paragraph. | 1, 2, 4 | **Unchanged.** `meta-pantavisor/overview` (live) opens: "meta-pantavisor is the Yocto/OpenEmbedded layer... It provides recipes, BitBake classes, and KAS configurations for producing initramfs images and container pvrexport bundles" — same five undefined terms, same paragraph. Glossary still has no entries for any of them. |
| 8 | **No Pantahub concept page.** Referenced across 39 files. The claim/challenge model, device identity, and whether you can self-host are never explained. "Do I need this, and what am I locked into" has no answer. | `glossary.md` gives it three lines; `operate/device-access/remote-pantahub.md` is the only how-to. | 8, 11 | **Unchanged** (reference count now 47, up from 39 — the surface area grew, the concept page still didn't arrive). Glossary entry is still ~3 lines; `remote-pantahub` is still the only substantial page, and it's a how-to (claim steps), not a "what is this, what does it cost, can I self-host" concept page. |
| 9 | **The best conceptual page is orphaned.** `overview/composable-firmware.md` explains the two-phase Build(Yocto)/Maintain(pvr) model — the mental model the whole product rests on — and is omitted from its own section index's topic list. | `overview/index.md` enumerates 13 topics + CI; omits `composable-firmware.md`, `glossary.md`, `examples/`, `port/`, `testing/`. | 1, 2 | **Unchanged.** Live `meta-pantavisor/overview` index lists 12 Topics/Build Guide/CI entries; still omits `composable-firmware`, `glossary`, `examples/`, `port/`, `testing/` by name. |
| 10 | **`pvr/docs` links to nothing.** The entire tree has exactly one outbound link, and it's to a third-party Go templating library. It never links to `pantavisor/docs` (which defines its enums), never to the glossary, never to `PVR_TEMPLATES.md` (the only docs for the `--arg` system `app.md` references six times). | `grep -rhoE '\]\((\.\./\|http)[^)]*' pvr/docs \| sort -u` → one hit, `github.com/Masterminds/sprig`. | 12 | **Superseded — the tree this described no longer exists.** Its live successor, the single `cli-tools/pvr-cli` page, links inward fine (to `containers`, its own `cli-tools/` siblings). But its "official documentation" link points to `https://docs.pantahub.com/pvr/` — **a different domain from `docs.pantavisor.io` entirely** — and there's a `gitlab.com/pantacor/pvr/-/packages` link too. Both are off-site for a docs-only-scoped reader. Noted here as a fresh observation, **not independently re-verified** (is `docs.pantahub.com` a legitimate separate property, and does it duplicate or replace this content? unknown) — worth a maintainer look before promoting to its own gap number. |
| 11 | **The board index omits the board the quickstart is built on.** `how-to-install/boards/index.md` lists 4 boards; 6 exist. Unlinked: `raspberry-pi.md` — the board the entire getting-started path uses — and `imx8qxp-b0-mek.md`. | `ls` the boards dir (7 files inc. index) vs the 4 entries under "Supported Boards". | 4, 9 | **Unchanged, arguably sharper.** Live board index still lists exactly the same 4 (Colibri iMX6ULL, Verdin iMX8MM, iMX8MN VAR-SOM, iMX8MM VAR-DART); `raspberry-pi` and `imx8qxp-b0-mek` are still unlinked. Raspberry Pi is now even more clearly the flagship board — `getting-started/start` names it as *the* quickstart path — which makes its absence from the board index harder to justify, not easier. |
| 12 | **`pantavisor/docs/issues.md` is stale.** Marked `draft: true`, states "There are currently no open issues" while listing only resolved ones. A meta-page shipped in a docs tree. | `pantavisor/docs/issues.md` | 12 | **Resolved — moved to [Resolved](#resolved).** |

## Found by the pack itself

Gaps 13–16 were surfaced by the first dry-runs (2026-07-16) and verified afterwards.
They're the evidence that the pack finds things its author didn't already know — which
is the only reason to run it. **Live status** reflects re-verification against
`https://docs.pantavisor.io/development/` on 2026-07-23.

| # | Gap | Evidence (2026-07-16, local trees) | Lane | Live status (2026-07-23) |
|---|---|---|---|---|
| 13 | **`pvr deploy` is described contradictorily across repos.** `meta-pantavisor/.../pvr-cli.md:191` says it "builds the directory on disk; it does **not** push to a device — use `pvr post` for that." `pvr/docs/commands/lowlevel.md:88` opens "Deploy a checked-out repository **to a device**." The rest of the `lowlevel.md` sentence agrees with `pvr-cli.md`, so the summary line is the defect — but a reader hits one or the other, not both. Compounded by every page in `develop/` using "deploy" as prose for `pvr post`. | Both lines verified 2026-07-16. | 3, 10 | **Largely resolved.** The `lowlevel.md` half of the contradiction has no live equivalent (no standalone `pvr/docs` tree — see gap #10), so the cross-repo contradiction can't reproduce. The surviving `pvr-cli` page is internally precise: "`pvr deploy`... builds the directory on disk; it does not push to a device — use `pvr post` for that." Residual, milder issue: the same page's "Best Practices" section still uses "deploy"/"deployment" loosely ("Test applications locally before deploying to production devices") right below the precise definition. |
| 14 | **A cross-repo link promises a definition the target doesn't contain.** `meta-pantavisor/docs/overview/boot-flow.md:33` — "see [Pantavisor](../../pantavisor/overview/revisions.md) for the trail/revision model". `revisions.md` contains the word "trail" **zero times**. | `grep -c "trail" pantavisor/docs/overview/revisions.md` → `0`. | 1, 2 | **Unchanged, word for word.** Live `meta-pantavisor/overview/boot-flow` still reads "...containers (see Pantavisor for the trail/revision model)" linking to `pantavisor/overview/revisions`. That page still contains "trail" **zero times**. |
| 15 | **Two pages state different defaults for `PANTAVISOR_FEATURES`.** `overview/meta-pantavisor.md:76` omits `container-mdev`; `overview/build-system.md:110` and `:119` include it. The two pages also duplicate the directory tree and recipe tables verbatim, so they'll keep drifting. | Both lines verified 2026-07-16. | 1, 2, 6 | **Unchanged, same specific defect.** Live `meta-pantavisor/overview/meta-pantavisor` lists the `??=` default as `"dm-crypt dm-verity autogrow runc tailscale debug rngdaemon pvcontrol xconnect"` (no `container-mdev`); live `meta-pantavisor/overview/build-system` lists the same default *with* `container-mdev` appended. Still drifting. |
| 16 | **The serial adapter is "optional" on one page and "crucial" on the next**, and is de-facto required by every verification and rollback path. `start/index.md:26` "Optional but recommended: a USB-to-TTL serial adapter". `start/download-and-flash.md:18` "**USB to TTY serial converter**: This is a crucial tool". Neither says what one is, which to buy, or that TTL and TTY here name the same object. | Both lines verified 2026-07-16. Console access is assumed by `develop/application/view.md:11`, `access-applications.md:11`, and rollback at `remove.md:57`. | 3, 4, 9 | **Unchanged, word for word.** Live `meta-pantavisor/getting-started/start`: "Optional but recommended: a USB-to-TTL serial adapter for console access." Live `.../start/download-and-flash`: "USB to TTY serial converter: This is a crucial tool for debugging your device." Still no cross-reference explaining TTL and TTY name the same thing here. |

## Lane coverage

Each core persona should catch at least one seeded gap. Personas with no seeded gap in
their lane (**5** and **7**) are exploratory — they're in the set because we have *no*
evidence about those readers yet, which is itself worth knowing. Don't treat a thin
report from them as prompt failure; treat it as a first data point.

| Persona | Seeded gaps in lane |
|---|---|
| 1 — Yocto, no containers | 1, 5, 7, 9, 14, 15 |
| 2 — Buildroot, no Yocto, no containers | 1, 6, 7, 9, 14, 15 |
| 3 — Cloud-native, no embedded | 5, 13, 16 |
| 4 — App dev on device | 5, 6, 7, 11, 16 |
| 5 — OTA migrator | *(none — exploratory)* |
| 6 — BSP bring-up | 15 |
| 7 — Security reviewer | *(none — exploratory)* |
| 8 — Adoption evaluator | 6, 8 |
| 9 — Field support | 11, 16 |
| 10 — Git-fluent pvr newcomer | 2, 4, 13 |
| 11 — Manufacturing / provisioning | 2, 8 |
| 12 — AI agent consumer | 1, 3, 4, 10, 12 |

**Read this against the 2026-07-23 live-status column above before treating a run as a
failure.** Persona 10's entire seeded lane (2, 4, 13) is now tagged "largely resolved" —
a clean persona 10 report on the live site is expected, not evidence the prompt broke.
Persona 8's lane includes gap 6, now fully resolved; a clean report there is likewise
expected. Persona 12's lane includes gap 10, now superseded rather than reproducible in
its original form.

## Maintaining this file

**When a run surfaces a new gap, verify it and add it here.** Ground truth that only
ever shrinks is ground truth nobody trusts — the set should grow as the personas teach
us about readers we hadn't modelled.

**When a gap gets fixed, don't delete the row — move it to Resolved below** with the
date and the commit or PR. A prompt that used to catch gap #4 and now reports nothing is
either a fixed doc or a broken prompt, and only a resolved-list tells you which.

## Resolved

| # | Gap | Resolved | How |
|---|---|---|---|
| 6 | `getting-started/` had no landing page — rendered as an auto-generated card list with no narrative. | 2026-07-23 (confirmed fixed; underlying commit/date not tracked here) | Live `meta-pantavisor/getting-started/start` opens with real narrative: "The fastest way to understand Pantavisor is to run it — flash a real device, boot it, and ship your first update," followed by a "Paths" section. No longer auto-generated. |
| 12 | `pantavisor/docs/issues.md` was a stale meta-page ("no open issues" while listing only resolved ones). | 2026-07-23 (confirmed fixed; underlying commit/date not tracked here) | The page no longer exists on `/development/` at all (`https://docs.pantavisor.io/development/pantavisor/issues` → 404). Removed rather than fixed, which closes it just as well. |

Gaps #2, #4, #5, #10, and #13 above are **substantially changed but not moved here** —
each still has a residual note in its Live status cell. They're judgment calls for
whoever next confirms them hands-on rather than clean fixes; see each row.
