# Persona 07 — Prompt B — Targeted task

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt B — Targeted task

> **Task: from the docs alone, write the trust-model section of your assessment.** Where
> the docs don't support a claim, write "NOT DOCUMENTED" rather than inferring — you
> would be professionally negligent to do otherwise, and the NOT-DOCUMENTED list is
> exactly what we're measuring.
>
> Cover, with a citation or a NOT DOCUMENTED for each:
>
> 1. **Root of trust.** Where does it start? Secure boot? Whose keys, held where, rotated
>    how? What's the chain from silicon to userspace?
> 2. **The signing scheme.** You'll encounter `pvs@2` and references to JWS somewhere.
>    Specify it: what's signed, what algorithm, which key, verified by what code, at what
>    moment, and **what happens on verification failure** — does it refuse, warn, or
>    proceed?
> 3. **Update integrity.** Can a device be induced to install an artifact you didn't
>    sign? Consider downgrade and replay explicitly. Do the docs address either?
> 4. **Container isolation.** Containers are not a security boundary by default. What's
>    the actual isolation story — namespaces, cgroups, seccomp, AppArmor, capabilities?
>    What can a compromised container reach? Can it reach the update mechanism?
> 5. **Device identity.** How does a device prove it's itself to the cloud? What's the
>    credential, where is it stored, and what stops it being copied to another device?
> 6. **SBOM and CVE.** Can you produce an SBOM? Is there a disclosure process, a security
>    contact, a CVE feed?
>
> Then a final judgement: **how much of this assessment did you assemble from a document
> that was written to answer it, versus reverse-engineered from pages about other
> things?** For a regulated buyer, that ratio *is* the finding.
>
> Report against `../../rubric.md`.
