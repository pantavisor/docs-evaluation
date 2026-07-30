# docs-eval — persona-driven documentation gap evaluation

A pack of reader personas that drive a model through real tasks against the live
Pantavisor docs, reporting exactly where each reader stalls. `RUNBOOK.md` runs a check
and files a report; `FIXBOOK.md` turns one finding from that report into a draft PR
against the repo that actually owns the affected page.

Runs target `https://docs.pantavisor.io/development/` by default (see
[Versions](#versions) to target another one). No local docs checkout is needed to run a
check — only `FIXBOOK.md` needs one, to make the actual edit.

## What's here

| File | Purpose |
|---|---|
| `personas/01-yocto-no-containers/` … `12-ai-agent-consumer/` | One folder per persona: `persona.md` (scope + persona card) plus `promptA.md`/`B`/`C`. |
| `RUNBOOK.md` | Runs one persona/prompt pair against the live docs and files the report. |
| `answers/` | One report file per run, under `<NN>-<persona-slug>/`, plus `index.md`. |
| `FIXBOOK.md` | Turns one finding from an `answers/` report into a draft PR in the owning repo. |
| `rubric.md` | Severity scale, gap taxonomy, report format. Every run reports against this. |
| `ground-truth.md` | Confirmed gaps mapped to the persona that should catch each — the answer key for validating *prompts*, not docs. |

## Check the docs (find gaps)

```bash
claude -p --permission-mode acceptEdits \
  "Follow RUNBOOK.md in the docs-eval repo. persona=01 prompt=A"
```

`--permission-mode acceptEdits` lets the run write its report and commit without
stopping to ask (swap in whatever your setup uses instead). One command = one persona +
one prompt letter; each writes its own file to `answers/<NN>-<slug>/`. A full pass on
one persona is three separate invocations (`prompt=A`, `B`, `C`) — never batched, since
each needs its own cold session (see [Rules](#rules-that-keep-results-meaningful)).

**Personas** — pick a `persona=<NN>`:

| # | Persona | Boundary |
|---|---|---|
| 01 | Yocto integrator, no containers | Knows BitBake cold, never run `docker run` |
| 02 | Buildroot engineer, no Yocto, no containers | Neither build system the docs assume, nor containers |
| 03 | Cloud-native dev, no embedded | Inverse of 01 — knows OCI, never flashed anything |
| 04 | App developer targeting a Pi | Hard stop at "build an OS" |
| 05 | OTA engineer migrating off Mender/RAUC | Maps everything onto A/B partitions until stopped |
| 06 | BSP bring-up engineer | Wants a work estimate; trusts only serial consoles |
| 07 | Security reviewer | *Undocumented means it doesn't exist* |
| 08 | Adoption evaluator (tech lead) | Optimises for risk and exit cost |
| 09 | Field support operator | Reading at 2am, won't read theory |
| 10 | Git-fluent dev meeting pvr | Will be *confidently wrong* |
| 11 | Manufacturing / provisioning engineer *(extended)* | 40 seconds per unit |
| 12 | AI agent consumer *(extended)* | Can't ask a follow-up, sees the link graph |

**Prompts** — pick a `prompt=<A|B|C>`:

- **A** — Cold-start journey: land at an entry point, stop at the first thing unresolved.
- **B** — Targeted task: one job crossing a repo boundary.
- **C** — Jargon audit: every term used before it's defined.

**Watching progress:** plain `claude -p` prints nothing until the run finishes. Add
`--output-format stream-json --verbose --include-partial-messages` and pipe through
`jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'`
to watch the text as it's generated.

**Reading results:** findings are tagged S1–S4 (`rubric.md`). S1 = the answer isn't in
the docs at all; S2 = it's there but nothing led the reader to it — those get fixed
differently (writing vs. linking). Watch for the same gap surfacing across personas from
different angles; that's a page that needs to exist, not a one-off.

## Fix a finding (turn an answer into a docs PR)

`docs.pantavisor.io`'s own repo (`docs.pantavisor`) is **not** where fixes go — its
`reference/` content is generated at build time from a release tarball, not committed
source, so a PR there is silently overwritten on the next build. The real source lives
in whichever repo owns the URL path segment:

| URL segment | Repo | Docs live under |
|---|---|---|
| `meta-pantavisor/...` | `github.com/pantavisor/meta-pantavisor` | `docs/` |
| `pantavisor/...` | `github.com/pantavisor/pantavisor` | `docs/` |
| `pvr/...` | `gitlab.com/pantacor/pvr` | `docs/` |

```bash
claude -p --permission-mode acceptEdits \
  "Follow FIXBOOK.md in the docs-eval repo. Fix this finding: \
   answers/01-yocto-no-containers/2026-07-30-promptA-claude-sonnet-5-development.md, \
   the undefined-'trail' row for meta-pantavisor/overview/images"
```

One finding per invocation — same reasoning as one prompt per `RUNBOOK.md` session,
so a fix is reviewable and scoped. `FIXBOOK.md` locates the real source file, re-checks
the gap still exists, makes the minimal edit, and opens a **draft PR** — it never
merges. See `FIXBOOK.md` for the full routing logic and edge cases.

## Versions

Every persona/prompt uses a `{VERSION}` placeholder, resolved to `development` unless
told otherwise. Append `version=stable` (or an RC tag like `029-rc4`) to a `RUNBOOK.md`
invocation to target a different published version. `development` is the version this
pack's `ground-truth.md` was verified against; a gap on another version may just be that
version lagging behind, not a fresh defect — treat non-default runs as a separate,
deliberate audit rather than folding them into a regular check cadence.

## Rules that keep results meaningful

- **Fresh session per persona, one prompt per session.** Context bleed destroys the
  knowledge boundary a persona depends on — `RUNBOOK.md`/`claude -p` invocations do this
  automatically; don't paste multiple personas or prompts into one manual session.
- **Docs-only scope.** A report citing anything outside `https://docs.pantavisor.io/<VERSION>/`
  means the fence leaked — discard the run.
- **Prefer each page's `.md` export** (`<page-url>.md`, same as the site's "Copy page"
  button) over the rendered HTML — cleaner to quote, and it's what this pack's
  `unlinked`/`broken-path` findings are verified against.
- **Never fetch `/llms.txt`** or similar site-wide AI summaries, even though they're on
  `docs.pantavisor.io` — they hand a persona the exact conclusions this pack exists to
  test whether the docs themselves get across.

## Maintaining the pack

When a run surfaces a new, real gap, verify it and add it to `ground-truth.md`. When a
persona misses a gap that's seeded in its own lane, fix the *prompt*, not the docs —
that inversion is the whole point of `ground-truth.md`. When a gap gets fixed, move it
to Resolved there rather than deleting the row.
