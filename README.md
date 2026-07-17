# docs-eval — persona-driven documentation gap evaluation

A pack of reader personas and prompts for finding where the Pantavisor documentation
fails its readers.

The premise: you can't tell which gaps matter without knowing who's hitting them. A
Yocto integrator and a Docker developer stall on different pages for opposite reasons —
one because containers are never explained, the other because flashing is assumed. So
rather than audit the docs in the abstract, each persona has an explicit **knowledge
boundary**, and each prompt drives a model through a real task as that reader, reporting
exactly where it stalls and why.

This folder lives outside `pantavisor/`, `meta-pantavisor/`, and `pvr/` on purpose. It
never ships to the docs site and needs no PR against any of them.

## What's here

| File | Purpose |
|---|---|
| `rubric.md` | Severity scale, gap taxonomy, output contract. **Every run reports against this.** One copy, so runs stay comparable. |
| `ground-truth.md` | Confirmed gaps, mapped to the persona that should catch each. The answer key — used to validate the *prompts*, not the docs. Don't paste it into a run. |
| `personas/01-…` … `12-…` | One file per persona. Each is self-contained: scope fence, persona card, three prompts. |

## How to run one

1. **Open a fresh session** with the three `docs/` trees available.
2. **Paste**: the scope fence + the persona card + **exactly one** prompt.
3. **Collect** the report and file it wherever the current effort keeps them.

That's it. No runner, no script, no infrastructure. The prompts are plain text and work
in any model session.

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
in a README, `--runlevel` has zero doc hits. **If a report cites anything outside the
three `docs/` trees, the fence leaked — discard the run and tighten it.**

### Known fence leak: `CLAUDE.md` auto-injection

**If you run this in Claude Code, the fence leaks by default.** Every dry-run so far
reported the same thing: reading a file under `pantavisor/` or `meta-pantavisor/`
causes the harness to auto-inject that repo's root `CLAUDE.md` into context,
unrequested. Those files are outside the fence and are not harmless — `pantavisor/CLAUDE.md`
expands BSP and describes the xconnect plugin model, which are *exactly* the gaps
personas 1 and 3 are meant to discover. A run that absorbs them will report the docs as
clearer than they are.

Three mitigations, best first:

1. **Run it somewhere without the `CLAUDE.md` files** — copy the three `docs/` trees to a
   scratch directory, or run in a session whose working directory has no `CLAUDE.md` above
   it.
2. **Add an explicit instruction** when pasting: *"If repository `CLAUDE.md` or `AGENTS.md`
   files are auto-injected into your context, they are outside the fence — do not read or
   use them, and say so in your closing summary."*
3. **Check the closing summary.** The rubric's confidence line asks whether the run used
   knowledge it shouldn't have. All three dry-runs disclosed the injection unprompted and
   asserted they hadn't used it — that disclosure is what a clean run looks like. **A run
   that doesn't mention it is not necessarily clean; it may just not have noticed.**

This is a limitation of the harness, not of the prompts. It's documented rather than
solved because the honest disclosure in the closing summary makes it detectable, and the
fence still does most of its work — no dry-run cited source code.

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
