# Persona 05 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narrative

Landed on `meta-pantavisor/getting-started/start`, per the prompt. The page's content
links only go to flashing, Docker, and `pvr` install — nothing about migrating off an
existing tool. But the very last line on the page beat me to my first A/B question before
I even asked it: "Pantavisor manages the entire device update itself. There is no
separate A/B image updater to install or configure underneath it." Mapping attempt #1:
"so where's the A/B partition pair?" — pre-empted, not confirmed or corrected by evidence
yet, just asserted.

Content links didn't lead toward migration, so I used the sidebar (a real reader's normal
next move once the article itself runs out) and found a "Migrate to Pantavisor" category
with named guides for Mender, RAUC, SWUpdate, Balena, and Docker Compose. I run Mender, so
I opened that one.

The Mender page is the strongest single page in this run. It states the model difference
up front — "Mender updates the image... Pantavisor replaces that model entirely... There
is no A/B duplication of the whole rootfs" — then gives a concept-mapping table that does
exactly what I need done for me:

- A/B rootfs slots → "Revisions in the trail... Rollback is to the previous revision, not
  a mirrored partition — no doubled storage for the full rootfs." Mapping attempt #2:
  "so rollback is a slot flip" — **corrected**: it's a pointer to a prior revision, not a
  partition swap, and there's no second mirrored rootfs sitting idle.
- Mender client (a service on the OS) → "Pantavisor **is** PID 1." Mapping attempt #3:
  "so there's an update agent watching the OS" — **corrected**: no agent layered on top,
  the runtime and updater are the same process.
- "Commit after successful boot (+ state scripts)" → "Health-gated commit + bootloader
  try/rollback," linked to an `atomicity-and-trust` page. This is exactly my "at which
  instant does the switch happen" question, so I followed it.

