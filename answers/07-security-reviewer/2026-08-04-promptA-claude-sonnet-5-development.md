# Persona 07 — Prompt A — 2026-08-04

Target: https://docs.pantavisor.io/development/
Date: 2026-08-04
Model: claude-sonnet-5
Version: development
Run by: manual (fresh session, docs-framework repo)

## Trust model, as assembled

I landed on `start`, which has nothing security-shaped on it — three "paths" (flash a
Pi, run in Docker, install `pvr`) and a prerequisites list. Nothing here says "security"
or "trust," so I went hunting through whatever it linked to, and then whatever those
pages linked to. What follows is the model as it accreted, in the order I hit it, with
every inference I had to make between documents flagged.

1. **`start` → glossary** (via the AppEngine footnote). The glossary itself defines
   dozens of terms — but not `secure boot`, `signing`, `key`, `TPM`, `attestation`,
   `SBOM`, `CVE`, or `trust`. Its `State` entry pointed me at the state-format
   reference; that's where the model actually starts.
2. **State format reference → Section 9, "Security."** `_sigs/<container>.json` is a
   JWS-based artifact-verification manifest: `alg`, `x5c` cert chain or `jwk`,
   `signature`. The root table marks it **Mandatory: No** — so on first read, signing
   looked optional, full stop.
