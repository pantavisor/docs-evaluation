# Persona 03 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## Task

Read `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start` and
`https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop`, following
links a developer would plausibly follow, and produce a jargon audit: every term used
before it is defined, judged strictly from a cloud-native/Kubernetes background with zero
embedded experience.

Path actually followed: `start` → `start/download-and-flash` (the primary "flash a
Raspberry Pi" path) → `operate/device-access/serial-port` (linked from
`download-and-flash`'s wiring step) → `develop/cli-tools/pvr-cli` (the "Install `pvr`"
link from `start`) → `develop` → the four "Manage applications" links
(`develop/application/install`, `.../configure`, `.../view`,
`.../access-applications`, `.../remove`) → `getting-started/benchmarks` (linked from
`develop`'s opening line) → `develop/cli-tools` (index) → `develop/cli-tools`'s
"Reference" link, which despite being labeled "exhaustive, generated CLI documentation"
actually leads to `/development/pantavisor/overview` (a conceptual index) → its
`Revisions` and `BSP` sub-pages, followed specifically to check whether jargon that
survived unexplained through the getting-started pages was defined anywhere else
in-version → `overview/pantavisor-development` and `overview/container-development`
(the "deeper development" links on `develop`), checked for the same reason. I also
peeked at `how-to-install/docker` (the "no hardware?" alternative on `start`) since a
full jargon catalog should cover both paths a reader might take from that page, not just
the one with hardware in hand.

## Jargon audit

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? | Evidence | Severity | Type |
|---|---|---|---|---|---|---|---|
| `trail` (revision trail) | [`.../develop/application/install`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install) | No — not even on the concept-dedicated `/development/pantavisor/overview/revisions` page, which defines "revision" but never structurally defines "trail" itself, only that it "enables rollback" | Some kind of ordered history of past device states I could inspect, like `git log` | Can't confirm — the word recurs on `configure`, `remove`, and the overview index, always assuming I already know what it is | "Every application on a Pantavisor device is an LXC container added to the device's revision trail." (`install`) / "The previous revision (with the container) is kept in the trail" (`remove`) | S1 | `missing-concept` |
| `the base` / `certified base` | [`.../develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) | No — checked `/development/pantavisor/overview`, `overview/revisions`, and `overview/bsp`; neither "the base" nor "certified base" appears on any of them | The BSP + runtime foundation apps sit on top of, verified/signed by some authority before it's allowed to run | Unconfirmed — nothing ever says what "certified" means mechanically (signed by whom, checked how) or names "the base" as a formal component | "Apps are versioned and updated independently of the base. ... never touches the certified base." | S1 | `missing-concept` |
| `bootloader` | [`.../operate/device-access/serial-port`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/operate/device-access/serial-port) | No | Some low-level firmware that runs before the OS kernel starts — this is outside knowledge I'm not supposed to have per my persona's boundary, not something the docs taught me | Can't confirm from the docs; they never define it, just use it | "The serial console ... shows the full boot sequence from bootloader to Pantavisor startup." | S1 | `undefined-jargon` |
| `status goal` | [`.../develop/application/view`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/view) | No — checked `develop/cli-tools/pvr-cli` specifically since it's the CLI reference; the phrase does not appear there | An enum like a Kubernetes pod phase (`Pending`/`Running`/`Ready`) a container must reach for a deploy to count as healthy | Unverifiable — `status goal` is listed as a `pvcontrol ls` column and used again on `configure` ("all containers reach their status goal, it is committed") but its possible values are never given anywhere I reached | "`pvcontrol ls` shows the Pantavisor view of each container: its name, group, status, status goal, and restart policy" | S1 | `missing-concept` |
| `container` (no live pull/reconcile) | [`.../develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("add containers with the `pvr` CLI") | Partially — the mechanics (LXC namespace, read-only SquashFS rootfs, built once by `pvr app add`/`update`) are stated on `install` and `access-applications`; the specific contrast with my assumption — that nothing pulls or reconciles it live the way `docker run`/a Deployment controller would — is never stated anywhere I found | A live OCI/Docker-style container: `pvr app add --from nginx:stable-alpine` pulls and runs an image the way `docker run` would | No — it's a pre-baked, static SquashFS image assembled once at commit time; there is no running pull loop, and the docs never say so explicitly, they only let the correct mechanics imply it if you read closely | "Pantavisor containers are isolated LXC namespaces." (`access-applications`) + "Each container's root filesystem is a read-only SquashFS image (`root.squashfs`)." (`configure`) | S1 | `missing-concept` |
| `revision` | [`.../develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("through the revision workflow") | Yes — `/development/pantavisor/overview/revisions`: "A revision is composed by a BSP ... plus a number of containers" — but only reachable via `cli-tools`'s "Reference" link, whose own anchor text ("exhaustive, generated CLI documentation") describes it as pure CLI docs, not a concept page, so I nearly didn't click it | A git-commit-like snapshot I can diff and roll back piecemeal | Partially — it IS a snapshot, but it's the entire device state as one atomic unit (BSP + every container together), not a per-file or per-container delta like a git commit; that atomicity is never stated even on the defining page, I inferred it from the definition's wording | "Configure — overlay files and runtime manifests through the revision workflow" (`develop`) vs. "For exhaustive, generated CLI documentation, see the Reference." (`cli-tools`, the only link to the page that actually defines it) | S2 | `broken-path` |
| `BSP` / `DTBs` | [`.../develop/application/configure`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/configure) (`bsp/ ← BSP component (squashfs files, DTBs)`) | `BSP` yes, via the same mislabeled "Reference" detour to `/development/pantavisor/overview/bsp` ("Board Support Package: kernel, modules, firmware..."); `DTBs` no, not expanded anywhere I reached | A fixed, opaque low-level firmware/bootloader blob I'd never touch | Roughly right for `BSP` once found, but never spelled out to "Board Support Package" anywhere on the natural getting-started path; `DTBs` (Device Tree Blobs) I only recognized from outside embedded-Linux convention, not from anything the docs said | "bsp/ ← BSP component (squashfs files, DTBs)" | S2 | `broken-path` |
| `serial console` / `USB-to-TTL adapter` | [`.../start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) ("a USB-to-TTL serial adapter for console access") | Thin — `download-and-flash` calls it "a crucial tool for debugging your device," and `serial-port` adds "the most direct access path ... shows the full boot sequence," but neither explains what the adapter physically is, how to pick or buy one, or that USB-to-TTL isn't a standard part on any machine I own | Some optional debug-logging cable, skippable if things work fine | Partially — "optional but recommended" on `start` undersells it; `download-and-flash` calls it "crucial" for seeing boot failures, but nothing tells a reader who owns none what to buy | "Optional but recommended: a USB-to-TTL serial adapter for console access." (`start`) vs. "This is a crucial tool for debugging your device." (`download-and-flash`) | S2 | `broken-path` |
| `deploy` (`pvr deploy` vs. the generic verb) | [`.../develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("drop a container from the device state and deploy") | Yes, but the word is overloaded across two different meanings | Since `develop`, `configure`, and `remove` all use "deploy" to mean "ship the change to the device," I assumed the CLI subcommand `pvr deploy` would be the one that pushes to hardware | No — `pvr deploy <dir> [repos]` only composes a local directory on disk; it is `pvr post` that actually reaches the device. I'd have run the wrong command expecting it to touch hardware | "`pvr deploy <deploy-dir> [source-repos]+` composes ... It builds the directory on disk; it does **not** push to a device — use `pvr post` for that." (`pvr-cli`) | S3 | `undefined-jargon` |
| `service mesh` (pv-xconnect) | [`.../develop/application/access-applications`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/access-applications) | Yes, in the same paragraph | An Istio/Linkerd-style sidecar layer intercepting and routing L7 network traffic between services | No — it's socket/device-node injection directly into a container's namespace; there's no proxy, no interception, no network hop involved at all | "use the **pv-xconnect** service mesh. It injects sockets or device nodes directly into a consumer container's namespace." | S3 | `undefined-jargon` |
| `kas` / `kas workspace` | [`.../develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) ("hack on the Pantavisor runtime itself with the kas workspace") | No — checked the linked `overview/pantavisor-development` page itself; `kas` appears only inside command invocations (`kas-container build ...`) and file paths (`kas/with-workspace.yaml`), never introduced or explained | Some kind of Yocto/BitBake build-orchestration wrapper, guessed from faint adjacent familiarity, not from these docs | Unconfirmable — the docs never say what `kas` is | "Use `kas/with-workspace.yaml` to develop pantavisor source locally while rebuilding through the Yocto layer." | S3 | `undefined-jargon` |
| `flash` (verb) / disk `image` | [`.../start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) ("flash a real device, boot it" / "flash a pre-built starter image") | Yes, one click away on `download-and-flash` ("Flash the Image to Your MicroSD Card... We recommend pvflasher") | "Flash" read like a deploy step; "image" read as a Docker image (layers, tags, registry) | No to both, but resolved within one page load — "flash" means writing a raw disk image to a block device (destructive, one-shot, physical media), and "image" here means a whole-disk `.wic` file, not a Docker image | "flash a pre-built starter image and boot real hardware in about 30 minutes" (`start`) | S4 | `undefined-jargon` |

## What worked without issue

The four "Manage applications" pages (`install`, `configure`, `view`, `remove`) and
`access-applications` are internally consistent and share the same clone → edit → commit
→ `pvr post` loop every time, backed by real copy-pasteable commands and realistic
example output (directory trees, `pvr status` diffs, `lxc-ls -f`/`pvcontrol` output).
Once that loop clicked, I never got lost inside it. `pvr-cli`'s "Common Usage Patterns"
section did more to teach the actual workflow than any single paragraph of prose on
`develop`.

## Closing summary

- **Task outcome**: completed with detours.
- **Worst finding**: S1 — `trail` is used repeatedly across `install`, `configure`, and
  `remove` as the thing an app is added to, removed from, and rolled back through, but it
  is never structurally defined anywhere I could reach — not even
  `/development/pantavisor/overview/revisions`, the one page whose whole job is to
  explain the state model, defines "revision" but not what a "trail" of them actually is.
- **What worked**: [`.../develop/cli-tools/pvr-cli`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli)'s
  "Common Usage Patterns" section — a complete, concrete, copy-pasteable workflow that
  taught the clone/edit/commit/post loop better than any of the prose describing it.
- **Confidence**: mostly held the line — every jargon guess above is logged as a guess,
  not asserted as fact. One soft spot: `bootloader` and `DTBs` I recognized from outside
  general Linux/embedded convention rather than anything these docs said, and I'm
  flagging that rather than quietly passing it off as something the docs taught me.
