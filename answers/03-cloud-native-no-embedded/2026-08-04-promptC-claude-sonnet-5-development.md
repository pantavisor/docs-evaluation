# Persona 03 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: sub-agent

## Task

Read `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start` and
`https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop`, following
links a developer would plausibly follow, and produce a jargon audit: every term used
before it is defined, judged strictly from a cloud-native/Kubernetes background with zero
embedded experience.

Path actually followed (in-character, as a developer with hardware in hand would click):
`start` → `start/download-and-flash` → `develop/cli-tools/pvr-cli` (the "Install `pvr`"
link from `start`) → `develop/application/configure` → `develop/application/view` →
`develop/application/access-applications` → `develop/application/remove` (the four
"Manage applications" links on `develop`) → `operate/device-access/serial-port` (linked
from `download-and-flash` for wiring the serial adapter mentioned in its prerequisites).
I did not follow the "no hardware? run in Docker (AppEngine)" alternative on `start`,
since this persona has a physical Pi in hand — noted below where that matters.

## Jargon audit

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? | Evidence | Severity | Type |
|---|---|---|---|---|---|---|---|
| `trail` | [`.../getting-started/develop/application/remove`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/remove) | No | Some kind of history/log of applied revisions, guessed from context alone — never told what it actually is, how deep it goes, or how it relates to a "revision" | Can't tell — never defined, so I don't actually know if my guess is right | "Pantavisor stops the container and removes it from the trail on the next boot." ... "The previous revision (with the container) is kept in the trail: to restore it, roll back by running that revision" | S1 | `missing-concept` |
| `status goal` | [`.../getting-started/develop/application/configure`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/configure) | No | Some enum of container lifecycle states (like a K8s pod phase: Pending/Running/Ready) that a container must reach for the deploy to be considered successful | Unverifiable — the term recurs three more times (`view`, `access-applications`, `serial-port` pages) and is never once given its possible values | "if the new revision starts cleanly and all containers reach their status goal, it is committed as the new permanent state" | S1 | `missing-concept` |
| `base` / `certified base` | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) | No | The BSP + Pantavisor runtime layer, cryptographically signed, that apps sit on top of and can't modify — reasonable guess by analogy to a signed OS image, but "certified" implies some verification process/authority that's never named | Partially — the directory listing on `configure` (`bsp/`, `network/`) hints at what's *in* it, but nothing ever explains what "certified" means mechanically (signed by whom, checked how, what breaks if it doesn't verify) | "Apps are versioned and updated independently of the base. A CVE fix in your app ships as a small container layer — not a full-image flash — and never touches the certified base." | S1 | `missing-concept` |
| `container` (Pantavisor sense) | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("Install apps — add containers with the `pvr` CLI...") | Yes, but only after two more pages and pieced together myself | A live OCI/Docker container: `pvr app add --from nginx:stable-alpine` pulls an image and runs it the way `docker run` would, same layered image, same runtime | No — it's an LXC namespace running from a **pre-baked, read-only SquashFS image** assembled once at commit time by `pvr app add`/`pvr app update`; there's no running daemon pulling or reconciling it the way Docker/containerd does | "Pantavisor containers are isolated LXC namespaces." (`access-applications`) + "Each container's root filesystem is a read-only SquashFS image (`root.squashfs`)." (`configure`) | S1 | `missing-concept` |
| PVR CLI Reference → `docs.pantahub.com` | [`.../getting-started/develop/cli-tools/pvr-cli`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli) | N/A — scope-fence note, not a term | N/A | N/A | "**[PVR CLI Reference](https://docs.pantahub.com/pvr/)** - Legacy Pantahub documentation" — a different domain than `docs.pantavisor.io`, labeled "Legacy," presented as the place for the *complete* command reference | S2 | `outside-docs` |
| `service mesh` (pv-xconnect) | [`.../getting-started/develop/application/access-applications`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/access-applications) | Yes, in the same paragraph | An Istio/Linkerd-style thing: sidecar proxies intercepting and routing L7 traffic between services over the network | No — it's socket/device-node injection directly into a namespace (closer to a bind-mount than a proxy); no traffic interception, no L7 routing, no network hop at all | "use the **pv-xconnect** service mesh. It injects sockets or device nodes directly into a consumer container's namespace." | S3 | `undefined-jargon` |
| `state` (device state) | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) ("Install `pvr` — the client used to inspect and change device state") | Partially — shown by example (clone → edit → commit → `pvr post`), never contrasted explicitly | A continuously-reconciled desired state, like a Kubernetes controller watching a manifest and correcting drift | No, as far as I can tell — every workflow I read is an explicit, one-shot `pvr post` push; nothing runs on a loop watching for drift, but no page ever says this outright, I only inferred it from the CLI examples always ending in a manual `pvr post` | "the client used to inspect and change device state" (`start`); every example workflow ends with an explicit `pvr post http://<device-ip>:12368` | S3 | `missing-concept` |
| `deploy` | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("Remove — drop a container from device state and deploy") | Yes, but the word is overloaded | Ship the change to the device — matches how the word is used on `develop` and in `configure`/`remove` | Yes for the plain verb, but `pvr deploy` (the actual CLI subcommand) is a *different, local-only* operation that never touches a device — I'd have run the wrong command expecting it to update hardware | "`pvr deploy <deploy-dir> [source-repos]+` composes ... into a local deployment directory ... It builds the directory on disk; it does **not** push to a device — use `pvr post` for that." | S3 | `undefined-jargon` |
| "AppEngine" | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) | Not checked — the alternative wasn't on my path (I have a Pi) | Some kind of PaaS runtime, like Google App Engine or Cloud Foundry | Unverifiable from where I stood — I never clicked through, precisely because the name gave me false confidence I already knew what it was | "**No hardware?**[Run Pantavisor in Docker (AppEngine)](/development/meta-pantavisor/getting-started/how-to-install/docker) on your workstation." | S3 | `undefined-jargon` |
| `BSP` / DTBs | [`.../getting-started/develop/cli-tools/pvr-cli`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli) (`.pvr#os /tmp/export.tgz#bsp`) | Partially — a parenthetical only, on `configure` | Some fixed low-level firmware/bootloader blob, treated as a black box | Roughly, but only from outside inference (Linux embedded convention), not from anything the docs say — "BSP" is never expanded to "Board Support Package," and "DTBs" (Device Tree Blobs) is never expanded or explained at all | "bsp/ ← BSP component (squashfs files, DTBs)" | S3 | `undefined-jargon` |
| `restart-policy` value `system` | [`.../getting-started/develop/application/configure`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/configure) | No | I know "restart policy" from Docker (`on-failure`, `always`, `no`) — assumed `system` was just another value in that same enum | Can't confirm — the docs never say what values `restart-policy` can take or that `system` is special beyond "triggers a full reboot instead of a container restart," which is a materially different consequence than any Docker restart policy value | "a full reboot only happens when a `system` restart-policy container or the BSP changed" | S3 | `undefined-jargon` |
| `image` (disk sense) | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) ("flash a pre-built starter image") | Yes, quickly, by context | My first reflex was a Docker image (layers, tags, registry) | No — resolved fast once "flash" and "microSD card" appeared in the same sentence, but the half-second of wrong assumption was real | "flash a pre-built starter image and boot real hardware in about 30 minutes" | S4 | `undefined-jargon` |
| `revision` | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("the revision workflow") | Yes, by repeated example, never by explicit definition | Roughly a git commit — a versioned snapshot I can diff/roll back | Mostly right, but the docs never state the one thing that matters most: a revision is the *entire device state as one atomic unit* (all containers + config together), not a per-file or per-container change like a git commit | "Configure — overlay files and runtime manifests through the revision workflow" (`develop`); "Post the new revision to the device's pvr endpoint" (`configure`) | S4 | `undefined-jargon` |

