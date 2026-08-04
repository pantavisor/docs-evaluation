# Persona 02 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> Concrete situation. Your current product: Buildroot, custom external tree, ~40 MB
> squashfs rootfs, an i.MX6, updates today by shipping a full image over USB stick to a
> technician. The pain: every field update is a truck roll and a full reflash.
>
> **Task: from the docs alone, produce the honest answer to four questions.**
>
> 1. **Can I use this at all without adopting Yocto?** Not "is Yocto good" — can I get a
>    device running this with a Buildroot rootfs, yes or no? Find where the docs say so.
>    If they never say either way, that's your finding.
> 2. **What is the smallest thing I'd have to change** about how I build today?
> 3. **What does it cost me in flash?** I'm at 40 MB and I care. Find real numbers.
> 4. **What's the update story** — what actually goes over the wire on a field update,
>    and how big is it? Don't stop at meta-pantavisor's side — the wire mechanism is
>    the runtime's, so follow it into `pantavisor/` if that's where the real answer
>    lives.
>
> For each: answer, cite it, or record its absence. Do not soften an absence into "the
> docs imply..." — either a page says it or it doesn't.
>
> Report against `../../rubric.md`.
