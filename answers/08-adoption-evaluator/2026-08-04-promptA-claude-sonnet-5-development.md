# Persona 08 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: manual (fresh session, docs-framework repo)

## Narration — what I hunted for and whether I found it

Landed on the `start` page as instructed. It gives three paths (flash a Pi in ~30
minutes, run in Docker/AppEngine on a workstation, or use `pvr` against an existing
device) and one reassuring distinction up front — Pantavisor replaces the whole
update stack, no separate A/B updater underneath. That's a fast, concrete
time-to-first-value number for question 3, so I moved straight to the sidebar and
went hunting for the bad news, in this order: **licensing** (the open-source/vendor
boundary I was told I'd have to work to find), **benchmarks and comparisons**
(question 2's "what am I locked into," plus the Yocto-fit question for question 1),
**security and compliance**, **migrate → when not to use** (the actual "when not to
use" page the prompt told me to go looking for), **solutions**, and the **glossary**
(for "content-addressed," a term I was told I've never heard of). I skipped the
build-system, boot-flow, and CLI-reference material under `overview/` — that's
implementation detail my engineers will read, not me.

The licensing page and the "when not to use" page are the two best pages on the
site for this task — both answer exactly the question I brought to them, in plain
language, without me having to infer anything. The Yocto comparison page directly
answers "can my team do this." The benchmarks table has a clean, legible
open-source/offline-operation row that answers the lock-in question favorably. The
gaps I found are about unpublished evidence and one misleading front-door framing,
not about the product hiding something — which is itself the finding the prompt
told me to report explicitly if I found it.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security` | A, hunting for compliance/vendor-risk answers before recommending a foundation for a regulated device line | I need to know whether secret management, CVE handling, and Cyber Resilience Act / IEC 62304 / IEC 62443 positioning are even addressed before I'd stake my name on this for a new device line, and the page whose whole job is to tell me that says it's future work, not present content. | "**Planned coverage** — Upcoming pages will address secret management, SBOM generation and CVE/update processes, the recertification model..., Cyber Resilience Act alignment, and IEC 62304/IEC 62443 positioning." | S1 | `missing-concept` | Publish at least a stub/roadmap page per planned topic instead of one bullet list, or pull the "heightened security standard" framing until secret management and CVE process are written. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/benchmarks` | A, hunting for the efficiency/lock-in evidence to back a nine-month commitment | I came in looking for a reason to say no, and the one page built to give me independently reproducible numbers instead tells me the numbers don't exist yet — I have to take the comparison table on faith. | "**Note — measured numbers coming.** The reproducible payload-size, update-time, and flash-write benchmarks... ship with the first public deliverable. Until then, treat the table above as the shape of the comparison, not as published figures." | S1 | `missing-concept` | Publish the promised numbers, or mark the comparison table itself as directional/unverified rather than presenting it next to a claim of reproducibility. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start` → `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker` | A, checking the "Docker Alternative" the front door offers to evaluators without hardware | The front door sold me a no-hardware path as the alternative to flashing a Pi. Six of my eight engineers don't know Yocto, so I clicked it hoping to hand this to one of them for a same-day look — instead the page tells me to go build the image myself through the Yocto/BitBake toolchain first, the exact thing the "alternative" was supposed to let me skip. | Start page: "For those without hardware, running Pantavisor in Docker (AppEngine) on a workstation is available." Docker page: "There's no pre-built AppEngine tarball to download — build one yourself via Get Started (this repo's Yocto/BitBake toolchain, build target `pantavisor-appengine-distro`), then continue below." | S3 | `wrong-audience` | Either ship a pre-built AppEngine tarball for evaluators, or change the Start page's framing so "Docker Alternative" doesn't imply it's the lower-friction, no-Yocto option. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/solutions/reproducible-builds`, `.../benchmarks` | A, trying to understand the "content-addressed" claim that recurs across the pages making the efficiency case | "Content-addressed" is the load-bearing word behind most of the efficiency and dedup claims I was evaluating, and I was told going in I've never heard of content-addressed device state. Each page re-explains the mechanic inline well enough that I wasn't blocked, but the one glossary entry that actually defines it ("Object") is never linked from any of the pages that use the term. | Glossary: "**Object** — A content-addressed file blob, named by its SHA-256 hash." Benchmarks page: "Pantavisor versions the device as content-addressed objects..." — no link to the glossary entry. | S4 | `undefined-jargon` | Link "content-addressed" to the glossary's "Object" entry on first use in the benchmarks, solutions, and reproducible-builds pages. |

## Absence of findings — what worked

- **Licensing** (`.../getting-started/licensing`) answered the open-source/vendor
  boundary in two sentences with no digging required: "Pantavisor code that runs on
  the device → MIT. Backend services → Apache-2.0," plus an explicit statement that
  "the underlying code repositories are fully open source. The commercial offering
  is the managed Pantacor Hub service, not the software licensing itself.
  Organizations may self-host the open components." This is exactly the answer I was
  told I'd have to fight for, and I didn't have to fight for it.
- **When not to use Pantavisor** (`.../migrate/when-not-to-use`) is the single best
  page on the site for my bias. It states the boundary plainly ("You cannot own PID 1
  or reflash the device... there is no supported hybrid"), and pre-empts three
  objections I would have raised myself, including the exact one I was primed for —
  "My apps are Docker images" — with a direct answer instead of silence.
  It also explicitly disclaims the topics that aren't reasons to avoid the product,
  which reads as more trustworthy than a page that only lists reasons to buy.
- **Pantavisor vs Yocto** (`.../benchmarks/vs-yocto`) directly answers "can my team
  do this": a "When to choose Yocto alone" list includes "Your team has no container
  expertise," stated as a legitimate reason to *not* add Pantavisor — a vendor page
  telling me when not to buy is unusual and works in its favor.
- The **benchmarks comparison table**'s "Offline operation" (✅ Full) and "Open
  source" (✅ 100%) rows, read together with the licensing page, directly answer the
  cloud-dependency half of question 2 — I did not find a hidden mandatory
  cloud-service dependency anywhere I looked.

## The one-page recommendation

**Recommendation: Prototype first. Do not adopt outright, and don't say no yet
either.**

**Can my team do this?** Yes, with the team I have. The one-time BSP/Yocto build is
owned by the two engineers who already know Yocto (`vs-yocto` page is explicit that
Pantavisor is a Yocto companion, not a replacement — build once, then app-level
iteration via `pvr` doesn't require deep container expertise). The one wrinkle: the
docs frame a Docker-based, no-hardware evaluation path as available to engineers
without Yocto knowledge, and it isn't — it still needs the Yocto/BitBake toolchain.
Get a Raspberry Pi instead; the flash-and-boot path genuinely needs no Yocto
knowledge and is documented at ~30 minutes.

**What am I locked into?** Less than I expected, on paper. On-device code is MIT,
backend is Apache-2.0, and the vendor's own docs say the backend (Pantahub) is
self-hostable — the managed service is the commercial product, not the code. The
comparison table also claims full offline operation with no forced cloud
dependency. I went looking for the trap here and didn't find one in the
documentation; I'd still want that self-hosting claim demonstrated, not just
stated, before I fully believe it.

**When do I know it's working?** Days, not a quarter, per the docs' own numbers:
~30 minutes to first boot on a Pi, and a 2–5 minute container update cycle after
that (versus a 30–60 minute full Yocto rebuild without Pantavisor). That's fast
enough to get a real answer inside a one-week prototype.

**What's actually missing, and why it's not a "no":** two things kept this from
being a clean "adopt" — the efficiency numbers backing the vendor's own comparison
claims aren't published yet, and the compliance content (secret management, CVE
process, CRA/IEC 62304/IEC 62443 positioning) is explicitly listed as future work.
**Both of these are documentation and publication gaps, not evidence of product
gaps** — the "when not to use" page is specific and credible enough that I trust
the vendor to be straight with me about limitations, which raises my confidence
that the missing pieces are genuinely just not written yet rather than a
mechanism to hide. I would not let either gap alone kill this, but I would not
sign a nine-month commitment on the strength of unpublished numbers either.

**The ask:** one Yocto-literate engineer plus one app engineer, one Raspberry Pi,
one week. Get a real prototype booted and treat that week's own numbers as the
actual dataset for the vendor's still-unpublished benchmark claims. In parallel,
ask the vendor directly for their SBOM/CVE/CRA roadmap before any production
commitment — that's not answerable from the docs today.

## Closing summary

- **Task outcome:** completed.
- **Worst finding:** the Security and compliance section — the page whose entire
  premise is that Pantavisor "meets a heightened security standard" because it runs
  as PID 1 — lists secret management, SBOM/CVE process, and Cyber Resilience
  Act/IEC 62304/IEC 62443 positioning as "planned," not present, leaving the
  compliance half of a new-device-line risk assessment unanswerable from the docs
  alone (`https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security`).
- **What worked:** `.../getting-started/migrate/when-not-to-use` — the single page
  that most directly earned my trust, by stating the product's real boundaries in
  plain language and pre-empting the objections I came in planning to raise myself.
- **Confidence:** high. I did not use any prior knowledge of Yocto, containers, or
  content-addressed storage beyond what each page explained inline — where a term
  (content-addressed) was used without a link to its definition, I logged it as a
  finding rather than quietly relying on outside knowledge of hash-addressed
  storage systems.
