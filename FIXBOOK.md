# FIXBOOK — turning one finding into a docs PR

This file closes the loop `RUNBOOK.md` opens. `RUNBOOK.md` reads the live docs as a
persona and produces findings; `FIXBOOK.md` takes exactly **one** of those findings and
turns it into a draft pull request against the repo that actually owns the affected
page's source. Like `RUNBOOK.md`, it's runner-agnostic — a human pasting this into a
session, or `claude -p "Follow FIXBOOK.md ..."` — and it never merges anything itself.

## Where the docs actually live (read this before touching anything)

`docs.pantavisor.io` is built by the `docs.pantavisor` repo (Docusaurus). Its
`reference/` and `reference_versioned_docs/` folders are **entirely generated at build
time** (`scripts/sync-reference.mjs` downloads a per-release docs tarball and
`migrate-docs.js` converts it into those folders) — nothing under them is committed
source, and none of it survives the next build. **Never open a PR against
`docs.pantavisor`'s `reference/` content** — it accomplishes nothing.

The real source of every `/development/<X>/...` page lives in repo `<X>`'s own `docs/`
folder:

| URL path segment (right after the version) | Repo | Docs live under |
|---|---|---|
| `meta-pantavisor/...` | `github.com/pantavisor/meta-pantavisor` | `docs/` |
| `pantavisor/...` | `github.com/pantavisor/pantavisor` | `docs/` |
| `pvr/...` | `gitlab.com/pantacor/pvr` | `docs/` |

The mapping from URL to file is near-literal: strip `/<VERSION>/<repo-segment>/` from
the finding's Location URL, and what's left is the path under that repo's `docs/`,
minus a `.md` extension — e.g.
`https://docs.pantavisor.io/development/meta-pantavisor/overview/images` →
`docs/overview/images.md` in the `meta-pantavisor` repo. A URL ending in a directory
(no trailing segment, e.g. `.../overview/`) maps to that directory's `index.md`
instead. This isn't guaranteed exact — `migrate-docs.js` does rename `_index.md` →
`index.md` and remaps a couple of legacy links — so step 3 below always confirms the
file exists and its content matches before editing it.

**This routing table can go stale.** The live site's structure has changed before (see
`ground-truth.md`'s provenance notes and `RUNBOOK.md`'s 404-handling step) — if a
finding's URL segment doesn't match any row above, or the repo you'd expect a page in
doesn't have a matching file, stop and say so rather than guessing a new mapping.

## Invocation contract

> Follow `FIXBOOK.md` in the docs-eval repo. Fix this finding: `<paste one finding row,
> or a path + row reference into an answers/ report, e.g.
> answers/01-yocto-no-containers/2026-07-30-promptA-claude-sonnet-5-development.md,
> the "trail" / overview/images row>`
>
> Repo checkouts: `<local paths to meta-pantavisor/, pantavisor/, pvr/ if you have them
> — otherwise this session will need to clone them>`

**Exactly one finding per invocation** — the same discipline as `RUNBOOK.md`'s "one
prompt per session." Fixing several findings in one pass risks steamrolling multiple
pages in a single PR, which is much harder for a human to review and is exactly the
scope creep a one-finding-per-run granularity exists to prevent.

## Steps

1. **Parse the finding.** You need its Location (URL), Task step, What blocked me,
   Evidence, Severity, Type, and Suggested fix. If given a report file + row reference
   instead of a pasted row, open that file and read the exact row — don't paraphrase
   from memory or from a different row in the same table.

2. **Route to a repo.** Take the URL path segment immediately after the version
   (`/development/`, or whatever `<VERSION>` the report's header names) and match it
   against the table above. If the finding's Location is off-domain, or its Type is
   `outside-docs` pointing somewhere that isn't one of these three repos' `docs/`
   folders, stop — this class of finding isn't a same-repo content fix, and deciding
   where the content *should* live is a bigger judgment call than this runbook makes.
   Flag it for human triage instead.

3. **Locate the real source file** in that repo's local checkout (or a fresh clone if
   none was given), following the mapping above. Confirm you've found the right file by
   checking its rendered content against the finding's Evidence quote — if the quote
   isn't in the file, the docs have moved since the report was filed; find where the
   content actually is now, or stop and say you couldn't.

4. **Re-verify the finding still applies.** Docs change between when a report is filed
   and when it's actioned. If the gap described is already fixed, say so and stop —
   don't open a redundant PR.

5. **Make the minimal edit** the Suggested fix describes. This is a gap report, not a
   rewrite brief: touch only what's needed to close this one finding, match the
   surrounding file's style and voice, and don't expand scope to nearby issues you
   notice along the way — even genuine ones. Note those separately for a future finding
   instead of folding them in here.

6. **Open a branch, commit, push, and open a draft PR** in the *target repo* (not
   docs-eval):
   - Branch name: something short and descriptive, e.g. `docs-eval/define-trail-glossary-link`.
   - Commit message: describe the change; reference the persona/prompt/date this
     finding came from.
   - PR description must include: a link back to (or quote of) the originating
     `answers/` report row, the original "What blocked me" and "Evidence," what
     changed and why in a sentence or two, and an explicit note that this is a
     **draft** for human review — never assert the fix is final or verified.
   - Mark the PR as draft if the platform supports it.

7. **Do not merge.** Report the PR URL back and stop. Whether and when it merges is a
   maintainer's call, not this runbook's.

## What this is not

It doesn't batch-fix a whole report — that's the point of the one-finding-per-run rule.
It doesn't touch `ground-truth.md` (that's a separate, human maintenance step — see
`README.md`'s "Maintaining the pack"). It doesn't invent scope beyond the one Suggested
fix sentence. And it never opens a PR against `docs.pantavisor`'s generated `reference/`
folders — see "Where the docs actually live" above.