3. **`containers` overview** (linked from the state format's `run.json` row), for the
   "what can a compromised container reach" half of the task. This is where roles live:
   `mgmt` vs `nobody`, each gating a different slice of the `/pantavisor/` control
   surface. `mgmt` containers additionally get "Challenge and device-id information for
   Pantacor Hub" — first appearance of the word "Challenge," unexplained.
4. **`updates` overview** (linked from `containers`' restart-policy section), to see
   what actually happens when a revision fails. Its `WONTGO`/`ERROR` state tables list
   "State not fully covered by signatures" and "Signature validation failed" as causes
   — which contradicts what I inferred in step 2 (signing looked optional; here it can
   fail a boot). Both point at `storage#state-signature`, so I followed that.
5. **`storage` overview → "Integrity."** This is the closest thing to a real security
   page on the whole site, and it resolves step 2/4's contradiction: validation has
   four severities (`disabled`/`audit`/`lenient`/`strict`), `lenient` is the default,
   and `lenient` only validates artifacts that *are* signed — which is why "Mandatory:
   No" and "can fail a boot" are both true depending on config. It also names real
   algorithms (RS256, ES256, ES384, ES512), states the two verification points (boot
   and update-receipt), and describes both an on-disk public key and an x.509
   certificate-chain option with a build-time root certificate. This is the first page
   that reads like it was written by someone who has this persona's job.
6. **`storage` → "Boot"** and, from there, **`bsp` → "Bootloader"** (both linked
   in-page). This is where the model runs out. `bsp`'s Bootloader section describes
   U-Boot/GRUB purely as a revision pointer reader — it loads whatever kernel+initrd
   the last-written revision names. Step 5's validation happens "right after Pantavisor
   is initialized" — i.e., after the bootloader has already loaded and executed
   whatever it was pointed at. Nothing between power-on and that point is described as
   verified. The "roots" half of "from the roots — secure boot, key custody" never
   materializes.
7. **`disks` overview** (linked from `storage`'s Disks section), for storage
   encryption. This is thorough and good: CAAM/DCP hardware-backed key sealing via the
   kernel trusted/logon keyring, cipher choices, key-migration procedure. Genuinely
   answers "how are secrets at rest protected," for the device side.
8. **`remote-control` overview** (the link target for step 3's "Challenge... for
   Pantacor Hub"), to resolve the undefined term. It doesn't. The client state machine
   (`register`/`claim`/`sync`/`login`/`wait hub`/...) never uses the word "challenge"
   anywhere on the page.
9. **`local-control` overview** (linked from `containers`' roles section), to check
   what actually gates the `mgmt` control socket once a container has it mounted. It
   doesn't say — the page's own example is "you can always directly use cURL... to
   attack the control socket endpoints," with no credential in the request.
10. **`pvr-cli`** (linked from `start`, for the `pvr sig` commands referenced in step
    2). Its own "Official Documentation" section punts the full command reference to
    `docs.pantahub.com` — a different domain, out of my fence, labeled "Legacy."

## Findings

| Location | Task step | What blocked me | Evidence | Severity | Type | Suggested fix |
|---|---|---|---|---|---|---|
| `https://docs.pantavisor.io/development/pantavisor/overview/bsp`, `https://docs.pantavisor.io/development/pantavisor/overview/storage` | A, step 6 — tracing the chain back to a hardware root of trust | I'm told exactly how Pantavisor validates a revision's signature once it's running, but nothing describes what (if anything) authenticates the bootloader, kernel, or initrd it just loaded *before* that point. The trust chain has no documented beginning. | bsp: "Pantavisor is solely responsible for writing the target revision into the bootloader environment; the bootloader only reads from it at boot time to locate the kernel and initrd to load." storage: validation happens "when booting up, right after Pantavisor is initialized." | S1 | `missing-concept` | State explicitly what (fused key, HAB/AHAB, U-Boot verified boot, or nothing) authenticates the boot chain before Pantavisor's own signature check runs. |
| `https://docs.pantavisor.io/development/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli` | A, step 10 — understanding how a signature is actually produced | The one page in-fence that should explain `pvr sig add`/`pvr sig ls` sends me off-domain for the real reference, to a site explicitly labeled legacy. | "## Official Documentation ... **[PVR CLI Reference](https://docs.pantahub.com/pvr/)** - Legacy Pantahub documentation" | S1 | `outside-docs` | Either maintain the `pvr sig` command reference on `docs.pantavisor.io` or drop "Legacy" and make clear it's still the authoritative source. |
| `https://docs.pantavisor.io/development/pantavisor/overview/containers`, `https://docs.pantavisor.io/development/pantavisor/overview/remote-control` | A, step 8 — resolving what a compromised `mgmt` container gains | `mgmt` containers are granted "Challenge... information for Pantacor Hub" as a named privilege, but the page that phrase links to never uses the word "challenge" or describes a challenge-response mechanism anywhere. | containers: "Challenge and device-id information for Pantacor Hub." remote-control's state list (init/register/claim/sync/login/wait hub/report/idle/prep download/download) never mentions it. | S1 | `undefined-jargon` | Define what the Hub "Challenge" is (protocol, format, what it authenticates) either where it's named or on the page it links to. |
| `https://docs.pantavisor.io/development/pantavisor/overview/storage` | A, step 5 — establishing key custody for the OEM's own signing key | I'm told the device-side public key/root cert is placed at build time, but nothing says how the OEM should generate, store, or rotate the *private* key used to sign revisions in the first place — the half of the key pair that actually matters for a security sign-off. | "Additionally to artifact checksum, the state JSON can be signed. This can be done from your host computer or from an automated CI, and it is then validated from Pantavisor..." — no further guidance follows. | S1 | `missing-concept` | Document a recommended signing-key custody practice (HSM, offline signing, rotation cadence) for OEMs, or state plainly that this is left entirely to the integrator. |
| `https://docs.pantavisor.io/development/pantavisor/overview/local-control` | A, step 9 — establishing what gates the `mgmt` control socket | Nothing states whether the `pv-ctrl` socket requires any credential beyond a container having the `mgmt` role mounted into it. The page's own usage example uses a bare `cURL` call. | "Besides this option, you can always directly use cURL or any other HTTP client to attack the control socket endpoints." | S2 | `missing-concept` | State explicitly whether `pv-ctrl` performs any authentication/authorization beyond the socket being present in the container's mount namespace. |
| `https://docs.pantavisor.io/development/pantavisor/reference/pantavisor-state-format-v2`, `https://docs.pantavisor.io/development/pantavisor/overview/updates`, `https://docs.pantavisor.io/development/pantavisor/overview/storage` | A, steps 2–5 — reconciling "signing is optional" with "unsigned state fails to boot" | The schema reference says the signature manifest is not mandatory; two pages later, the update-state reference says an update can fail specifically because it isn't "fully covered by signatures." The two only reconcile once a third page explains severity levels — I carried the wrong assumption for two full hops. | state-format-v2: "`_sigs/<container>.json` \| Signature Manifest \| No". updates: "State not fully covered by signatures \| The revision has some element that is not signed." | S3 | `broken-path` | Cross-link the state-format Security section and the updates failure-cause table directly to `storage#state-signature`'s severity-level explanation, rather than leaving the reader to notice the tension and resolve it via a third page. |

## Absence of findings

Once I reached `storage`'s "Integrity" section, the artifact-checksum and
state-signature story is genuinely well specified: named algorithms (RS256, ES256,
ES384, ES512), two explicit validation points (boot and update-receipt), four severity
levels with a stated default, and both an on-disk-key and a certificate-chain-of-trust
option with the CN-matching logic spelled out for OEM vs default root certs. `disks`
is equally strong on data-at-rest: CAAM/DCP hardware-backed key sealing via the kernel
trusted/logon keyring, explicit cipher strings, and a documented key-migration path
from legacy NXP blobs to the mainline trusted-key subsystem — I would not have to guess
at any of that for a review. `containers`' Roles section cleanly draws the privilege
line between `mgmt` and `nobody` containers in terms of exactly which sockets and
metadata each gets mounted. None of these needed a detour outside `docs.pantavisor.io`.

## Verdict

**Blocked pending information.** Not because any one gap is unrecoverable, but because
the persona I'm running as cannot sign an assessment with the boot chain's root of
trust undocumented — everything downstream (state signature, disk encryption, role
separation) is real and well-specified, but it all sits on top of a bootloader stage
the docs never claim is authenticated at all. That's the one gap that would need to
close before "go."

Questions that would unblock me, most important first:

1. What authenticates the bootloader/kernel/initrd chain before Pantavisor's own state
   signature validation runs — is there a hardware/fused root of trust, U-Boot verified
   boot, or is this stage intentionally left unauthenticated?
2. What is the "Challenge" that `mgmt` containers get access to for Pantacor Hub, and
   what does it protect against (device impersonation? replay?)?
3. What is the recommended custody practice for the OEM's own state-signing private
   key — is offline/HSM signing expected, and is there a rotation story?
4. Does the `pv-ctrl` control socket perform any authentication of its own, or is
   container placement (the `mgmt` role) the entire access-control boundary?
5. Is there a maintained, in-domain reference for `pvr sig` command semantics, given
   the current one is off-domain and labeled legacy?

## Closing summary

- **Task outcome:** blocked pending information — the parts of the trust model that
  exist (state signature, disk encryption, container role separation) are genuinely
  well documented once found, but the chain has no documented root, and I won't sign
  off on a security posture I can only see the middle of.
- **Worst finding:** the docs describe in detail how Pantavisor verifies a revision's
  signature once it is running, but never say what — if anything — verifies the
  bootloader, kernel, and initrd it just loaded to get there, leaving the trust chain
  with no documented beginning.
- **What worked:** `https://docs.pantavisor.io/development/pantavisor/overview/storage`
  (the "Integrity" section) — named algorithms, explicit validation points and failure
  behavior, and a real explanation of the certificate-chain-of-trust option. The single
  best page on the site for this persona.
- **Confidence:** I stayed in character — every claim above traces to a quote I
  actually fetched, and I logged the "Challenge" term, the signing-optional/mandatory
  contradiction, and the bootloader silence as gaps rather than filling them in from
  what I'd expect a product like this to do. I did not visit anything outside
  `docs.pantavisor.io/development/`, and where a page pointed off-domain
  (`docs.pantahub.com`) I recorded it as a finding instead of following it.
