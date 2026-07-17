# Rubric and output contract

Every persona run reports against this rubric. Do not restate it in persona files —
link here. Keeping one copy is what makes runs comparable across personas and over
time.

## Severity

| Level | Name | Test |
|---|---|---|
| **S1** | Blocker | I could not complete the task from the docs alone. I had to guess, leave the docs, or stop. |
| **S2** | Detour | I completed it, but only by leaving the intended path — another repo's docs, a README, an external site. |
| **S3** | Friction | I completed it, but a term went undefined or a rationale was missing and I proceeded on assumption. |
| **S4** | Polish | Inconsistency, stale content, dead index entry, missing frontmatter. Didn't block me. |

The line that matters most is S1 vs S2. If you *found* the answer somewhere in the three
`docs/` trees, it's S2 at worst — the information exists, the path to it doesn't. If the
answer is not in the trees at all, it's S1. Be strict about this: S1 means the
documentation does not contain the answer, not that the answer was hard to find.

## Gap taxonomy

Exactly one tag per finding. If two fit, pick the one that describes the *fix*.

| Tag | Means |
|---|---|
| `missing-concept` | The docs never explain a mechanic or model they depend on. |
| `undefined-jargon` | A term is used as though known. Distinct from `missing-concept`: this is a word, that is an idea. |
| `broken-path` | The information exists but nothing leads from where I was to where it is. |
| `no-example` | Documented in the abstract, with no runnable or concrete instance. |
| `wrong-audience` | The page assumes a background its readers, arriving by the documented route, won't have. |
| `unlinked` | A page exists but nothing links to it, or an index omits its own children. |
| `stale` | Contradicts reality, or describes something that has moved on. |
| `outside-docs` | The answer lives somewhere a docs-site reader can't reach (README, source, external site). |

## Report format

One table, most severe first. One row per finding.

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `pvr/docs/commands/app.md:88` | B, step 2 | I have to pick a `--status-goal` but nothing tells me what the values mean. | "`--status-goal` — one of `MOUNTED`, `STARTED`, `READY`" | S1 | `missing-concept` | Link the enum to wherever these are defined, or define them here. |

Field rules:

- **Location** — `repo/docs/path/page.md:line`. Always a line. A finding without a
  citation is not a finding.
- **Task step** — which prompt and which step. This is what makes a report actionable
  rather than a list of opinions.
- **What blocked me** — one sentence, first person, in character. "I have to pick a
  value and nothing tells me what they mean," not "the enum is undocumented."
- **Evidence** — short verbatim quote. If the finding is that something is *absent*,
  quote the place a reader would reasonably expect it and say what isn't there.
- **Suggested fix** — **one sentence, hard limit.** This is a gap report, not a rewrite.
  The person fixing it knows the docs better than you do; your job is to say precisely
  where the reader fell through, not to draft the patch.

## Two rules that keep reports honest

**Report only what blocked you as this persona.** Not what you think is generally
suboptimal about the documentation. If you didn't hit it while doing the task, it isn't
yours to report. A persona report is evidence of a specific reader failing at a specific
step — that's its whole value, and general commentary dilutes it.

**Absence of a finding is a finding.** If you completed a task cleanly, say so and say
which pages carried you. Knowing what works is how we avoid rewriting pages that are
already doing their job.

## Closing summary

End every run with four lines:

- **Task outcome** — completed / completed with detours / blocked at step N.
- **Worst finding** — the one S1 that most deserves attention, in a sentence.
- **What worked** — the page that most helped, with a path.
- **Confidence** — did you find yourself using knowledge you were told you don't have?
  Say so honestly. A run that broke character is still useful, but the reader must know.
