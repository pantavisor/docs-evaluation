# Ground truth — a control set for the prompts

A prompt pack that finds nothing looks exactly like documentation with no gaps. This
file tells the two apart.

Below are gaps confirmed to exist as of 2026-07-16, each mapped to the persona whose
knowledge boundary should make it unmissable. **If a persona run doesn't surface the
gaps in its own lane, the prompt is too weak — fix the prompt, not the docs.** That
inversion is the point of this file.

Do not paste this file into a persona run. It is the answer key.

## Confirmed gaps

All verified directly against the trees on 2026-07-16. Evidence commands are given so
they can be re-run when the docs change.

| # | Gap | Evidence | Lane |
|---|---|---|---|
| 1 | **The glossary is unreachable.** `meta-pantavisor/docs/overview/glossary.md` defines ~20 core terms — AppEngine, BSP, Groups, Object, Pantahub, pvtx, Revision, Trail, xconnect — i.e. precisely the vocabulary the other two trees leak. Nothing anywhere links to it. | `grep -rlin "glossary" pantavisor/docs meta-pantavisor/docs pvr/docs` returns **only `glossary.md` itself**. Zero inbound references, case-insensitive. | 1, 2, 12 |
| 2 | **`pvr claim` is undocumented, and there's a signposted dead end into it.** The command is registered (`pvr/cmd/claim.go:29`, wired at `pvr/cmd/pvrcli/pvrcli.go:238`) and absent from `pvr/docs/`. Worse: `pvr/docs/commands/wifi.md:129` tells the reader `claim-info` outputs "the device ID and claim challenge needed to claim the device on Pantacor Hub" — then no documented command accepts that challenge. | `grep -rn "claim" pvr/docs/` hits only `wifi.md`'s `claim-info` and the index row pointing at it. | 10, 11 |
| 3 | **`--runlevel` is undocumented on `app add` / `app install`.** A live deprecated alias for `--group` (`pvr/cmd/app/appadd.go:34`). Also undocumented: setting both `--group` and `--runlevel` is a hard error (`appadd.go:144`). Anyone hitting an older tutorial or CI job has nothing to search for. | `grep -rn "runlevel" pvr/docs/` → **0 hits**. | 12 (named explicitly); 10 (generically, via prompt B task 4) |
| 4 | **Cross-repo enums are unresolvable.** `pvr/docs/commands/app.md` gives `--group {data,root,platform,app}`, `--restart-policy {system,container}`, `--status-goal {MOUNTED,STARTED,READY}` as bare enum lists. All three are *defined* in `pantavisor/docs/overview/containers.md`. Nothing links them. This is the sharpest testable gap in the corpus. | Compare `app.md` flag tables against `containers.md` §Groups / §Restart Policy / §Status Goal. | 10, 12 |
| 5 | **How a Docker/OCI image becomes an LXC container is never explained.** This is the core mechanic of `pvr app add --from`, and it is absent from all three trees. `meta-pantavisor/docs/getting-started/migrate/docker-compose.md:24` states that it happens — "Pulls the Docker/OCI image and converts it to an LXC container" — and stops there. | Absent everywhere; no page explains the conversion. | 1, 3, 4 |
| 6 | **`getting-started/` has no landing page.** The `_category_.json` has no `link:` key, so the most important section in the corpus renders as an auto-generated card list: no narrative, no "which path am I on", no "what is this" before the reader is told to flash an SD card. | `cat meta-pantavisor/docs/getting-started/_category_.json` — keys are `label`, `collapsed`, `position`. No `link`. | 2, 4, 8 |
| 7 | **Yocto, BitBake and KAS are never defined** (35 and 37 files use them respectively), and none are in the glossary. Defensible inside `overview/`; but the assumption leaks into ~21 `getting-started/` pages. | `overview/index.md:9` opens the section with "Yocto/OpenEmbedded layer", "BitBake classes", "KAS configurations", "initramfs", "pvrexport" — five undefined terms in one paragraph. | 1, 2, 4 |
| 8 | **No Pantahub concept page.** Referenced across 39 files. The claim/challenge model, device identity, and whether you can self-host are never explained. "Do I need this, and what am I locked into" has no answer. | `glossary.md` gives it three lines; `operate/device-access/remote-pantahub.md` is the only how-to. | 8, 11 |
| 9 | **The best conceptual page is orphaned.** `overview/composable-firmware.md` explains the two-phase Build(Yocto)/Maintain(pvr) model — the mental model the whole product rests on — and is omitted from its own section index's topic list. | `overview/index.md` enumerates 13 topics + CI; omits `composable-firmware.md`, `glossary.md`, `examples/`, `port/`, `testing/`. | 1, 2 |
| 10 | **`pvr/docs` links to nothing.** The entire tree has exactly one outbound link, and it's to a third-party Go templating library. It never links to `pantavisor/docs` (which defines its enums), never to the glossary, never to `PVR_TEMPLATES.md` (the only docs for the `--arg` system `app.md` references six times). | `grep -rhoE '\]\((\.\./\|http)[^)]*' pvr/docs \| sort -u` → one hit, `github.com/Masterminds/sprig`. | 12 |
| 11 | **The board index omits the board the quickstart is built on.** `how-to-install/boards/index.md` lists 4 boards; 6 exist. Unlinked: `raspberry-pi.md` — the board the entire getting-started path uses — and `imx8qxp-b0-mek.md`. | `ls` the boards dir (7 files inc. index) vs the 4 entries under "Supported Boards". | 4, 9 |
| 12 | **`pantavisor/docs/issues.md` is stale.** Marked `draft: true`, states "There are currently no open issues" while listing only resolved ones. A meta-page shipped in a docs tree. | `pantavisor/docs/issues.md` | 12 |

