# Persona 10 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> Four tasks. Do them in order — the order is the point.
>
> **Task 1 — onboard a device.** You have a brand-new device. It's on your network. You
> want it in your account so you can manage it. **Work out, from the docs alone, the
> complete sequence from unboxed device to a device you can push to.** Follow every lead.
> If a page tells you to obtain something, find the command that consumes it. Do not stop
> at "a page mentions this exists" — trace it to a runnable command. If the trail ends,
> **say exactly where it ended and what you were left holding.**
>
> **Task 2 — add a container.** Add nginx to that device. Find the command. Then, before
> running it, read its flags — properly, the way you would before touching production.
>
> **Task 3 — the flags.** Read the flags the add-container command actually documents.
> If `--group` and `--status-goal` are among them with a list of permitted values,
> continue: what does each value *mean*? Pick one for a web server and justify it. Where
> is that meaning documented — did anything link you there from the flag, or did you have
> to go hunting? **Could you have chosen correctly without leaving the `cli-tools/`
> pages?** If either flag isn't mentioned on this page at all, say so plainly instead of
> forcing an answer — a flag control mechanism that governs container startup order and
> isn't documented at the command that sets it is, itself, the finding. Also check the
> runtime's own reference docs (`pantavisor/`) for the state-format definition of these
> fields — not just the `cli-tools/` pages.
>
> **Task 4 — the CI job.** A colleague's script uses a flag your docs don't mention. You
> search for it and find nothing. **What do you do?** How long before you conclude the
> docs are incomplete rather than that you can't search? Would you conclude the flag was
> removed and delete it from the script?
>
> Report against `../../rubric.md`.
