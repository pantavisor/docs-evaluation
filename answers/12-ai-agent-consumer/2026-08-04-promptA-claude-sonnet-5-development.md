# Persona 12 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: claude -p

## The answer I would actually give the developer

> To deploy an app to your Pantavisor device, use the `pvr` CLI over the local network —
> the docs call this the recommended method for development and automation (there's also
> a web-UI method and a remote-via-Pantahub method if you'd rather not use the CLI).
>
> 1. **Install `pvr`** on your workstation:
>    ```bash
>    curl -sL https://gitlab.com/pantacor/pvr/-/raw/master/install.sh | bash
>    ```
> 2. **Find your device's IP.** Run `pvr device scan` to discover Pantavisor devices on
>    your local network over mDNS (your workstation and the device need to be on the same
>    local network segment for mDNS to see it). Devices expose their pvr endpoint on port
>    12368.
> 3. **Clone the device's current state:**
>    ```bash
>    pvr clone http://<device-ip>:12368/cgi-bin mydevice
>    cd mydevice
>    ```
> 4. **Add your app as a new container**, pulling from Docker Hub or a private registry,
>    matching `--platform` to your device's CPU architecture (`linux/arm64` for a Pi
>    4/iMX8-class board, `linux/arm/v7` for a 32-bit board like an iMX6 — pick the one
>    that matches your specific board):
>    ```bash
>    pvr app add myapp --from myregistry/myimage:tag --platform linux/arm64
>    ```
>    If you're pulling from a private registry you'll need to authenticate first.
> 5. **Review and commit the change:**
>    ```bash
>    pvr status
>    pvr add .
>    pvr commit -m "add myapp container"
>    ```
> 6. **Deploy it to the device:**
>    ```bash
>    pvr post http://<device-ip>:12368
>    ```
>    Pantavisor downloads the new container objects, stages them as a pending revision,
>    and switches to it. Most updates apply without a full reboot — a reboot only happens
>    if a `system`-restart-policy container or the BSP (bootloader/kernel) changed.
> 7. **Verify it's running**, either from the device's serial/SSH console:
>    ```bash
>    lxc-ls -f
>    # or:
>    pvcontrol container ls
>    ```
>    or from the pvtx web UI at `http://<device-ip>:12368/app`.

## Self-audit

