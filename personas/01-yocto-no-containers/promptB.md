# Persona 01 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> Your team builds a Yocto image for an i.MX8 board. There's one application on it that
> changes weekly, and shipping a full image every week is killing you.
>
> **Task: work out, from the docs alone, what would actually change in your build and
> your release process if you adopted this — and what the weekly update would then look
> like end to end.**
>
> Be specific, as you'd have to be with your team lead:
> 1. What happens to your existing recipes and layers? Do they survive?
> 2. What does the weekly app update consist of — what artifact, built how, by what
>    command?
> 3. Your app currently ships as a rootfs directory installed by a recipe. The docs say
>    to add it "from a Docker image." **What is a Docker image, in terms you know, and
>    what has to be true of your app for that to work?** Answer only from the docs.
> 4. What does this cost you — flash, RAM, boot time, build time?
>
> Where the docs don't answer, don't fill the gap from general knowledge. Record it and
> state plainly what you'd have had to guess. Report against `../../rubric.md`.
