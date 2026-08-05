# Persona 04 — Prompt C — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: manual

## Jargon audit

Read `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start` then
`https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop`, following
links the way a hurried app developer would: I checked the glossary (linked from `start`
via "AppEngine") for terms that felt undefined, and clicked "base" on `develop` into the
trust model page when a term there wasn't explained on the page itself.

| Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
|---|---|---|---|---|
| **Yocto** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "build example containers in the meta-pantavisor **Yocto** layer" | No — checked the glossary (no entry), and `.../overview` (meta-pantavisor overview), which uses it twice ("the Yocto/OpenEmbedded layer that builds...") but never explains it either. | Some kind of build system or toolchain, mentioned almost in passing like it's optional. | No idea, and that's the problem — I was told before I started this session that I've never heard of Yocto and have a hard stop at building an OS. This word shows up on the "develop" page like it's just another CLI tool alongside `pvr`/`pvtx`/`pvcontrol`. Nothing tells me it's the actual embedded-Linux build system I'd be trying to avoid. |
| **PID 1** | [`.../getting-started/security/trust-model`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/security/trust-model) (reached by clicking "base" on `develop`) — opening sentence: "Pantavisor owns PID 1, so it must clear a higher bar than an updater you can delete." | No — not in the glossary, and the trust-model page uses it in its very first sentence with zero gloss. | The first process the kernel starts, i.e. Pantavisor has replaced the OS's init system, not just installed a container. | Probably right (I vaguely recognize "PID 1" from Docker container-init discussions), but I'm guessing from outside knowledge, not from anything the docs told me — and this is exactly the sentence where "develop applications" turned into "this thing lives underneath your OS," one click away from the page I was sent to. |
| **BSP** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "the frozen, certified **BSP** + Pantavisor runtime layer" | Yes — the glossary defines it ("the hardware part of a revision — kernel, device trees, firmware, and kernel modules"), but neither this mention nor its second use on the trust-model page links there. | Some kind of base image bundle, like a Docker base image. | No — it's kernel/firmware/device-tree build output, not an image I'd `FROM` in a Dockerfile. I only found out because I happened to already know the glossary existed (from the "AppEngine" link on the earlier page) and went looking on my own. |
| **device state** | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) — "Install `pvr` — the client used to inspect and change device state." | Yes — glossary entry "State (state JSON)," not linked from this sentence. | Whatever config `pvr` is currently pointed at — like a git working tree. | Roughly, per the glossary ("the JSON document that fully describes a revision"), but I only got that far by having a hunch to check the glossary, not because the page told me to. |
| **pvcontrol** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "CLI tools — `pvr`, `pvtx`, and `pvcontrol` cheatsheets and workflows" | Yes — glossary entry ("on-device CLI for managing containers and metadata"), not linked from this mention. | A third CLI sibling to `pvr`, no idea what it's for specifically. | Partly — it's on-device (vs. `pvr`, which is a workstation CLI per the glossary), a distinction the page itself never draws. |
| **pvtx** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "add containers with the `pvr` CLI, the **pvtx** web UI" | Yes, but only after two more hops: the link on this page goes to the "remote via Pantahub" page, which doesn't define `pvtx` either and links onward again to a "pvtx install guide." Only the glossary ("Pantavisor's on-device update-transaction tool") actually says what it is. | A transaction/deploy tool, guessing from the "tx" in the name. | Yes, per the glossary — but three clicks to confirm a guess is a real detour for something introduced this casually. |
| **pv-xconnect** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "wire services with **pv-xconnect**" | Sort of — the glossary defines "xconnect" (no `pv-` prefix): "inter-container service connectivity through service declarations in `services.json`." Nothing confirms these are the same thing. | Assumed it's the same tool as the glossary's "xconnect," just written differently here. | Unconfirmed — the docs never actually say so. |
| **kas** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "hack on the Pantavisor runtime itself using `kas`, the Yocto workspace/build tool this layer's KAS configs are written for" | Partially — the page's own phrase calls it a "Yocto workspace/build tool," but that just moves the ignorance one word over to "Yocto" (see above), and the actual link for `kas` goes off-domain to GitHub, which is out of scope for me to follow. | A Yocto-specific build config tool. | Technically yes, per the page's own words, but the definition is circular from where I'm standing — I still don't know what building with it would require of me. |
| **A/B image updater** | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) — closing note: "There is no separate A/B image updater to install or configure underneath it." | No — not in the glossary, not explained on either page. | Some redundant dual-partition update scheme, mentioned only to reassure me I don't need to set one up separately. | Probably, from vague past exposure to the term elsewhere, not from anything this page tells me — it's trying to put me at ease using a term I'm not actually equipped to evaluate. |
| **"the way a Kubernetes controller would"** | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) — "there's no background agent continuously reconciling drift the way a Kubernetes controller would" | N/A — not a defined term, but an analogy that assumes k8s reconciliation-loop knowledge I don't have. | Something that keeps re-applying a desired config automatically, based on how I've heard Kubernetes described secondhand. | Probably close enough, but I don't run Kubernetes day to day — explaining `pvr`'s one-shot model via a k8s comparison assumes I have exactly the background this persona doesn't. |
| **Pantahub** | [`.../getting-started/start`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start) — "Next steps" list: "Access your device over serial, the local network, or Pantahub." | Yes, eventually, on the `develop` page and in the glossary — but at this specific bare mention there's no link and no gloss. | Some cloud portal, guessing from the name alone. | Yes, confirmed later, but at the point of this sentence I was purely guessing. |
| **CVE** | [`.../getting-started/develop`](https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop) — "A CVE fix in your app ships as a small container layer" | No — not in the glossary or on either page. | Common Vulnerabilities and Exposures, standard security-patch shorthand. | Yes — this one really is general industry vocabulary, not a Pantavisor-specific gap. Including it only for completeness; it didn't slow me down. |

**Terms that worked well (not gaps, noted for contrast):** "AppEngine" on the `start` page
is flagged inline ("not a PaaS") and links straight to its glossary entry at first use.
"containers" and "revision" on the `develop` page are likewise linked directly to their
glossary anchors right where they're first used — that pattern makes the gaps above (BSP,
device state, pvcontrol, pv-xconnect, Yocto) stand out more, since the same page does it
correctly in three other spots but not these.

## Closing summary

- **Task outcome:** completed.
- **Worst finding:** "Yocto" is used on the `develop` page as though it's just another
  tool name, and it is never defined anywhere I could reach (not the glossary, not the
  meta-pantavisor overview) — it's the exact concept I have a hard stop on, introduced
  with zero warning.
- **What worked:** the `start` page's handling of "AppEngine" — flagged as a specific
  term, glossed inline, and linked straight to its glossary definition at first use
  (`https://docs.pantavisor.io/development/meta-pantavisor/getting-started/start`).
- **Confidence:** mostly in character. I leaned on general software-industry knowledge
  I wasn't explicitly granted for "PID 1," "CVE," and the Kubernetes reconciliation-loop
  analogy — plausible background noise for a working app developer, but outside what the
  persona card states I know. I did not use any Pantavisor/Yocto/BSP-specific knowledge;
  every finding above reflects a term I genuinely could not resolve from the docs alone
  without deliberately going looking (glossary, one link deep into trust-model).
