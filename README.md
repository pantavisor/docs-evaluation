# docs-eval — persona-driven documentation gap evaluation

A pack of reader personas and prompts for finding where the Pantavisor documentation
fails its readers.

The premise: you can't tell which gaps matter without knowing who's hitting them. A
Yocto integrator and a Docker developer stall on different pages for opposite reasons —
one because containers are never explained, the other because flashing is assumed. So
rather than audit the docs in the abstract, each persona has an explicit **knowledge
boundary**, and each prompt drives a model through a real task as that reader, reporting
exactly where it stalls and why.

Runs target the live site, `https://docs.pantavisor.io/development/` by default — see
[Which version, and why](#which-version-and-why) below for what that means and how to
point a run at a different published version instead. This repo needs no local checkout
of `pantavisor/`, `meta-pantavisor/`, or `pvr/` and never ships to the docs site itself.

## What's here

| File | Purpose |
|---|---|
| `rubric.md` | Severity scale, gap taxonomy, output contract. **Every run reports against this.** One copy, so runs stay comparable. |
| `ground-truth.md` | Confirmed gaps, mapped to the persona that should catch each. The answer key — used to validate the *prompts*, not the docs. Don't paste it into a run. |
| `personas/01-yocto-no-containers/` … `12-ai-agent-consumer/` | One folder per persona: `persona.md` (scope fence + persona card) plus `promptA.md`, `promptB.md`, `promptC.md`. |
| `RUNBOOK.md` | Entrypoint an unattended or semi-attended Claude Code session (a cron job, `claude -p`, or similar) follows to run one persona/prompt pair and file the report. |
| `answers/` | Accumulated output of runs filed through `RUNBOOK.md` — one flat file per run under `<NN>-<persona-slug>/`, mirroring `personas/`, plus a running index. |

## How to run one

**Manually:**

1. **Open a fresh session.** No local checkout needed — the model just needs a
   web-fetch tool and network access to `docs.pantavisor.io`.
2. **Paste**: `personas/<NN>-<slug>/persona.md` + **exactly one** of
   `promptA.md` / `promptB.md` / `promptC.md` from that same folder. Both files use a
   `{VERSION}` placeholder that resolves to `development` unless you say otherwise — see
   [Which version, and why](#which-version-and-why) if you want a different one.
3. **Collect** the report and file it wherever the current effort keeps them (or drop it
   in `answers/<NN>-<slug>/` following the naming convention in `RUNBOOK.md`
   if you want it to show up alongside the accumulated history).

No runner, no script, no infrastructure required for a one-off run. The prompts are
plain text and work in any model session with fetch access.

**Unattended (any runner):** each trigger names a fixed `persona=<NN> prompt=<A|B|C>`
pair, optionally with `version=<VERSION>` (defaults to `development`), and runs
`RUNBOOK.md` end to end — fetch, report, write to `answers/`, commit. This works the
same whether the trigger is a Hermes cron job, a `claude -p "..."` invocation, or a CI
job — `RUNBOOK.md` doesn't assume a specific runner. See `RUNBOOK.md` for the exact
contract and `## Automated / unattended runs` below for how the pieces fit together.

### Example: running a test from the command line

The quickest way to try the pack is `claude -p`, run from the repo root so the relative
paths in `RUNBOOK.md` resolve. Each invocation is one persona/prompt (/version)
combination and writes its own report under `answers/` — see `RUNBOOK.md` for exactly
what it does at each step.

`RUNBOOK.md`'s later steps write a report file and commit it, so these examples pass
`--permission-mode acceptEdits` to let the session write to `answers/` and run `git
commit` without stopping to ask — swap in whatever your Claude Code setup uses to skip
interactive prompts if that's not it. Without some form of this, the run stalls asking
for permission the moment it tries to write the report file.

```bash
# Core pass: persona 01 (Yocto, no containers), prompt A (cold-start journey),
# default version (development).
claude -p --permission-mode acceptEdits \
  "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=A"

# Same pair, but against a different published version of the site.
claude -p --permission-mode acceptEdits \
  "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=A version=stable"

# Security reviewer, targeted-task prompt — a good smoke test since persona 07
# has no seeded ground-truth gaps yet, so a thin report is expected, not a bug.
claude -p --permission-mode acceptEdits \
  "Follow RUNBOOK.md in the docs-eval repo. persona=07 prompt=B"

# Git-fluent pvr newcomer, jargon audit — the densest ground-truth lane, good for
# checking the pack still finds what it's supposed to after a docs change.
claude -p --permission-mode acceptEdits \
  "Follow RUNBOOK.md in the docs-eval repo. persona=10 prompt=C"
```

**Example: all three prompts for one persona.** A full pass on a persona means three
separate `claude -p` invocations, one per prompt letter — never one script looping
in-process, since each prompt still needs its own cold session (see "One prompt per
session" below; the rule applies to `claude -p` exactly as it does to a pasted-in
session). All three land in the same `answers/01-yocto-no-containers/` folder, one file
per prompt, so the persona ends up with a full comparable set:

| Prompt | What it tests | Command |
|---|---|---|
| A — Cold-start journey | Onboarding and pathing gaps | `claude -p --permission-mode acceptEdits "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=A"` |
| B — Targeted task | Cross-repo task gaps | `claude -p --permission-mode acceptEdits "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=B"` |
| C — Jargon audit | Undefined or misread terms | `claude -p --permission-mode acceptEdits "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=C"` |

**Full command reference — every persona, every prompt.** All 36 combinations follow
the same template:

```bash
claude -p --permission-mode acceptEdits \
  "Follow RUNBOOK.md in the docs-eval repo. <args>"
```

— substitute the `<args>` from the matching cell below. Core set (1–10) is the general
pass; extended set (11–12) is for auditing provisioning and machine-readability
specifically (see "The personas" below for what each boundary means).

| # | Persona | Prompt A `<args>` | Prompt B `<args>` | Prompt C `<args>` |
|---|---|---|---|---|
| 01 | Yocto integrator, no containers | `persona=01 prompt=A` | `persona=01 prompt=B` | `persona=01 prompt=C` |
| 02 | Buildroot engineer, no Yocto, no containers | `persona=02 prompt=A` | `persona=02 prompt=B` | `persona=02 prompt=C` |
| 03 | Cloud-native dev, no embedded | `persona=03 prompt=A` | `persona=03 prompt=B` | `persona=03 prompt=C` |
| 04 | App developer targeting a Pi | `persona=04 prompt=A` | `persona=04 prompt=B` | `persona=04 prompt=C` |
| 05 | OTA engineer migrating off Mender/RAUC | `persona=05 prompt=A` | `persona=05 prompt=B` | `persona=05 prompt=C` |
| 06 | BSP bring-up engineer | `persona=06 prompt=A` | `persona=06 prompt=B` | `persona=06 prompt=C` |
| 07 | Security reviewer | `persona=07 prompt=A` | `persona=07 prompt=B` | `persona=07 prompt=C` |
| 08 | Adoption evaluator (tech lead) | `persona=08 prompt=A` | `persona=08 prompt=B` | `persona=08 prompt=C` |
| 09 | Field support operator | `persona=09 prompt=A` | `persona=09 prompt=B` | `persona=09 prompt=C` |
| 10 | Git-fluent dev meeting pvr | `persona=10 prompt=A` | `persona=10 prompt=B` | `persona=10 prompt=C` |
| 11 | Manufacturing / provisioning engineer | `persona=11 prompt=A` | `persona=11 prompt=B` | `persona=11 prompt=C` |
| 12 | AI agent consumer | `persona=12 prompt=A` | `persona=12 prompt=B` | `persona=12 prompt=C` |

Append ` version=stable` (or any other published version tag) to an `<args>` cell to
run that same combination against a different site version — see "Which version, and
why" below.

**Watching progress while it runs.** Plain `claude -p` prints nothing until the run
finishes, which is a long silence for a task that's fetching several pages and writing a
report. Add streaming flags to watch it work:

```bash
claude -p "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=A" \
  --output-format stream-json --verbose --include-partial-messages
```

This emits newline-delimited JSON events (text as it's generated, each tool call, API
retries) instead of waiting silently for the final result. The output is raw JSON, not
prose — pipe it through `jq` to watch just the generated text, for example:

```bash
claude -p "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=A" \
  --output-format stream-json --verbose --include-partial-messages | \
  jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

`--include-partial-messages` is what gives token-by-token streaming — without it you
only see each message once it's complete. Skip these flags for a normal run; they're
only worth the extra noise when you actually want to watch a `RUNBOOK.md` invocation
fetch pages and write its report live.

To try a single prompt without going through `RUNBOOK.md` at all — no report file, just
the model's raw output in your terminal — pass the persona card and one prompt straight
in:

```bash
claude -p "$(cat personas/01-yocto-no-containers/persona.md) \
$(cat personas/01-yocto-no-containers/promptA.md)"
```

That's the same "paste into a fresh session" flow as the manual steps above, just fed in
from the shell instead of pasted by hand — useful for a quick check that a prompt still
reads sensibly, without touching `answers/` or git at all.

### Three rules that keep results meaningful

**A fresh session per persona.** Context bleed destroys the knowledge boundary. If
persona 1 learns what a container is during its run, persona 2 pasted into the same
session is no longer a reader who's never touched containers — it's a reader who just
read the docs. The boundary is the instrument; don't contaminate it.

**One prompt per session, too.** Prompt C (jargon audit) asks what terms went undefined.
Run it after prompt A in the same session and the answer is "none, I read them all."

**Docs-only scope is not negotiable.** A model with source access answers "how do I claim
a device?" from `cmd/claim.go` and reports the docs as fine. Several confirmed gaps are
*only* visible under this constraint — `pvr claim` is undocumented, pvr's env vars live
in a README, `--runlevel` has zero doc hits. **If a report cites anything outside
`https://docs.pantavisor.io/<VERSION>/`, the fence leaked — discard the run and
tighten it.**

**Same domain isn't automatically in scope.** `docs.pantavisor.io` serves two very
different things a run must tell apart: per-page content, and site-wide AI aids.

- The **"Copy page" button** on every page corresponds to a `.md` export at that page's
  own URL (`<page-url>.md`) — the same article content a human reader sees, just
  cleaner to fetch and quote, and it's what this pack's own `unlinked`/`broken-path`
  findings were verified against (article content, not sidebar chrome). Every persona's
  scope fence (rule 1) tells the run to prefer this over the rendered HTML page.
- The site also serves a root **`/llms.txt`** — a hand-written agent orientation guide
  that explains core concepts directly (e.g. "Pantavisor is PID 1... a `state.json`
  manifest...") and links to bare, non-`/development/` pages. This one is **never** in
  scope: reading it hands the model the exact conclusions this pack exists to test
  whether the *docs page a reader actually lands on* gets across. It's excluded
  explicitly in every persona's scope fence (rule 2), not just implied by the domain
  rule — this file wouldn't otherwise be caught by "no domain other than
  `docs.pantavisor.io`," since it *is* that domain.

### Which version, and why

The live site is versioned, and every persona/prompt uses a `{VERSION}` placeholder
rather than a hardcoded path so a run can target any published version.

**Default: `development`.** Runs target `/development/` unless told otherwise — not the
bare (stable) URL. The stable version is missing entire sections (`getting-started/`
doesn't exist there at all, confirmed 2026-07-23) that every persona prompt uses as an
entry point; `/development/` is the version that's structurally complete. This means a
default run is evaluating docs-in-progress, not what a visitor sees today with no
version selected — worth remembering when weighting a finding.

**Running against a chosen version instead.** Tell the session which version to use
before it starts — manually, by saying so alongside the pasted persona/prompt; or via
`RUNBOOK.md`'s invocation contract, with `version=<VERSION>` (e.g.
`persona=07 prompt=B version=stable`, or a release-candidate tag like `029-rc4`). Every
`{VERSION}` placeholder in the scope fence and the prompt resolves to that value for the
whole run.

**Read a non-default run's findings differently.** A gap on `stable` might just be
`stable` lagging `development`, not a fresh docs defect — the persona/prompt pack itself
is only verified against `development` (see `ground-truth.md`'s provenance note). If a
non-`development` run reports a page or section is missing entirely, that's most useful
as a coarse "how far behind is this version" signal, and worth checking against
`development` before treating it as a new finding. A non-`development` run is a
deliberate, separate audit — don't blend it into the regular weekly cadence of
`development` runs (see `RUNBOOK.md`'s "Suggested cadence").

### Historical: the `CLAUDE.md` fence leak

Earlier versions of this pack ran against local checkouts of `pantavisor/docs/`,
`meta-pantavisor/docs/`, and `pvr/docs/`, and reading files under those repos in Claude
Code caused their root `CLAUDE.md` to auto-inject into context — leaking exactly the
gaps personas 1 and 3 were meant to discover. Now that runs fetch
`docs.pantavisor.io` over the web instead of reading local files, this specific leak
doesn't apply. It's recorded here in case this pack is ever pointed at a local checkout
again.

## The personas

**Core set (1–10)** — run these on a general pass.

| # | Persona | The boundary that makes it useful |
|---|---|---|
| 1 | Yocto integrator, no containers | Knows BitBake cold, has never run `docker run` |
| 2 | Buildroot engineer, no Yocto, no containers | Neither build system the docs assume, nor containers |
| 3 | Cloud-native dev, no embedded | Inverse of 1 — knows OCI, has never flashed anything |
| 4 | App developer targeting a Pi | Hard stop at "build an OS"; the impatience is the instrument |
| 5 | OTA engineer migrating off Mender/RAUC | Maps everything onto A/B partitions until stopped |
| 6 | BSP bring-up engineer | Wants a work estimate; trusts only serial consoles |
| 7 | Security reviewer | *Undocumented means it doesn't exist* — professional standard, held rigorously |
| 8 | Adoption evaluator (tech lead) | Optimises for risk and exit cost, not elegance |
| 9 | Field support operator | Reading at 2am with a customer on the phone; won't read theory |
| 10 | Git-fluent dev meeting pvr | Will be *confidently wrong* — the most valuable failure mode |

**Extended set (11–12)** — run when auditing those angles specifically.

| # | Persona | Boundary |
|---|---|---|
| 11 | Manufacturing / provisioning engineer | 40 seconds per unit; a network dependency stops the line |
| 12 | AI agent consumer | Cannot ask a follow-up, will interpolate confidently, can see the link graph |

## The three prompts

Every persona has the same three, so findings are comparable across readers:

- **A — Cold-start journey.** Land at an entry point, pursue a real goal, stop at the
  first thing you can't resolve. Surfaces onboarding and pathing gaps.
- **B — Targeted task.** One job-to-be-done that crosses a repo boundary, because that's
  where this corpus is weakest.
- **C — Jargon audit.** Every term used before defined, judged strictly from this
  persona's background. The key column is *"what I assumed it meant"* — **a term the
  reader guessed wrong about is worth ten they merely didn't know**, because a reader who
  guesses wrong never comes back to check.

## Reading the results

Findings come back tagged S1–S4 (see `rubric.md`). The distinction that matters most is
**S1 vs S2**: S1 means the answer isn't in the docs at all; S2 means it's there but
nothing led the reader to it. Both are real, and they get fixed completely differently —
one is writing, the other is linking.

Weight by persona, not just severity. An S2 for the security reviewer (had to assemble
the trust model from four pages) may matter more commercially than an S1 for the BSP
engineer, because one of them writes the assessment your buyer reads.

Watch for **agreement across personas**. A gap that only persona 4 hits is a persona
question. A gap that 1, 3, and 4 all hit from different directions — as the missing
Docker→LXC explanation does — is a page that needs to exist.

## Automated / unattended runs

`RUNBOOK.md` is runner-agnostic — anything that can start a cold Claude Code session and
hand it a fixed trigger prompt can drive it: a [Hermes](https://hermes-agent.nousresearch.com/docs)
cron schedule, a one-off `claude -p "Follow RUNBOOK.md ... persona=07 prompt=B"`
invocation, or a CI job. One persona/prompt/version combination per run. Each run:

1. Fetches live, following the scope fence exactly as a manual run would, against
   whichever `version` the trigger named (`development` if it named none).
2. Writes a dated report to
   `answers/<NN>-<persona-slug>/<YYYY-MM-DD>-prompt<X>-<model-slug>-<VERSION>.md`.
3. Appends a row to `answers/index.md`.
4. Commits both — but does not push. Whether and where these commits get synced is
   configured outside this repo.

Triggering a run means giving it a fixed trigger prompt naming one persona and one
prompt letter, and optionally a version — see `RUNBOOK.md`'s **Invocation contract** for
the exact text. A single "run the eval" trigger with no parameters isn't supported on
purpose: fixed combinations are what keep runs across weeks comparable.

Because each such invocation is a cold Claude Code session, "a fresh session per
persona" and "one prompt per session" (above) hold automatically — there's no risk of
the context bleed a human running several personas back-to-back in one chat has to watch
for.

Read `answers/index.md` over time, not just the latest run: a finding that stops
reproducing after a docs change is as informative as a new one, and it's the only way to
tell a fixed doc from a broken prompt without re-reading every report.

## Maintaining the pack

**When a run surfaces a new gap, verify it and add it to `ground-truth.md`.** The set
should grow as personas teach us about readers we hadn't modelled.

**When a persona misses a gap in its own lane, fix the prompt, not the docs.** That
inversion is why `ground-truth.md` exists. A pack that finds nothing is indistinguishable
from documentation with no gaps, and the whole thing is worthless the moment you can't
tell those apart.

**When a gap gets fixed, move it to Resolved rather than deleting it** — otherwise you
can't distinguish a fixed doc from a broken prompt on the next run.

Personas 5 and 7 currently have **no seeded gaps** in their lanes. They're exploratory:
we have no evidence about those readers yet, which is itself worth knowing. A thin report
from them is a first data point, not a prompt failure.

## Does the pack work? — dry-run results, 2026-07-16

Three prompts were run in isolated sessions before this pack was considered done.
*(Predates the 2026-07-23 retarget to the live site — these runs were against local
`docs/` checkouts. Kept as evidence the prompts themselves work; see the provenance note
in `ground-truth.md` for what's changed since.)*

**It finds seeded gaps.** Persona 10 prompt B independently found the unlinked
`--status-goal` enum and traced the claim trail to its dead end at
`wifi.md:129`. Persona 1 prompt C found the orphaned glossary and orphaned
`composable-firmware.md` without being told either existed.

**It finds gaps its author didn't know about.** The dry-runs surfaced four new defects,
since verified and added to `ground-truth.md` as #13–16 — including a genuine cross-repo
contradiction about what `pvr deploy` does, and a link at `boot-flow.md:33` that promises
"the trail/revision model" and points at a page containing the word "trail" zero times.
This is the result that justifies the pack: it is not merely a checklist of what we
already knew.

**It produces findings a gap list wouldn't.** Persona 10's best finding was not that
`--status-goal` is unlinked — it's that a reasonable engineer picks `READY` for a web
server *because it reads as the strongest value*, and stock `nginx:alpine` never emits
the required readiness signal, so the revision reproducibly rolls itself back. That's a
production incident derivable only by making a reader actually choose.

**The personas are genuinely different readers.** Persona 1 and persona 3 were run on the
same prompt (C) as a comparability check. Their term lists overlap by roughly a quarter —
but the overlap is entirely *product* jargon nobody knows (`pvrexport`, `pvtx`, `trail`),
which both readers *should* flag. On the false friends they are near-perfect inverses:
persona 1 reads "image" as a flashable disk image and is wrong about the Docker sense;
persona 3 reads it as a container image and is wrong about the disk sense. Persona 1 reads
"state" as Yocto sstate; persona 3 reads it as Kubernetes desired state.

**Refinement to the comparability test:** don't test for disjoint *term* lists — shared
product jargon is a correct result. Test for disjoint **wrong guesses**. If two personas
guess wrong the same way, their boundaries aren't real and the cards need sharper "never
touched" sections.