| Claim | Source (page URL) | Or: how did I get this? |
|---|---|---|
| `pvr` CLI over local network is "the recommended method for development and automation" | [`.../develop/application/install`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install) | Read it — exact phrase on the page. |
| Install command `curl -sL .../install.sh \| bash` | [`.../develop/cli-tools/pvr-cli`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli) | Read it — "Installation" section. |
| `pvr device scan` finds devices via mDNS on the local network; port 12368 | [`.../develop/cli-tools/pvr-cli`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli) | Read it — but only by reading past the installation section into a "Local network" subsection I wasn't directed to from the deploy walkthrough itself (see finding below). |
| Workstation and device must share a local network segment for mDNS to work | — | **Knew it.** Not stated anywhere in the fetched pages — general networking knowledge about mDNS/multicast not routing across subnets. |
| `pvr clone http://<device-ip>:12368/cgi-bin mydevice` | [`.../install/local-pvr`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr) | Read it — Step 1. |
| `pvr app add <name> --from <image> --platform <arch>` | same page | Read it — Step 2. |
| `linux/arm64` = Pi 4/iMX8-class, `linux/arm/v7` = iMX6-class | same page | Read it — the example comments say this, but the page never tells you how to determine *your* board's architecture from a model name; matching "your board" to one of the two examples is mine. |
| "Pick the one that matches your specific board" (i.e., you may need to identify your own hardware) | — | **Inferred it** — the docs give two labeled examples and leave the mapping for anything else as an exercise for the reader. |
| `pvr status` / `pvr add .` / `pvr commit -m "..."` | same page | Read it — Step 3. |
| `pvr post http://<device-ip>:12368` deploys the revision | same page | Read it — Step 4. |
| "a full reboot only happens when a `system`-restart-policy container or the BSP changed" | same page | Read it — verbatim on the page, Step 4, but the page does not define `system`-restart-policy (see finding below); I'm repeating the sentence, not explaining it. |
| `system` restart-policy = "requires a reboot to update"; `container` restart-policy = "restarted in place" | [glossary](https://docs.pantavisor.io/development/meta-pantavisor/overview/glossary) | Read it — but only found by independently checking the glossary and a sibling page ([`.../application/configure`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/configure)) that happens to link the same phrase to this definition. Nothing on the actual install page I followed points here. |
| `lxc-ls -f` and `pvcontrol container ls` to verify the container is running | [`.../install/local-pvr`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr) | Read it — Step 5. |
| `pvcontrol` has a full command reference (`pvcontrol ls`, `container start/stop/restart`, `steps`, `devmeta`, `cmd reboot`, etc.) | [`.../cli-tools/pvcontrol`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvcontrol) | Read it — but only found by browsing the CLI Tools index myself; the deploy walkthrough that names `pvcontrol` never links to it. |
| pvtx web UI available at `http://<device-ip>:12368/app` | [`.../install/local-pvr`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr) | Read it — Step 5. |
| Private-registry pulls need authentication | same page (links to [Authenticating Against a Private Registry](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/access-applications#authenticating-against-a-private-registry)) | Read it — linked directly from the step. |

**What fraction of the answer was actually documented?** By line-item count above: 10 of 14 claims are "Read it" directly off a page in the natural click-path from `start` → `install` → `local-pvr`. 2 more are "Read it" but only because I went spelunking through pages the walkthrough itself never linked to at the point of need (the `pvr-cli` reference's buried device-scan section, the glossary, and a sibling `configure` page) — a reader who stayed strictly on the linked path would not have found them and the answer would have those two items as gaps, not citations. The remaining 2 are genuine interpolation: one general networking fact I know but the docs never state, and one inference about matching hardware to an architecture flag the docs only demonstrate by example. So: roughly 70% cleanly sourced from the direct path, ~15% sourced from the site but only via off-path digging, ~15% outside knowledge or inference.

**Would the user be able to tell the difference?** No. Every line in the answer above — from `pvr clone` down to the mDNS caveat — is phrased with the same flat, imperative confidence. Nothing in the prose marks "workstation and device must share a network segment" as something I know rather than something the docs said, and nothing marks "pick the arch that matches your board" as my inference rather than an instruction. A developer running this verbatim would trust the network caveat and the architecture guess exactly as much as the verbatim `pvr post` command — that's the trap this persona exists to demonstrate.

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | Step 1, "Clone the Device" | The very first command needs `<device-ip>`, and nothing on this page or in its linked prerequisite tells me how to get it — I have to already know it. | `pvr clone http://<device-ip>:12368/cgi-bin mydevice` with no discovery instructions on the page. | S2 | `broken-path` | Link "device IP" in Step 1 to the `pvr device scan` mDNS-discovery command (already documented on the `pvr-cli` reference page). |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | Step 4, "Deploy to the Device" | I'm told a reboot only happens for a "`system` restart-policy container" but the page never says what that means, and doesn't link it — I'd have to repeat the phrase back without understanding it. | "a full reboot only happens when a `system` restart-policy container or the BSP changed" — no link, no definition on this page. | S2 | `undefined-jargon` | Link `system` restart-policy to `overview/glossary#restart-policy`, the same way the sibling `application/configure` page already does for the identical sentence. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | Step 5, "Verify" | I'm told to verify with `pvcontrol container ls`, but "pvcontrol" here isn't a link — the full reference for what `pvcontrol` can do lives on a different page I only found by browsing the CLI Tools index directly, not by following anything on this page. | "Or via `pvcontrol` : ```pvcontrol container ls```" — plain text, no link, in both the `.md` export and the rendered page. | S2 | `unlinked` | Link "`pvcontrol`" in Step 5 to `develop/cli-tools/pvcontrol`. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools` | Structural check (this persona counts links) | The `.md` export of this page silently drops the hyperlink for the `pvcontrol` list item (and I found the same pattern on the `getting-started/start` page's "Next steps" list) even though the rendered HTML page really does link it — an agent that follows this pack's own instruction to prefer the `.md` export would conclude `pvcontrol` has no reference page at all. | `.md` export: `- **pvcontrol** - Low-level system control interface` (plain text, no link). Rendered HTML: `pvcontrol` → `/development/meta-pantavisor/getting-started/develop/cli-tools/pvcontrol`. | S3 | `broken-path` | Fix the `.md`-export generator to preserve link syntax on bold-led list items — this is a site-tooling bug, not a missing page. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/application/install/local-pvr` | Step 4, "Deploy to the Device" | The page never says what happens if the new revision *doesn't* come up cleanly — I only know Pantavisor auto-rolls-back because the sibling `application/configure` page states it for what is otherwise the identical deploy step. | `local-pvr`: "If the new revision runs cleanly, it becomes the new permanent state." (silent on the failure case) vs. `configure`: "If it fails, the previous revision is restored automatically." | S3 | `stale` | Add the same failure/rollback sentence from `application/configure` Step 4 to `install/local-pvr` Step 4 — the two pages describe the same `pvr post` behavior and shouldn't diverge. |

**Absence of a finding, noted:** the core `pvr app add` → `pvr status/add/commit` → `pvr post` sequence on `install/local-pvr` is otherwise a clean, complete, copy-pasteable walkthrough with real commands and real example output — it's the page that carried this whole answer, and none of the above findings are about that sequence itself.

## Closing summary

- **Task outcome:** completed with detours.
- **Worst finding:** S2 — Step 1 of the deploy walkthrough (`install/local-pvr`) hands the reader `pvr clone http://<device-ip>:...` with zero indication of how to find `<device-ip>`; the answer exists (`pvr device scan`, on the `pvr-cli` reference page) but isn't linked at the point of need, so an agent without the discipline to keep digging would fabricate a discovery method instead.
- **What worked:** `.../getting-started/develop/application/install/local-pvr` — a genuinely complete, numbered, copy-pasteable install→commit→deploy→verify walkthrough with real commands and real example output; it did almost all of the work for this answer.
- **Confidence:** Mostly stayed in character, but I did use one piece of outside knowledge (that mDNS discovery requires the workstation and device to share a local network segment) and one inference (mapping "your board" to one of two example `--platform` values) — both logged in the self-audit table above rather than silently folded into the answer. Everything else traces to a page fetched during this run, though two of those traces required leaving the page the walkthrough actually links and searching the site structure myself, which this persona is explicitly permitted to do.
