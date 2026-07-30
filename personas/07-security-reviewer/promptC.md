# Persona 07 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/security` end to end, then follow every
> link out of it that bears on trust — likely into
> `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/security/trust-model`,
> `.../security/atomicity-and-trust`, and `https://docs.pantavisor.io/{VERSION}/pantavisor/overview/storage`.
>
> Produce a table of **every term used before it is defined**, judged strictly from your
> background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
>
> Your background is deep in crypto, so the gaps here won't be crypto words — they'll be
> product words load-bearing for security claims. Hunt for:
>
> - **`pvs@2`**, **`_sigs`**, **`x5c`**, **`__system__`** — is the scheme ever actually
>   specified, or only named?
> - **`revision`**, **`object`**, **`trail`**, **`step`** — you cannot assess integrity of
>   a thing you can't define. Are they defined anywhere you could reach from a security
>   page?
> - **`claim`**, **`challenge`**, **device identity** — these are authentication
>   primitives being used as product nouns. Is the mechanism ever specified?
>
> Add one column for this persona only: **"Security-load-bearing?"** — yes if a security
> claim depends on the term. Any row that's both undefined *and* security-load-bearing is
> automatically S1, regardless of how the rest of the run went.
>
> Report against `../../rubric.md`.
