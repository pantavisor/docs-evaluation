# Persona 04 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> Concrete: your app is a container image. You have a Raspberry Pi 4 and an SD card. You
> want the app on the Pi today, and you want to ship a bugfix to it next week from your
> desk.
>
> **Task: from the docs alone, write the exact sequence of commands you'd run, start to
> finish.** A runnable script, or as close as the docs let you get.
>
> Then answer four questions honestly:
>
> 1. **Could you actually run this?** Not "does a page exist" — could *you*, with your
>    knowledge, type these commands and have them work? Where's the first one you don't
>    understand?
> 2. **Did you at any point have to build an operating system?** If yes: at which step,
>    and did the docs warn you before you started? Being ambushed into a 50 GB build at
>    step 9 is a much worse finding than being told at step 1.
> 3. **Where does your app image go?** You have an image on Docker Hub. Trace, from the
>    docs, what happens to it between your registry and the device. Do you understand the
>    mechanism? Does it survive intact? That journey crosses from the build-tooling
>    docs into the runtime's own docs (`pantavisor/`) at some point — say where, or
>    say it never does.
> 4. **The bugfix.** Next week's update — what's the command, and what does the device do
>    while it happens? Does your service go down? Does the whole device reboot?
>
> Where the docs don't answer, record the absence rather than guessing. Report against
> `../../rubric.md`.
