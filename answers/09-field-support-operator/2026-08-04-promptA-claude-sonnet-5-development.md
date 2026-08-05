# Persona 09 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Narration

It's 2am. I land on `.../meta-pantavisor/getting-started/start` — the first search
result — and scan it, not read it. The page is a "first time setting this up" page
(paths to flash an image, install `pvr`), which isn't my situation — I have a device,
it's down. Nothing in the article body is labeled "troubleshooting." The only thing that
matches my situation ("serial console cable, physical access") is the **Serial Console**
link under Prerequisites, so I click that.

`.../operate/device-access/serial-port` is good: connect the USB-to-TTY adapter,
`minicom -b 115200 -D /dev/ttyUSB0`, watch for the Pantavisor banner, press Enter for
the debug shell. The banner itself lists `lxc-ls` and `pventer -c <CONTAINER>` as
"Useful commands" — that's what stops me from typing `journalctl` or
`systemctl status` before I even try them, which is exactly the trap I'd have walked
into. Good catch by the docs.

From here I have real commands: `lxc-ls -f` (container list + LXC state), `pventer -c
<container>`, and a `pvcontrol` family — `pvcontrol ls` ("status, group, status goal,
restart policy"), `pvcontrol daemons ls`, `pvcontrol graph ls`, `pvcontrol steps
show-progress current`, `pvcontrol buildinfo` — plus `tail -f /pv/logs/<revision>/...`
for raw logs. I run `pvcontrol ls` first, out of habit — I want a status board before I
start reading raw text. **This is the first command whose output I don't fully
understand**: the column is literally called `status` and `status goal`, and nothing on
this page tells me what values those take or which ones mean "this is broken." I fall
back to what I actually know how to do — read the raw logs — instead of trusting the
status column.

This page has no further inline content link forward, so I go back to the rendered
`start` page and check its nav for anything titled "Troubleshooting" — there is one, and
it's not in the `.md` export I read first (see finding below). Clicking it lands me on
`.../getting-started/troubleshooting`, which is much more what I wanted: a "Quick
Diagnostics" block with the exact same commands, a "Common Issues" list (boot-loops,
stuck OTA, failed Pantahub claim, `pvr clone`/`pvr post` connection-refused), and a link
to an FAQ. The FAQ's "Connectivity Issues" section has a "My device is not showing up on
the network" entry that's a near-exact match for my customer's complaint: check the
cable/Wi-Fi, `ip addr show eth0` from serial, `pvr device scan` from my workstation,
`tail` the Pantavisor log for DHCP/network-container errors. That's a real runbook I can
hand the technician line by line.

**Do I fix it, or wake someone up?** I fix it. Between the Quick Diagnostics commands
and the FAQ's network-not-showing-up steps, I have a concrete path: check the interface,
check the log, check which container(s) are down via `lxc-ls -f`, read their console
logs for the actual error. I never had to leave `docs.pantavisor.io` to do it. The one
place I got stuck — interpreting the `status`/`status goal` values themselves — I worked
around with raw log-reading rather than being blocked outright, which is exactly the
skill the persona card says I actually have.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/serial-port` | A, debug-shell step, running `pvcontrol ls` | I run `pvcontrol ls` to get a status board before reading logs, but nothing on this page, the troubleshooting page, or the pvr CLI reference tells me what the `status` or `status goal` column values mean or which ones mean "broken" — I have to fall back to raw log-reading instead of trusting the command meant to summarize state. | `pvcontrol ls  # list containers: status, group, status goal, restart policy` — no legend anywhere in this path. (The actual enum — `INSTALLED`/`MOUNTED`/`BLOCKED`/`STARTING`/`STARTED`/`READY`/`RECOVERING`/`STOPPING`/`STOPPED` and the status-goal options — only exists on `/development/pantavisor/overview/containers`, a conceptual page not linked from the serial-console, troubleshooting, or pvr-CLI-reference pages I actually used.) | S2 | `broken-path` | Link the `status`/`status goal`/`restart policy` columns in `pvcontrol ls` (serial-port.md and troubleshooting.md) to the Containers overview page that defines those values. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start` | A, initial scan of the landing page | The page's own "Next steps" — the two lines that should have pointed me toward install/operate docs — read as plain unlinked text in the `.md` export I was told to prefer, so scanning the copy-paste content alone gives no way to click through; I only found the real destinations (and the separate "Troubleshooting" nav entry) by falling back to the rendered page. | `.md` export: "- Install your first application with `pvr` .\n- Access your device over serial, the local network, or Pantahub." — no markdown link syntax on either line, though the rendered HTML has real hrefs to `.../develop/application/install/` and `.../operate/device-access/`. | S4 | `broken-path` | Fix the `.md`/copy-page export so links at the end of list items keep their markdown syntax. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/troubleshooting` | A, reading the "Build & layer pitfalls" closer / looking for the FAQ | The page's pointer to the FAQ is unreadable as a link in the `.md` export — the words are even run together — so a reader relying on the export alone wouldn't know a dedicated FAQ page exists at all (I only found it because a different, earlier link on the same page happened to point into `troubleshooting/faq`). | "See theFAQ for more specific questions and answers." — no link, no space, in the `.md` export. | S4 | `broken-path` | Fix the `.md` export so this trailing sentence keeps its link and spacing. |

## Closing summary

- **Task outcome**: completed with detours.
- **Worst finding**: no S1 this run — the worst is S2: `pvcontrol ls`'s `status`/`status
  goal` columns are never defined anywhere in the serial-console → troubleshooting →
  pvr-CLI-reference path a field operator actually follows, even though the definitions
  exist on the site (an unlinked conceptual page under `/pantavisor/overview/`).
- **What worked**: `.../getting-started/troubleshooting` — its "Quick Diagnostics" block
  and the FAQ's "My device is not showing up on the network" entry gave a concrete,
  in-order runbook that matched the customer's actual complaint and let me stay on
  `docs.pantavisor.io` the whole time.
- **Confidence**: mostly stayed in character. One deliberate break: after hitting the
  undefined `status`/`status goal` columns, I checked `/development/pantavisor/overview/
  containers` — a conceptual page the persona would never read at 2am — purely to
  determine whether the definition exists anywhere on the site at all (required to grade
  S1 vs. S2 correctly). That page's content was never used to actually complete the
  in-character task; the narrated fix relies only on logs and the troubleshooting/FAQ
  pages the persona would really read.
