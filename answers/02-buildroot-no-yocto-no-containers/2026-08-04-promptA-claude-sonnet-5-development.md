# Persona 02 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p (RUNBOOK.md automated invocation)

## Narration

Landed on the Start page as instructed. It's a "flash a real device and ship an
update" quick-start with three paths (flash a Raspberry Pi, run in Docker/AppEngine,
install `pvr`) and nothing yet about how images get built — no mention of Yocto,
BitBake, or Buildroot anywhere on this page.

Followed the first path, **Download and flash on a Raspberry Pi**, since the page
frames it as "the fastest way to understand Pantavisor." That page is a clean,
self-contained pre-built-image walkthrough (download a `.wic.bz2`, flash it with
`pvflasher`, boot it) — no build system touched at all. Good sign, but it's a demo on
someone else's board, not my "next product." I want to know what happens when it's
*my* hardware.

The page's compatibility callout pointed me at the **Supported Devices** page. My
board isn't on the list (obviously — it doesn't exist yet), but the page has a
"Don't See Your Board?" section pointing at porting guides, and, more importantly, a
**"Building for a Device"** section with a **Get Started** link promising "your first
build." That's the link that actually answers my question, so I followed it.

**Get Started** is where it falls apart for me. Prerequisites are just "Docker, Git,
~50GB disk" — no mention of Yocto experience being expected. Then immediately: git
worktrees for parallel `kas/bitbake` builds, `SSTATE_DIR`/`DL_DIR` concurrency
semantics, `kas-container`, `devtool` workspaces, `TMPDIR`, `bitbake.lock`,
`hashserve.sock`. None of `kas`, `bitbake`, `sstate`, `recipe`, `layer`, or `poky` is
defined anywhere on this page. I only recognize two of these words at all — "Yocto"
itself appears twice, once in a parenthetical about directory-locking semantics
("Yocto's `SSTATE_DIR` and `DL_DIR` are designed for concurrent access...") and once
inside a code-adjacent aside ("If you include meta-pantavisor in your own **Yocto
project**..."). Neither is a framing statement — I have to notice both, in passing,
to conclude this is Yocto/BitBake under the hood. There's a "Direct BitBake (for
integrators)" section, but that's still BitBake — no alternate command lane for
Buildroot.

I remembered the Start page mentioned a glossary (for "AppEngine"), so I checked it
for `kas`/`bitbake`/`Yocto`/`layer`/`recipe`/`sstate` before giving up on definitions.
It's a real, alphabetized glossary — but none of those terms are in it. `BSP` is
defined, but only as "built from meta-pantavisor," which doesn't explain what
meta-pantavisor *is* in build-system terms.

Last thing I checked before deciding: the Start page's other primary path, "Run
Pantavisor in Docker (AppEngine)," in case it's a way to evaluate the product without
touching any of this. It isn't — that page is a test-harness doc (`test.docker.sh`,
local/remote test suites) that either points you back at `get-started.md` to build
the appengine tarball yourself (Yocto again), or off-site to
`docs.pantahub.com/requirements-appengine` for "full requirements and installation."

**Verdict:** I have my answer, and it's not the one I wanted. Building a real image
for real hardware runs entirely through meta-pantavisor's Yocto/BitBake tooling —
`kas`, `bitbake`, `sstate`, recipes, all of it. Buildroot is never mentioned as an
option, a migration path, or even acknowledged as something a reader might be coming
from. I'd tell my boss: this looks like it works and the update model is genuinely
different from what we do today, but adopting it is a build-system rewrite from
Buildroot to Yocto, not a bolt-on — budget for that, or don't do this. I'm closing
the tab; I got the yes/no I came for, but the docs never actually told me — I had to
piece it together from two off-hand mentions of a word I only recognize because I
already knew Yocto existed.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/get-started` | A — deciding whether adoption means abandoning Buildroot | Nothing on this page or anywhere I reached from it ever states plainly "this build system is Yocto/BitBake" or says a word about Buildroot; I had to notice two passing uses of "Yocto" in unrelated technical asides and infer the rest myself. | Prerequisites list is just "Docker installed and running / Git configured / ~50 GB free disk space for builds" — no mention of Yocto/BitBake background, and "Buildroot" appears nowhere on the page. | S1 | `missing-concept` | Add a one-line framing statement up front ("meta-pantavisor builds images using Yocto/BitBake via kas") and, ideally, a line for readers coming from Buildroot about what carries over and what doesn't. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary` | A — looking up unfamiliar terms hit on the Get Started page | I checked the glossary for `kas`, `bitbake`, `sstate`, `Yocto`, `layer`, and `recipe` after hitting them unexplained on the build page — none are entries, despite the glossary explicitly promising "the terms you will meet throughout these docs." | Glossary entries jump from "Groups" to "Object" — no `kas`, no `bitbake`, no `Yocto`, no `layer`, no `recipe`, no `sstate`. | S1 | `undefined-jargon` | Add glossary entries (or links to a Yocto-101 primer) for `kas`, `bitbake`, `layer`, `recipe`, and `sstate` since the build guide assumes all five. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/how-to-install/docker` | A — checking the "no hardware" path as a possible Yocto-free evaluation route | The page's own pointer for "full requirements and installation options" sends me off the docs site entirely, and the only in-scope way to get a runnable image still routes back through the Yocto build guide. | "For full requirements and installation options see: **docs.pantahub.com/requirements-appengine**" and "To build the tarball yourself, see get-started.md." | S2 | `outside-docs` | Either bring the AppEngine requirements/install steps onto docs.pantavisor.io, or state clearly that a pre-built appengine tarball isn't available and building one requires the Yocto toolchain. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start/download-and-flash` | A — following the "other boards" pointer for non-Raspberry-Pi hardware | The sentence about board-specific guides names a target but gives me nothing to click. | "The same principles apply to other boards — seeInstall on hardware for board-specific guides." (no link, text runs straight into the next word) | S4 | `broken-path` | Turn "Install on hardware" into an actual link to the how-to-install index. |

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding:** on `.../meta-pantavisor/overview/get-started`, the page that's supposed to be my first build never states that the toolchain is Yocto/BitBake or says anything about Buildroot — I had to infer it from two off-hand mentions of the word "Yocto" buried in unrelated paragraphs about git-worktree sstate sharing, and even then got no sense of migration cost.
- **What worked:** `.../getting-started/start/download-and-flash` — a clean, complete, self-contained walkthrough for trying Pantavisor on a Raspberry Pi with a pre-built image; no build system touched, nothing assumed I knew Yocto.
- **Confidence:** mostly stayed in character. I recognized "Yocto" and "BitBake" as loaded terms only because the persona card lets me know Yocto exists by name even though I've never used it — that's within the stated boundary, not a break of it. I did not use any deeper Yocto/BitBake knowledge (I don't know what `sstate` or a `recipe` actually is, and said so).