## Found by the pack itself

Gaps 13–16 were surfaced by the first dry-runs (2026-07-16) and verified afterwards.
They're the evidence that the pack finds things its author didn't already know — which
is the only reason to run it.

| # | Gap | Evidence | Lane |
|---|---|---|---|
| 13 | **`pvr deploy` is described contradictorily across repos.** `meta-pantavisor/.../pvr-cli.md:191` says it "builds the directory on disk; it does **not** push to a device — use `pvr post` for that." `pvr/docs/commands/lowlevel.md:88` opens "Deploy a checked-out repository **to a device**." The rest of the `lowlevel.md` sentence agrees with `pvr-cli.md`, so the summary line is the defect — but a reader hits one or the other, not both. Compounded by every page in `develop/` using "deploy" as prose for `pvr post`. | Both lines verified 2026-07-16. | 3, 10 |
| 14 | **A cross-repo link promises a definition the target doesn't contain.** `meta-pantavisor/docs/overview/boot-flow.md:33` — "see [Pantavisor](../../pantavisor/overview/revisions.md) for the trail/revision model". `revisions.md` contains the word "trail" **zero times**. | `grep -c "trail" pantavisor/docs/overview/revisions.md` → `0`. | 1, 2 |
| 15 | **Two pages state different defaults for `PANTAVISOR_FEATURES`.** `overview/meta-pantavisor.md:76` omits `container-mdev`; `overview/build-system.md:110` and `:119` include it. The two pages also duplicate the directory tree and recipe tables verbatim, so they'll keep drifting. | Both lines verified 2026-07-16. | 1, 2, 6 |
| 16 | **The serial adapter is "optional" on one page and "crucial" on the next**, and is de-facto required by every verification and rollback path. `start/index.md:26` "Optional but recommended: a USB-to-TTL serial adapter". `start/download-and-flash.md:18` "**USB to TTY serial converter**: This is a crucial tool". Neither says what one is, which to buy, or that TTL and TTY here name the same object. | Both lines verified 2026-07-16. Console access is assumed by `develop/application/view.md:11`, `access-applications.md:11`, and rollback at `remove.md:57`. | 3, 4, 9 |

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

## Maintaining this file

**When a run surfaces a new gap, verify it and add it here.** Ground truth that only
ever shrinks is ground truth nobody trusts — the set should grow as the personas teach
us about readers we hadn't modelled.

**When a gap gets fixed, don't delete the row — move it to Resolved below** with the
date and the commit or PR. A prompt that used to catch gap #4 and now reports nothing is
either a fixed doc or a broken prompt, and only a resolved-list tells you which.

## Resolved

*(nothing yet)*