## What worked without issue

The four "Manage applications" pages (`configure`, `view`, `access-applications`,
`remove`) are internally consistent and walk the same clone → edit → commit → `pvr post`
loop every time, with real copy-pasteable commands and realistic example output
(directory trees, `pvr status` diffs, `lxc-ls -f` output). Once I accepted that loop as
the mechanism, I never got lost inside it. `pvr-cli.md`'s "Common Usage Patterns" section
was the single most useful thing I read — a full numbered workflow end to end.

## Closing summary

- **Task outcome**: completed with detours.
- **Worst finding**: S1 — `container` is used throughout `develop` and its four
  sub-pages as though it means the same thing it does in Docker/Kubernetes, and nothing
  on the path a developer naturally follows says otherwise until deep into
  `access-applications`, where it turns out to be an LXC namespace running a pre-baked
  read-only SquashFS image with no live pull/reconcile step — a materially different
  runtime model that the docs never state up front or contrast with the assumption they
  know most new readers will bring.
- **What worked**: [`.../getting-started/develop/cli-tools/pvr-cli`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli)'s
  "Common Usage Patterns" section — a complete, concrete, copy-pasteable workflow that
  taught the clone/edit/commit/post loop better than any of the prose describing it.
- **Confidence**: mostly held the line. Two soft spots: for `bootloader`/`BSP` I leaned on
  outside embedded-Linux convention rather than anything the docs said, which I'm flagging
  rather than hiding; and for `revision`'s "atomic whole-device" property I'm inferring
  from the shape of the examples, not from an explicit statement, so treat that row as a
  reasoned guess, not a confirmed reading.
