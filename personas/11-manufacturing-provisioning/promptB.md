# Persona 11 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> **Task: from the docs alone, write the manufacturing work instruction** for a single
> unit, from bare board to shippable. Every step, with a time estimate.
>
> Then answer the questions that decide whether you can take the job:
>
> 1. **What makes each device unique?** Serial, MAC, key, certificate — what's per-unit,
>    when is it generated, by whom, and where is it stored? If two units left your line
>    identical, would anything catch it?
> 2. **Does the station need internet?** Yes or no, and cite the page. A line that needs
>    an outbound internet connection to a vendor's cloud is a completely different quote
>    and a completely different security review. **If the docs don't clearly answer this,
>    that's S1** — you cannot quote the job.
> 3. **Claiming.** You'll encounter the idea that a device gets "claimed" by its owner.
>    Trace it end to end: **what does the line have to produce for that to be possible
>    later?** Is there a per-unit artifact — a code, a challenge, a token — that has to be
>    captured, printed on a label, or handed to the customer? Follow this to a runnable
>    command. If the trail ends at a description with no command, say precisely where.
> 4. **Time per unit.** Add it up. Where's the long pole? Is there a per-unit step that
>    can't be parallelised?
> 5. **Failure and rework.** A unit fails at test. Can you reflash and re-provision it, or
>    is something now permanently burned? Can you reuse an identity, or is that a
>    duplicate-identity incident waiting to happen?
> 6. **Test.** How does the station verify a unit is good before it ships? Is there a
>    documented check?
>
> Report against `../../rubric.md`.
