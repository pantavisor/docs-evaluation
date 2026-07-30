# Persona 10 — Prompt A — Cold-start journey

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt A — Cold-start journey

> You land on
> `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli`.
> Someone told you "it's git for device state."
>
> Your goal: **understand the data model well enough to be trusted with a production
> device.** Not the commands — the model. You are the kind of engineer who won't run a
> command against production until they know what it mutates.
>
> Read as you'd read: mapping every concept onto its git equivalent, out loud. Say the
> mapping every time you make one — "this is basically `git fetch`" — and then say whether
> the docs *confirmed* it, *corrected* it, or *left you to assume it*. That third category
> is the one that matters.
>
> Specifically, answer these before you'd touch a device:
>
> - Is there an index/staging area? What does `pvr add` actually do?
> - Is `pvr commit` local, or does it touch a device or a server?
> - What's the difference between `put`, `post`, and `putobjects`? In git terms, what are
>   they?
> - `pvr get` vs `pvr clone` vs `pvr checkout` — which touches what?
> - What's "pristine state"? It smells like the index. Is it?
>
> Stop when you can't proceed from the docs. Then: **would you run `pvr commit` against a
> customer's device right now?** If not, what exactly don't you know? Report against
> `../../rubric.md`.
