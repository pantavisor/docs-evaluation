# Persona 09 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> **Task: from the docs alone, write the runbook you wish you had.** Five scenarios. For
> each: the commands, in order, that you'd actually run.
>
> 1. **Device is up but the app isn't running.** How do you see what's running? What's the
>    equivalent of `docker ps` / `systemctl status`? How do you see its logs? How do you
>    restart just the app? Check whether the runtime's own docs (`pantavisor/`) cover
>    this as directly as meta-pantavisor's operate/ pages do — you'd take whichever
>    answers faster at 2am.
> 2. **Device is unreachable over the network.** You have serial. What do you type to find
>    out what's going on? What does a healthy boot look like so you can spot a bad one?
> 3. **Last update broke it.** How do you find out what version it's on, what it was on
>    before, and how do you go back? Can the technician do it locally, without the cloud?
> 4. **Disk is full.** What fills up, what's safe to delete, what breaks if you delete the
>    wrong thing?
> 5. **You need to know what's actually on this device** to tell the engineer you're about
>    to wake up. What do you run to get a complete picture to paste into Slack?
>
> For each scenario, three honest judgements:
>
> - **Did you find it?** Which page — or is it absent?
> - **Could you run it at 2am** without reading a concept page first? A command you have
>   to go learn a model to understand is not usable in an incident.
> - **What did you have to guess?** Guessing on a production device at 2am is how devices
>   get bricked. Every guess here is a real risk, so record them precisely.
>
> Report against `../../rubric.md`.
