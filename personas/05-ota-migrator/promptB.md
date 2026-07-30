# Persona 05 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> **Task: from the docs alone, answer the six questions you'd have to answer before
> putting this in front of your CTO.** Cite a page for each, or record the absence.
>
> 1. **The atomic instant.** During an update, there is one moment where the device is
>    committed to the new version. Where is it? What's the mechanism — a bootloader flag,
>    a symlink, a partition flip, a file rename? Pin it down.
> 2. **Power-fail.** Cut the power at each stage: mid-download, mid-write, after write
>    before reboot, mid-reboot, after boot before health-check. What's on the device in
>    each case? Do the docs tell you, or do they just say "atomic"?
> 3. **Rollback trigger.** What decides an update failed? Boot counter, watchdog, health
>    check, cloud signal? Who decides, the device or the server? How many boot attempts
>    before it gives up?
> 4. **What's in the update.** Mender ships a full rootfs artifact. What ships here, and
>    how big is it for (a) an app-only change, (b) a kernel change? Find real numbers or
>    record their absence.
> 5. **Signing.** What's signed, by whom, verified when, and against what trust root? Can
>    a device be made to install something you didn't sign?
> 6. **Staged rollout.** Can you do 1% → 10% → 100% with an abort? If the docs don't say,
>    that's a finding, and for your fleet it may be a blocking one.
>
> Be rigorous about S1 vs S2 (see `../../rubric.md`): if the answer exists somewhere in the
> three trees but the migrate page never leads you there, that's a `broken-path` detour,
> not a missing answer. Both matter; they get fixed differently.
>
> Report against `../../rubric.md`.
