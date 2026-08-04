# Persona 06 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> **Task: from the docs alone, produce the porting checklist for your board.** Every file
> you'd create or edit, every variable you'd set, in order.
>
> Then answer these, which are the questions that actually consume the days:
>
> 1. **What does this need from my bootloader?** I have working U-Boot. What must it do
>    that it doesn't do now? Which variables, which commands, what boot sequence? Be
>    exact — quote the docs or record the absence.
> 2. **What does it need from my kernel?** Config options, drivers, initramfs
>    requirements. Anything that would make me rebuild my defconfig?
> 3. **My storage is NAND with UBIFS**, not eMMC. Is that supported? Where do the docs
>    say so? (If the answer is buried in a page about something else, that's a
>    `broken-path` finding.)
> 4. **When it doesn't work — and it won't, the first six times — how do I debug it?**
>    Serial console, yes, but what am I looking *at*? What does a healthy boot look like,
>    so I can spot an unhealthy one? Is there a porting troubleshooting page at all?
>    Check whether that healthy-boot picture is documented on the runtime's own pages
>    (`pantavisor/`) as well as meta-pantavisor's BSP docs — a description that only
>    exists on one side is itself a finding.
> 5. **Where's the boundary?** At what point does my job end and the container people's
>    job start? Do the docs ever draw that line, or do they assume one person does both?
>
> Report against `../../rubric.md`.
