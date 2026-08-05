# Persona 08 — Prompt B — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narrative

Prompt B hands me six fixed questions, so I went hunting question by question rather
than reading front-to-back. The homepage itself answered a chunk of question 2 up
front — a dedicated "Pantahub" section stating "Pantahub is optional. Pantavisor
updates a device fully standalone over the local network" — which is exactly the
distinction my mental model was primed to miss. It did not, however, say whether
Pantahub is open source or self-hostable, so I kept digging.

`pantavisor/overview/remote-control` (the runtime-side page named for exactly this
topic) turned out to be a dead end for the open-source/self-host half of the
question — it documents the Hub client protocol in detail but never states Hub's
license or self-hosting status, and links off-domain to
`github.com/pantacor/pantahub-base` for "the API" without saying what that repo's
terms are. I found the real, direct answer two hops later, off a completely
different trail: `getting-started/security` → `solutions/secure-ota`'s
"Self-hosting for sovereignty" section states plainly "self-host Pantahub — it is
fully open source." I only reached that page because I was chasing the trust/atomicity
angle, not the vendor-risk angle — a reader asking my exact question ("is this a
library or a vendor") wouldn't naturally end up on an OTA-security implementation
page to find it.

Once there, I went back to the top-level sidebar (falling back past the article-only
`.md` export, the way I would if I'd run out of in-page links) to check whether a
more direct page existed, since this is supposed to be the most important question
in my evaluation. It does: "Project, licensing, and governance" at
`getting-started/licensing`, which states the MIT/Apache-2.0 split per project and
says outright "You can self-host the open components; the commercial relationship is
the managed service and support, not a closed core." I checked every page I'd
already read for an in-content link to this page — the homepage, `remote-control`,
`security`, `benchmarks`, `migrate`, `migrate/when-not-to-use`, and the FAQ — and
found none. It is not reachable by following links from where a reader like me would
naturally go looking for it.

Question 5 (limits) sent me to `migrate/when-not-to-use`, which is the strongest
page on the whole site for my bias — it states the boundary in plain language
("Pantavisor is the Linux init process and container runtime," "no supported
hybrid," "use Docker/Podman/Kubernetes" for datacenter workloads) and preempts
objections instead of just listing them. Per the prompt's own instruction I checked
the `pantavisor/` runtime section too (`pantavisor-architecture`, `local-control`,
`init-mode`) for a second, runtime-side limits statement — there isn't a dedicated
one, but nothing there contradicts the meta-pantavisor page either, so I treat
question 5 as answered from the meta-pantavisor side alone.

Question 1 (team capability) never got a direct answer. The `start` page's
prerequisites list is entirely about hardware and tooling (Pi, Docker, Git, disk
space) — no statement of assumed skill level anywhere. I had to piece together an
answer myself: `get-started` (the Yocto build guide) assumes BitBake/KAS familiarity
without saying so; `container-development` and the glossary define LXC/container/
AppEngine inline but never say Docker knowledge is (or isn't) sufficient.

Question 6 (maturity) came up completely empty. I checked the homepage, the
`security`/`atomicity-and-trust` pages (the ones whose whole job is to earn trust),
the `benchmarks` page, and finally the `community` page (support/contribute/social
links) — none state who runs this in production, at what scale, or for how long.
The closest thing to a claim is the community page's "outstanding projects may be
featured... in case studies," which is an invitation, not evidence.

Question 4 (time to first value) has a clean, fast answer for supported hardware —
the `start` page's own "~30 minutes" — but for the realistic case of a new device
line (hardware not in the CI-supported list), the `Porting Pantavisor` guide walks
through three concrete steps with zero time or expertise estimate anywhere.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/community` | B, Q6 — maturity | I need to know who else runs this and at what scale before betting nine months on it, and the page whose job is community/trust content never mentions a single production user, deployment count, or duration — only an invitation to submit case studies in the future. | "Outstanding projects may be featured in the official documentation, case studies, and community highlights." | S1 | `no-example` | Publish at least one named production deployment (even anonymized: industry + fleet size + years in field) somewhere reachable from Security, Benchmarks, or Community. |
| `https://docs.pantavisor.io/development/meta-pantavisor/overview/port/` | B, Q4 — time to first value, longest pole | My actual hardware is very unlikely to be on the CI-supported list for a new device line, and the porting guide details three concrete steps (platform, machine, build) with no estimate anywhere of how long that takes or what expertise it needs beyond the build guide's own scope. | "Adding a new device involves three steps, each covered in its own page" — no duration or skill-level statement follows. | S1 | `missing-concept` | State a rough time/expertise range for porting to a new BSP (even "a few days for an experienced Yocto engineer, longer without one") next to the three-step outline. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/licensing` | B, Q2 — the cloud question | This is supposed to be the single most important question in my evaluation, and its cleanest, most direct answer sits on a page nothing I read links to — not the homepage's Pantahub blurb, not `remote-control`, not `security`, not `benchmarks`, not `migrate`, not the FAQ. I only found it by falling back to sidebar navigation after the article-content trail ran out. | "You can self-host the open components; the commercial relationship is the managed service and support, not a closed core." | S2 | `unlinked` | Link the Licensing page from the homepage's Pantahub section and from `pantavisor/overview/remote-control`, the two places this question actually gets asked. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/remote-pantahub` | B, Q3 — exit cost | I need to know what happens two years and 20,000 devices in if I want out — does a fleet keep running, does anything expire or phone home if I unclaim or stop paying. This page covers claiming a device in detail but never mentions unclaiming, removal, or what happens on account cancellation; I had to infer continuity from a different page's mention of local-control fallback rather than get a direct answer here. | Page covers `pvr login`, device listing, and claiming end to end but has no section on unclaiming, removal, or account-level effects. | S3 | `missing-concept` | Add a short "leaving Pantacor Hub" section stating explicitly what happens to a device/fleet on unclaim or account closure. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start` | B, Q1 — team capability | I need to know what each of my three engineer groups (Yocto-literate, container-illiterate, neither) has to learn and roughly how long, and the entry-point page's prerequisites list only covers hardware and tooling (Pi, Docker, Git, disk space) — no statement of assumed skill level anywhere on the page. | Prerequisites: "A Raspberry Pi 3B/3B+/4... or Docker for the no-hardware path... A laptop or desktop to download and flash the image." — no mention of assumed prior knowledge. | S3 | `missing-concept` | Add one line to the Start page's prerequisites stating what background (Yocto? containers? neither?) each path assumes. |
| `https://docs.pantavisor.io/development/pantavisor/overview/remote-control` and `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting/faq` | B, Q2 — the cloud question | Chasing "is Pantahub open source" led me to two off-domain links (`github.com/pantacor/pantahub-base` and, on the FAQ, `docs.pantahub.com`) that this evaluation's scope fence doesn't let me follow — the actual self-hosting how-to, if one exists, lives entirely off `docs.pantavisor.io`. | Remote Control: "the [API](https://github.com/pantacor/pantahub-base)"; FAQ: "[Pantahub Documentation](https://docs.pantahub.com/)" | S4 | `outside-docs` | If Pantahub self-hosting instructions exist, mirror or link them from a page inside `docs.pantavisor.io`, not only via an off-domain repo/site. |

## Absence of findings — what worked

- **`migrate/when-not-to-use`** is the best page on the site for this persona. It
  states the boundary in plain language ("Pantavisor is the Linux init process and
  container runtime," no support for "bare-metal MCUs, RTOS systems"), calls out the
  no-hybrid constraint I was already suspicious of, and explicitly disclaims things
  that are *not* reasons to avoid it — which reads as more trustworthy than a page
  that only lists reasons to buy.
- The homepage's **Pantahub section** answers the standalone-vs-cloud half of
  question 2 in two sentences with zero digging: "Pantahub is optional. Pantavisor
  updates a device fully standalone over the local network."
- **`pantavisor/overview/local-control`** confirmed, directly, that local control
  keeps working "even if the device is already claimed in Pantacor Hub" and "can
  also be the only option if you disable remote control" — real evidence against a
  hard cloud dependency, not just a marketing assertion.
- **`start`** gives a fast, concrete time-to-first-value number for the supported-
  hardware path: "about 30 minutes" from flash to booted device.

## Closing summary

- **Task outcome:** completed with detours — all six questions got either a cited
  answer or a cited, explicit absence; question 2's real answer required leaving the
  in-content link trail entirely and falling back to sidebar navigation, and question
  6 came back empty despite checking every trust-oriented page on the site.
- **Worst finding:** the `community` page — the one page whose job is community/
  trust signal — never names a single production deployment, fleet size, or duration
  in the field anywhere I could find, leaving the "who else has bet on this" question
  a total blank for a nine-month, name-on-it commitment
  (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/community).
- **What worked:** `migrate/when-not-to-use` — it states the product's real
  boundaries in plain language and preempts the objections I came in planning to
  raise myself
  (https://docs.pantavisor.io/development/meta-pantavisor/getting-started/migrate/when-not-to-use).
- **Confidence:** high. I did not assume prior knowledge of Yocto, LXC, or
  content-addressed state beyond what each page explained inline, and I checked both
  `pantavisor/` and `meta-pantavisor/` for question 5 as instructed rather than
  stopping at the first section that answered it. Where sidebar fallback was
  involved (finding the Licensing page), I flagged that as the detour it was rather
  than quietly treating it as a normal in-content find.
