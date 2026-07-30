# Persona 12 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Structural audit

> This replaces the jargon audit. You can see the graph; use it.
>
> Across the whole site — `https://docs.pantavisor.io/{VERSION}/pantavisor/` and
> `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/` (which includes the PVR CLI
> reference, nested under `getting-started/develop/cli-tools/` rather than living as its
> own section):
>
> 1. **Orphans.** Which pages does *nothing* link to? For each, judge: does its content
>    matter? An orphaned test plan is S4; an orphaned page defining the corpus's core
>    vocabulary is S1.
> 2. **Incomplete indexes.** Which index or landing pages omit files that sit beside them?
>    Name the omitted pages and judge what's lost.
> 3. **The link graph between sections.** Count outbound links from `pantavisor/` to
>    `meta-pantavisor/` and back, and separately from the rest of `meta-pantavisor/` into
>    its own nested `cli-tools/` pages. Report the counts; **if any is zero or near it,
>    say what a reader standing in that section cannot reach.**
> 4. **Terms defined in one section, used bare in another.** Find them. These are
>    unresolvable by anyone — human or agent — reading only the section they landed in.
> 5. **Contradictions and staleness.** Anywhere two pages disagree, or a page describes
>    something that isn't so. Include meta-pages that have drifted from reality.
> 6. **Machine-readability.** Consistent frontmatter? Titles? Would this corpus chunk and
>    embed cleanly, or would a retrieval system return fragments that can't answer
>    anything because the definition lives in a different section with no link?
>
> Close with one sentence: **if an agent is the primary reader of these docs in two years,
> what is the single structural change that would most improve the answers it gives?**
>
> Report against `../../rubric.md`.
