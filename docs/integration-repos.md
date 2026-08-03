# Integration Repositories: Structure and Naming

## Summary

Substrate is acquiring its first end-to-end integrations — workloads that *run
on* Substrate rather than demonstrate it. This document records where that code
lives, how the repositories are named, and how fixes flow back into core.

In short: trivial demos stay in the core repository, each non-trivial
integration gets one dedicated repository under the `agent-substrate`
organization, and gaps in core are closed by making core configurable rather
than by patching it downstream.

## Motivation

Until now every in-repo example has been small enough to live beside the code it
exercises. The first real integrations are not: they carry their own container
images, dependencies, release cadence, and potentially their own maintainers.

Without a written convention, whichever repository happens to be created first
sets the precedent for everything after it. This document makes the convention
explicit instead, so that the choice is deliberate.

## Where code lives

**Stays in the core repository.** Trivial demos and keyless API exercisers — the
counter demo, and the lifecycle mocks that CI uses to drive create, resume, and
suspend. The test is roughly: no API keys, no external services, no third-party
accounts required to run it.

**Gets its own repository under `agent-substrate`.** Non-trivial, end-to-end
integrations, including their code, container images, manifests, and SDKs. One
repository per integration. These are too large to carry in core, and a
dedicated repository lets them have their own maintainers without granting
access to core.

**Not a second organization.** GitHub supports only one level of organization
parentage, so a nested org is not actually available; a sibling org would add
onboarding and access-management overhead that a small number of peer
repositories under `agent-substrate` does not.

| Thing | Where it goes |
|---|---|
| Counter demo, keyless lifecycle mocks used by CI | Core repository |
| Non-trivial end-to-end integration (code, images, manifests, SDKs) | `agent-substrate/<name>` |
| A fix or new knob in Substrate that an integration needs | PR to the core repository |

## Naming

Two cases, depending on what the repository actually is:

**Capability-named**, when it provides a general Substrate capability that
happens to have one implementation today. Prefer `code-execution-sandbox` over
`sandbox` (too broad) or a vendor's product name (too narrow).

**Integration-named**, when it integrates one specific third-party product. Name
it for the product, not the vendor behind it — `hermes`, not the name of the
organization that publishes Hermes.

Avoid:

- **Generic names** such as `sandbox` or `plugins`, which claim far more ground
  than any one repository covers.
- **Names that clone a vendor's API or brand**, which quietly commits the
  project to chasing someone else's naming decisions.
- **The `-integration` suffix.** Every repository in this category is an
  integration, so the suffix carries no information: `hermes`, not
  `hermes-integration`.

Third-party names may be used descriptively. When one is, the repository README
should note that the project is not affiliated with the third party and that
trademarks belong to their respective owners. Clear brand and policy edge cases
before the repository is created, not after.

## Upstreaming: prefer configurability over forks

Real integrations surface real gaps in core. Building a long-running agent on
Substrate turned up the need for configurable timeouts and golden-snapshot
warmup ([#487](https://github.com/agent-substrate/substrate/pull/487)), and ran
into suspend-safe actor networking
([#465](https://github.com/agent-substrate/substrate/issues/465)).

The path for those fixes should be short and well-travelled. An integration
repository that accumulates local patches against core behavior will bitrot, and
the gap it works around stays invisible to everyone else.

The corollary is a design preference for core: when core behavior blocks an
integration, prefer making that behavior **configurable with defaults
unchanged** over forking it, vendoring it, or special-casing the caller.

## Worked examples

These two validate the convention rather than merely following it:

- **`agent-substrate/code-execution-sandbox`** — capability-named. A sandboxed
  code-execution service built on Substrate, in the spirit of existing
  code-execution products but not modeled on any one of their APIs.

- **`agent-substrate/always-on-agent`** — a connection-holding agent: a
  multi-tenant gateway plus a suspendable per-conversation actor. Its first
  implementation is built on a third-party agent runtime, and the repository is
  capability-named rather than vendor-named while that name goes through the
  brand check described above. It is the third-party-name edge case in practice.

## Not settled yet

These are open questions for the maintainers, deliberately left out of scope
here so they do not block the first repositories:

- **Governance tiers.** Whether to distinguish "official" from "community"
  integrations with different review bars, as Home Assistant and Obsidian do.
  Likely worth revisiting once there are more than a handful of integrations.
- **Repository creation and access.** Who creates integration repositories, and
  who grants per-integration maintainer access.
