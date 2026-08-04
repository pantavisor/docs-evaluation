# Persona 05 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narrative

Prompt B hands me six fixed questions to answer before I'd put this in front of my CTO,
so I went hunting for a migration entry point rather than reading front-to-back. Guessing
`meta-pantavisor/getting-started.md` and a few sibling paths 404'd — the actual
"getting started" content lives nested under `overview/get-started`, not at the slug I'd
have guessed, and I don't get to infer from slugs, only from links I've actually followed.
`meta-pantavisor/overview.md` and `pantavisor/overview.md` (the two section landings) gave
me the lay of the land but neither links to anything about migrating off another OTA tool.
I had to fall back to the rendered page's sidebar (not just article content) to find it,
the way a reader runs out of in-article links and reaches for the nav: a "Migrate to
Pantavisor" category at `meta-pantavisor/getting-started/migrate/`, listing named guides
for Mender, RAUC, SWUpdate, Balena, and Docker Compose. I run Mender, so I opened that one.

`migrate/mender` is a strong page for my job specifically: it states the model difference
up front ("Mender updates **the image**... Pantavisor replaces that model entirely"), then
a concept-mapping table ending in "Commit after successful boot vs Health-gated commit +
bootloader try/rollback" — linked straight to `security/atomicity-and-trust`. That page is
exactly my question 1 and 2 territory, so I followed it.

`atomicity-and-trust` names real mechanism — U-Boot `bootcount` or `grub-editenv` one-shot
for the bootloader side, `fsync` + atomic rename for the object store, health-gated commit
per container, a hardware watchdog — but only asserts the power-fail guarantee as one
sentence, not a stage-by-stage walkthrough of what a Mender person would actually check.
Question 3 (rollback trigger) sent me to `pantavisor/overview/updates` for the state
machine (`NEW → SYNCING → QUEUED → DOWNLOADING → INPROGRESS → TESTING → UPDATED/DONE`,
with `WONTGO`/`ERROR`/`CANCELLED` failure states) and its ERROR-state causes table, which
does include a row for power/crash but as one generic line, not a per-stage breakdown.
`pantavisor/overview/containers` filled in the rest of question 3 cleanly: rollback is
device-local, gated on `status_goal` and `stable_timeout`, and `max_retries` exhaustion
during TESTING "always triggers a rollback regardless of the backoff policy" — a direct,
unambiguous answer to "who decides, the device or the server."

For question 1's exact mechanism I also checked `pantavisor/overview/disks` (its own
landing-page blurb promises "boot sequence") and `pantavisor/overview/pantavisor-architecture`
(state machine). Neither pins down the physical operation or a boot-attempt limit; `disks`
turned out to cover disk-mount ordering at runtime, not bootloader-level revision
selection. I checked the configuration reference for a bootcount-style limit and found
only `PV_REVISION_RETRIES` (default 10), which is documented as revision-transition
retries, not a boot-attempt count.

Question 4 sent me to `storage` (signing: state JSON signed, verified at boot and at
update receipt, against an on-disk RSA key or X.509 chain, with a configurable
`disabled`/`lenient`/`strict`/`audit` policy — lenient is the default and does let
unsigned artifacts through) which fully answers question 5 as a side effect. For sizes, I
went to `benchmarks/vs-mender` for the comparison table, then up to the `benchmarks`
index page when the table gave no numbers — same story there, explicitly deferred.

Question 6 took me back through `migrate/mender`'s "claim the device on Pantahub" link to
`operate/device-access/remote-pantahub`, which documents `pvr post <device-url>` — one
device, one URL — and metadata tags for organizing devices, but nothing that targets a
tagged group. Going up to the parent `operate` category page gave a direct answer instead
of silence: canary rollout guides are explicitly "coming soon."

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate` | B, Q6 — staged rollout | I need 1%→10%→100% with an abort for 40,000 devices, and the only deployment primitive I've found anywhere (`pvr post <device-url>`) targets one device; the parent category page tells me straight out that staged-rollout guides don't exist yet. | "Guides for canary rollouts, drift detection, air-gapped workflows, and watchdog integration are coming soon." | S1 | `missing-concept` | Document a cohort/group-targeted deployment path (even a manual one using the existing device-metadata tags) before claiming fleet readiness. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security/atomicity-and-trust` | B, Q2 — power-fail at each stage | I need to know what's on the device if I cut power mid-download, mid-write, after-write-before-reboot, mid-reboot, and after-boot-before-health-check, five distinct moments. The docs give me one blanket guarantee sentence, not a walkthrough, and the closest thing to per-cause detail — the ERROR-state table on the Updates page — has exactly one generic row for a power/crash cut, not five. | "At every point during an update, a power cut leaves the device able to boot *some* good revision." | S1 | `no-example` | Add a stage-by-stage table (the ERROR-causes table on the Updates page is the right model) showing on-disk state after a cut at each of the five points. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks` | B, Q4 — what's in an update, real numbers | I need real KB/MB figures for an app-only change versus a kernel/BSP change to size my rollout windows and flash-wear budget. The benchmarks index says outright the numbers aren't published, and the qualitative Mender comparison table repeats the same admission. | "The reproducible payload-size, update-time, and flash-write benchmarks (same hardware, same app change, image-updater vs Pantavisor) ship with the first public deliverable." | S1 | `no-example` | Publish at least one worked example with real byte counts for an app-only vs. a kernel-changing revision. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security/atomicity-and-trust` and `https://docs.pantavisor.io/development/pantavisor/reference/pantavisor-configuration` | B, Q1/Q3 — the exact atomic instant and how many boot attempts before giving up | The mechanism is named (bootloader try/rollback via `bootcount` or `grub-editenv` one-shot) but no page ever says how many failed boots it tolerates before giving up, and the configuration reference's only retry-shaped key (`PV_REVISION_RETRIES`, default 10) is documented as revision-transition retries, not a boot-attempt count I can point to as "the" threshold. | "**Bootloader-enforced try/rollback** (U-Boot `bootcount` + a trial/known-good revision pair, or `grub-editenv` one-shot)." | S3 | `undefined-jargon` | State the default (and configurable range, if any) of the boot-attempt threshold next to where `bootcount` is introduced. |

## Closing summary

- **Task outcome:** completed with detours — every question got either a cited answer or
  a cited, explicit absence; getting to the migration entry point required falling back to
  sidebar navigation once, and questions 1–3 required following the trail from
  `migrate/mender` into `pantavisor/overview/*` as the prompt itself anticipated.
- **Worst finding:** `security/atomicity-and-trust` — the one page whose entire purpose is
  proving the atomic-update claim to a skeptical reader — gives one asserted sentence
  ("a power cut leaves the device able to boot *some* good revision") instead of the
  stage-by-stage evidence a reader who tests this with a relay and a script would need
  before trusting it at 40,000-device scale
  (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security/atomicity-and-trust).
- **What worked:** `migrate/mender` — it stated the model difference up front, mapped
  every Mender concept I hold in my head to its Pantavisor equivalent in one table, and
  linked directly to the mechanism page and the revisions page instead of leaving me to
  find them (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/migrate/mender).
- **Confidence:** high. I never assumed an A/B partition pair applied here — the docs
  corrected that mapping explicitly via the concept table — and where a page didn't have
  an answer (sizes, staged rollout, boot-attempt count) I recorded the absence instead of
  filling it in from outside knowledge of Mender/RAUC-style systems I'm not supposed to
  bring to this reading.