`atomicity-and-trust` gives real mechanism, not just a word: bootloader-enforced
try/rollback ("U-Boot `bootcount` + a trial/known-good revision pair, or `grub-editenv`
one-shot"), health-gated commit per container, a crash-consistent object store ("objects
are written and `fsync`'d before the manifest is written and atomically renamed; a
manifest is never referenced before its objects are durable"), and a hardware watchdog
fed by PID 1. Mapping attempt #4: "a trial/known-good revision pair — that's basically an
A/B slot pair for the boot pointer" — **confirmed, narrowly**: at the bootloader level
there is a two-state try/rollback structure, it just points at revision metadata instead
of two full mirrored rootfs partitions. This page also states plainly that the actual
power-cut test results aren't published yet — only the guarantee and the planned
methodology are documented so far. I make my own living testing exactly this with a relay
and a script, so a promise of future evidence isn't evidence.

I followed the "Revisions in the trail" and BSP/containers links next, since I've never
mapped a container onto a rootfs/kernel/bootloader relationship before and the persona
mapping I'd wrongly bring is "a container is a rootfs slot." The `containers` and `bsp`
pages corrected that directly: BSP (kernel, modules, firmware, the Pantavisor binary
itself) is versioned in the same revision as application containers, and the bootloader
integration is a separate, small, board-specific layer (`uboot`, `uboot-ab`, `rpiab`,
`grub`) that only reads a pointer Pantavisor writes — it's not part of a revision at all.
`uboot-ab` explicitly uses two mtd partitions selected by Pantavisor, which is the closest
thing to real A/B in this stack, and it's confined to the kernel/initrd fast-path, not the
whole device.

The `containers` page also happened to define `status_goal` (`MOUNTED` / `STARTED` /
`READY`) under a "Status Goal" heading — a term both the Mender migration page and the
`vs-mender` benchmark page use without linking anywhere. I only found the definition
because I was already drilling into containers for an unrelated reason.

Cross-checked against `benchmarks/vs-mender`, which restates the same mapping as a
side-by-side table — no new information, but no contradictions either, and it's a clean
page to hand to a skeptical CTO.

For the fleet-scale half of my job — the actual reason I'd trust or not trust this for
40,000 devices — I followed the Mender page's "claim the device on Pantahub for remote
OTA" link to `remote-pantahub`. It documents claiming one device, authenticating `pvr`,
and pushing a revision with `pvr post <device-url>` — one device at a time — plus
attaching free-form metadata tags (`location=warehouse tier=production`) for
organization. Nothing on that page shows a revision being targeted at a tagged group
instead of one URL. I went up to the parent `operate` category page hoping for a fleet
rollout guide and got a direct, honest answer instead of a workaround: "Guides for canary
rollouts, drift detection, air-gapped workflows, and watchdog integration are coming
soon." Mapping attempt #5: "so where's the canary cohort, the staged rollout percentage,
the campaign object" — **not ignored, explicitly deferred**. The docs say outright this
isn't built yet, which I respect more than silence, but it doesn't change that it isn't
here.

Checked the linked FAQ for "What happens if an OTA update fails?" — consistent with
everything above: automatic rollback to the previous revision, kept in
`/storage/trails`, no manual step. Same page's "Official pvr Reference" link points off
`docs.pantavisor.io` entirely, to `docs.pantahub.com` — a fence I'm not allowed to follow,
noted and not taken.

Finally read `when-not-to-use` for candor. It's the page most likely to earn trust from a
paranoid reader: it states outright there's no supported hybrid where an old updater
shares PID 1 with Pantavisor, and separately confirms X.509 signing, dm-verity, dm-crypt,
and secure-boot support exist — items on my "know cold" list that I'd otherwise have gone
hunting for.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate` | A — looking for how to target a cohort/subset of my 40,000 devices for a staged rollout | I need to know how to roll a revision out to 5% of the fleet before the other 95%, and the only deployment primitive I've found anywhere (`pvr post <device-url>`) targets exactly one device; the docs tell me straight out this doesn't exist yet. | "Guides for canary rollouts, drift detection, air-gapped workflows, and watchdog integration are coming soon." | S1 | `missing-concept` | Document a cohort/group-targeted deployment path (even a manual one using the existing device-metadata tags) before claiming fleet readiness. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security/atomicity-and-trust` | A — verifying the "atomic update" claim before trusting it at fleet scale | I got real mechanism (bootcount try/rollback, fsync+atomic-rename object store, watchdog) instead of just the word "atomic," but the page admits the actual power-cut test results and rig aren't published — only the guarantee and planned methodology are. I have no independently verifiable evidence, only a description of the plan to produce some. | "This page ships with measured results and the rig design as part of hardening the trust story. Until then it documents the guarantee and methodology." | S1 | `no-example` | Publish the actual power-cut test logs/results, or don't claim this page is where the evidence lives until it is. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/migrate/mender` | A — translating "gates per container on its `status_goal`" from the Mender concept-mapping table | The table asserts commits are gated on a `status_goal` per container but doesn't say what values it takes or link anywhere; I only found the definition (`MOUNTED`/`STARTED`/`READY`) two hops later, on the Containers overview page, while chasing an unrelated question about what a container even is. | "Pantavisor gates per container on its `status_goal`, with no scripts to write." — no link attached. | S3 | `broken-path` | Link `status_goal` on this table (and on the `vs-mender` benchmark table, which repeats the same unlinked term) straight to its definition on the Containers page. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting/faq` | A — looking for the full `pvr` command reference | The page's "Official pvr Reference" link sends me off `docs.pantavisor.io` to `docs.pantahub.com`, a domain outside my allowed scope, when the site has its own CLI reference page already linked earlier in this same run. | "[Official pvr Reference](https://docs.pantahub.com/pvr/)" | S4 | `outside-docs` | Point "Official pvr Reference" at the local `pvr-cli` reference page instead of the external `docs.pantahub.com` domain. |

## Closing summary

- **Task outcome:** completed — I never had to leave `docs.pantavisor.io` or guess at a
  mechanism; I reached a definite verdict using only what the docs state (including two
  places where they honestly state something isn't done yet).
- **Worst finding:** the Operate-devices page admits outright that canary rollouts and
  staged-deployment guides are "coming soon," and the only deployment command I found
  anywhere (`pvr post <device-url>`) targets one device at a time — for a 40,000-device
  fleet, that's not a hard-to-find answer, it's an answer that doesn't exist on this
  version of the site yet (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate).
- **What worked:** the Mender-to-Pantavisor migration page's concept-mapping table
  (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/migrate/mender)
  did the exact job I needed — it named my A/B assumptions and corrected or confirmed
  each one in a single table, then handed me straight to the mechanism page
  (`atomicity-and-trust`) that shows how the atomic switch actually works instead of just
  asserting it.
- **Confidence:** high. I never assumed content-addressed storage or LXC-as-firmware
  behavior beyond what a given page stated — I read the BSP/containers pages specifically
  because that relationship was outside my stated knowledge, and I logged the two
  genuinely missing pieces (fleet cohorts, published test evidence) as gaps rather than
  filling them in from outside knowledge I don't have as this persona.

**Verdict (in character):** Need more information. The revision/health-gating/bootloader
mechanism is real, specific, and better-documented per-component than what I run today —
I'd trust the *update mechanism itself* on a single device. I would not yet stand in
front of my CTO and the person who got paged with me and commit 40,000 devices to it,
because the docs don't show me how to stage a rollout to a subset of the fleet (only
"coming soon"), and the power-fail safety claim — the exact thing I'd insist on proving
myself with a relay and a script — is documented as a plan, not as evidence. I'd pilot on
a small, hand-managed batch using the existing device-metadata tags as an ad hoc cohort,
and wait for both gaps above to close before a real staged rollout.
