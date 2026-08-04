# Run index

One row per run, newest first. Populated by `RUNBOOK.md` — see that file for the
process. Manual runs not filed through `RUNBOOK.md` aren't tracked here unless someone
adds them by hand.

| Date | Persona | Prompt | Version | Model | Outcome | Worst finding | Report |
|---|---|---|---|---|---|---|---|
| 2026-08-04 | 03 — Cloud-native developer, no embedded | A | development | claude-sonnet-5 | completed with detours | S1: nothing in this path explains how `pvr app add`/`update` authenticates against a private registry, forcing a guess that it reuses local Docker credentials | [link](03-cloud-native-no-embedded/2026-08-04-promptA-claude-sonnet-5-development.md) |
| 2026-07-30 | 01 — Yocto integrator, no containers | A | development | claude-sonnet-5 | blocked at step 3 | S1: "trail" is the load-bearing concept for the whole layer and is never defined anywhere reachable from the entry point | [link](01-yocto-no-containers/2026-07-30-promptA-claude-sonnet-5-development.md) |
