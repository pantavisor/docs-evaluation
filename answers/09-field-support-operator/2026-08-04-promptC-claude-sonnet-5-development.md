# Persona 09 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude (agent session, RUNBOOK.md invocation)

## Jargon audit — `getting-started/troubleshooting/` and `getting-started/operate/` only

Pages read (in crawl order, all fetched via their `.md` export, with rendered-HTML
fallback used only to check link targets the `.md` export dropped or mangled):

1. `.../troubleshooting`
2. `.../troubleshooting/faq`
3. `.../operate`
4. `.../operate/device-access`
5. `.../operate/device-access/serial-port`
6. `.../operate/device-access/local-network`
7. `.../operate/device-access/pvtx-ui`
8. `.../operate/device-access/remote-pantahub`

No conceptual pages were opened. Two extra columns (Severity, Type) are appended
to promptC.md's table beyond its own four, to keep this run comparable against
`rubric.md`'s taxonomy and to support the closing summary below.

| Term/command | First use (page URL) | Explained here? | What I'd have to go learn first | Could I run it at 2am? | Severity | Type |
|---|---|---|---|---|---|---|
| `curl -X POST ".../cgi-bin/pvtx/begin?empty=true"` | [pvtx-ui](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui) | No — quote: "For example, starting a fresh transaction: `curl -X POST \"http://<device-ip>:12368/cgi-bin/pvtx/begin?empty=true\"`" with nothing before or after it about what a "transaction" changes on the live device, whether it's reversible, or how to cancel one. | Whether "beginning a transaction" touches the running containers immediately or only stages something inert. | No — I will not fire an unexplained state-changing call at a device with a patient's equipment behind it. | S1 | `missing-concept` |
| pvtx UI **Status** field | [pvtx-ui](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui) | Partially — quote: "**Status** — `DONE` means the current revision is committed and stable. Other states (e.g. `TESTING`) appear while a revision is being evaluated." Only one good state and one in-progress state are named; no failure/error state is ever shown. | What a *broken* device's Status value looks like — the one thing I actually need at 2am. | No — I can read the field but can't tell "still testing" from "stuck/failed" from this page alone. | S1 | `missing-concept` |
| "the `pvtx` CLI" | [pvtx-ui](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui) | No — quote: "the same actions available from the `pvtx` CLI, exposed over HTTP." Named as if it's a real tool I might have, but nothing in either page tree I was allowed to read defines it, tells me if it's installed on the device or my workstation, or links to a reference for it. | Whether `pvtx` is a separate binary, part of `pvr`, or only exists as this web UI's backend. | No — there is nothing here to run; the tool is a name with no reference page in scope. | S1 | `missing-concept` |
| `pvcontrol ls` output columns: "status, group, **status goal**, restart policy" | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | No — quote: "`pvcontrol ls` # list containers: status, group, status goal, restart policy." The same unexplained column list repeats verbatim on the serial-port page. The FAQ separately says a container must "reach its health goal within the configured timeout" without ever tying that phrase back to this column or listing the values it can take. | The actual enum of status/goal values (what's normal vs. stuck) — not present on either page I was allowed to read for this task. | Partially — I can run the command, but I can't judge the output it prints. | S2 | `broken-path` |
| `pvr` (the CLI itself) | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | Not at first use — first appears in the heading "`pvr post`/`pvr clone` fails with connection refused" with no introduction, and is used in four more fixes before it's ever named. It's only defined in "What is `pvr`?" near the bottom of the FAQ's Technical Questions: "`pvr` (Pantavisor Revision) is a Git-like CLI for managing device state." | That this three-letter command is the primary tool, before I've been told that. | Yes, by copying the example verbatim — but I'm running a tool for four fixes before I know what it is. | S2 | `broken-path` |
| `lxc-ls -f` / "LXC" | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | Not at first use — quote: "`lxc-ls -f` — check running containers," no gloss. LXC itself is only explained four pages later, in the FAQ's "How does Pantavisor run Docker images?": "...converts it into an LXC container... runs under LXC, not Docker. There is no Docker daemon on the device." | That containers here are not Docker, so `docker ps`/`docker logs` habits don't transfer. | Yes, the command is given verbatim — but I'd assume `docker`-shaped behavior until I happened to reach the FAQ answer that says otherwise. | S2 | `broken-path` |
| `_config/pvr-sdk/etc/pvr-sdk/config.json` overlay (fix for "connection refused") | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | No — quote: "open it with a `_config/pvr-sdk/etc/pvr-sdk/config.json` overlay — see [Local Network]." The linked Local Network page never mentions this file or the `"listen"` key at all; the actual fix text ("setting `\"listen\": \"0.0.0.0\"`") only exists in a separate FAQ entry on the same topic that this link doesn't point to. | The exact JSON content to write — not on the page I was sent to. | No — the page I was told to check for the answer doesn't have it. | S2 | `broken-path` |
| `xconnect` | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | No — quote: "the defaults set by `pvbase.bbclass` (xconnect, pvcontrol, rngdaemon)." Reappears later as `pvcontrol graph ls # xconnect service graph` with no more context. | What xconnect actually does, so I'd know if its absence explains a symptom. | No — I don't know what a broken xconnect looks like or why I'd care. | S2 | `undefined-jargon` |
| "Build & layer pitfalls" section (`PANTAVISOR_FEATURES`, `:append` vs `+=`, `pvbase.bbclass`, `SRCREV`, `PKGV`) | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | No — self-scoped to "image builders using Yocto," quote: "never use `+=` in distro includes — it silently drops the defaults set by `pvbase.bbclass`." None of these terms are defined; the section assumes I build images, which I explicitly don't. | Yocto/BitBake basics — the thing I was told I'll never touch. | No, and I shouldn't have to — this is on my incident page regardless. | S2 | `wrong-audience` |
| Operate page "Tasks" list — 2 of 5 bullets ("Device access", "Update an application") | [operate](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate) | No — the `.md` export renders these two bullets as plain text with no link markup at all, while the other three keep their links. The rendered HTML confirms both are real links (`.../operate/device-access/` and `.../develop/application/install/`) — this is an export bug, not a missing page. | Nothing content-wise — but if I'm using the "Copy page" `.md` view this pack tells me to prefer, two of five task entry points are just unclickable text. | Partially — sidebar nav still gets me there; the preferred `.md` view doesn't. | S2 | `broken-path` |
| Device access comparison table (Method / When to use / …) | [device-access](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access) | No — the `.md` export collapses the entire table into one run-on line: "Serial console \| First boot, network not yet configured... \| Local network — SSH \| Day-to-day management... \| pvtx web UI \|..." with every row's link stripped. Rendered HTML has a normal table with working links to all four sub-pages. | Nothing — same export bug as above, worse here since it eats the table's link column entirely. | Partially — same as above, HTML fallback recovers it. | S2 | `broken-path` |
| `<revision>` placeholder in `tail /pv/logs/<revision>/...` | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | No — used raw in the very first diagnostic command with no note on how to fill it in. The pvtx UI's "Rev" field (a different, later-read page) is the only place that shows a current revision number. | Where to find "my current revision number" before I can even run the first suggested log command. | Partially — I'd have to already know or guess the value; the command as written doesn't work by itself. | S3 | `undefined-jargon` |
| `pvexport` package | [pvtx-ui](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/pvtx-ui) | No — quote: "apply from a `pvexport` / `.tar.gz` package (see API documentation below for the underlying calls)." Never says what's inside one or how to produce one outside of downloading it from Pantahub. | What a pvexport package actually contains, if I ever needed to build one by hand. | No — nothing here to run; it's a file format named but not shown. | S3 | `no-example` |
| "auto-recovery" retry count | [troubleshooting](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting) | No — quote: "auto-recovery retries it; after the retries are exhausted Pantavisor rolls back to the last `DONE` revision." No number of retries or time budget is given. | How long to wait before I decide the auto-recovery isn't going to save me and I should intervene manually. | Yes — nothing to run here, but I'm guessing how long "eventually" is on the phone with a customer. | S3 | `undefined-jargon` |

## What worked

The **serial console** page (`.../operate/device-access/serial-port`) and the
**remote Pantahub** page (`.../operate/device-access/remote-pantahub`) are the
strongest pages in scope: every command is concrete, copy-pasteable, and mostly
tied to a one-line explanation of what it does (`lxc-ls -f` → "Shows all containers
and their LXC state (RUNNING, STOPPED, etc.)"; `pventer -c sensor-app` → "Drops you
into the container's filesystem, process, and network namespaces"). The
**local-network** page's SSH section is also a good example of a destructive
gotcha handled *with* a warning: it explicitly says writing `~/.ssh/authorized_keys`
directly doesn't persist and tells you the supported alternative instead of letting
you find that out the hard way.

## Closing summary

- **Task outcome:** completed.
- **Worst finding:** S1 — the pvtx web UI's REST API section hands over
  `curl -X POST ".../cgi-bin/pvtx/begin?empty=true"` to "start a fresh transaction"
  on a live device with no explanation of what that does, whether it's reversible,
  or how to back out of it — exactly the kind of command that gets run at 2am with
  no idea what just happened.
- **What worked:** `.../operate/device-access/serial-port` — commands paired with
  plain explanations of their output, the one page in scope that consistently
  passes the "could I run it at 2am" test.
- **Confidence:** stayed in character. Every "explained here?" and "could I run it"
  judgment above is based only on what is or isn't present in the eight pages
  fetched for this run — I did not draw on outside knowledge of LXC, Yocto, or
  Pantavisor to fill gaps the docs themselves left open. One assist: `answers/index.md`
  (which this run's process requires reading before appending a row) surfaced that
  persona 09 / prompt A already found the `pvcontrol ls` status/goal values defined
  on an unlinked conceptual page elsewhere on this same site version — I used that
  only to correct my own severity call on that one row from S1 to S2 (info exists
  on-site, just not on a page this prompt's scope allowed me to read), not to add or
  remove any finding.
