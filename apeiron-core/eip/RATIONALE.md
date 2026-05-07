# RATIONALE.md

## Apeiron Protocol — Design Rationale

This document is not part of the formal ERC text.

Its purpose is to explain the design decisions behind Apeiron Protocol Core, preserve architectural reasoning, and keep important context outside the stricter normative style of the ERC draft.

The formal standard remains:
- `apeiron-core/eip/erc-apeiron-core.md`

Companion cartridge specifications remain separate, such as:
- `apeiron-core/eip/PONG.md`

Historical material and early drafts should remain archived separately.

---

## 1. Why Apeiron exists

Apeiron was designed around a simple but strong idea:

**long-lived token state, policy, and modular execution should not all live in the same trust domain.**

Most token systems merge too many responsibilities into one place:
- identity
- storage
- transfer behavior
- upgrade logic
- special-purpose application rules

Apeiron separates these concerns.

Its token unit is called a **Sign**.

Apeiron introduces:
- a **Console** for policy, execution control, compliance, and user authority
- a **ConsoleStorage** for critical Sign state
- an optional **Cartridge** for modular execution logic

This gives a more explicit architecture for trust, policy, recovery, and reuse.

---

## 2. Why the protocol uses three contracts

The current Apeiron architecture is built around:

1. **Console**
2. **ConsoleStorage**
3. **Cartridge** (optional)

### Why not only one contract?

A single contract makes it too easy for modular execution logic to gain dangerous authority over critical state, especially if one wants “transparent” or flexible cartridge execution.

### Why not only Console + Cartridge?

Because if a Cartridge is meant to behave like a connected module and there is any form of delegated or transparent execution, then protecting critical storage becomes harder or impossible to guarantee architecturally.

### Why three contracts?

The three-contract model solves this:

- **ConsoleStorage** holds the critical state.
- **Console** is the only contract allowed to mutate that state.
- **Cartridge** may extend execution behavior, but cannot directly mutate the critical storage contract.

This preserves flexibility without giving away the most sensitive authority.

---

## 3. Why ConsoleStorage exists

ConsoleStorage exists to protect the critical state of Signs.

It is the authoritative home of:
- Sign existence
- `tokenId`
- `key`
- stored metadata
- minimal counters and derived state required for Sign lifecycle

ConsoleStorage is intentionally narrow.

It should not be a second full policy engine.

It should only:
- expose state getters
- expose tightly restricted state mutation functions
- trust exactly one Console forever

Its binding to one Console should be immutable.

### Why this matters

If a user plugs a bad Cartridge into their own Console, that Cartridge may still cause local damage through the Console’s logic surface, but it still does **not** gain direct critical storage authority over the storage contract itself.

This is an important boundary.

It does not make the protocol paternalistic. It simply makes the boundaries legible and auditable.

---

## 4. Why Console exists

The Console is the user’s or entity’s main contract.

It is responsible for:
- owner/operator control
- recovery
- whitelists
- compliance
- locks and delays
- transfer-readiness
- inbound/outbound policy
- metadata resolution
- active Cartridge selection
- interaction with ConsoleStorage

The Console is also the canonical contract for compliant remote communication.

In Apeiron, **compliant remote communication is Console-to-Console**.

This means the Console is the proper place for:
- the canonical handshake
- remote endpoint discovery
- policy enforcement
- trusted use of the local Cartridge when one is connected

---

## 5. Why Cartridge exists

A Cartridge exists to let Apeiron support many kinds of execution logic without forcing all of that complexity into the Console core.

Examples:
- transfer logic
- collection minting logic
- games
- swap-like behavior
- app-specific workflows
- advanced execution patterns

The Cartridge should be:
- reusable
- replaceable
- optional
- smaller in scope than the Console
- unable to directly mutate critical Sign storage

A Cartridge may still be powerful, but it should not be the primary source of truth.

---

## 6. Why Sign state is split between identity and metadata

A Sign has:
- `tokenId`
- `key`
- `metadata`

These are not equivalent.

### `tokenId`

`tokenId` is the representation anchor.

It is:
- immutable while the Sign exists
- meaningful by design
- not necessarily unique
- potentially a hash, identifier, explanation, or other direct anchor

Apeiron intentionally gives `tokenId` semantic weight.

### `key`

`key` is the immutable local identifier.

It is:
- autoincremented by the Console / ConsoleStorage path
- unique for that Console
- never reused

Global uniqueness is derived from:

`(chainId, consoleAddress, key)`

### `metadata`

Metadata is mutable.

It exists for description, presentation, and evolving context, not for defining the protected identity core of the Sign.

---

## 7. Why `tokenId` is not required to be unique

Apeiron allows many Signs to share the same `tokenId`.

This is deliberate.

The protocol supports cases where multiple Signs represent:
- the same thing
- the same class
- the same reference anchor
- a semi-fungible represented object
- grouped or replicated representations of a real-world or logical asset

Uniqueness is not carried by `tokenId` alone.

Uniqueness is carried by:
- chain
- Console
- key

This is more flexible for structured representation systems than assuming every token anchor must be globally unique.

---

## 8. Why compliance is based on EXTCODEHASH

Apeiron compliance is intentionally deterministic and local.

Compliance is based on:
- allowed Console runtime code hashes
- allowed ConsoleStorage runtime code hashes
- allowed Cartridge runtime code hashes

This is preferable to a softer trust model based on:
- interface claims alone
- addresses alone
- branding
- social assumptions

### Why address still appears

Addresses still matter for:
- routing
- policy
- waiting lists
- pull approvals
- querying remote endpoint state

But the source of compliance truth is the runtime code hash, not the address.

---

## 9. Why compliance is 2-way or 3-way

In Apeiron, a remote path is not trusted just because a Console looks familiar.

The canonical compliance path is:

1. remote Console codehash
2. remote ConsoleStorage codehash
3. remote active Cartridge codehash, if a Cartridge exists

This gives stronger guarantees that:
- the policy contract is expected
- the critical storage contract is expected
- the optional execution module is expected

The third step is always valuable when a Cartridge exists, because the standard is not meant to be a gamble.

---

## 10. Why the handshake belongs in the Console

The Console is the canonical policy and compliance authority.

So the canonical handshake belongs there too.

The handshake should:
- inspect the remote Console
- verify its codehash
- discover the remote ConsoleStorage
- verify its codehash
- discover the remote active Cartridge
- verify its codehash if present

The handshake is useful for:
- incoming transfer acceptance
- Cartridge consumption
- generalized compliant remote flows
- keeping one canonical compliance path in the protocol

Cartridges may implement their own compliance routines, but they should not be expected to be the main source of compliance truth.

---

## 11. Why the Console must remain usable without a Cartridge

A Console should not become inert just because no Cartridge is connected.

That is why Apeiron keeps the following in the Console core:
- waiting lists
- pull pre-approvals
- inbound/outbound policy
- locks and delays
- metadata handling
- incoming compliant transfer path

This preserves baseline operability and avoids over-dependence on one active module.

---

## 12. Why transfer-readiness belongs in the Console core

At first glance, waiting lists and pull approvals may look like transfer-module concerns.

But in Apeiron they are more than that:
- they are service availability primitives
- they are expectation and authorization primitives
- they let a Console prepare for compliant activity
- they remain useful even when the active Cartridge changes

For that reason, they belong in the Console.

---

## 13. Why the incoming transfer path belongs in the Console

Apeiron requires an always-available compliant incoming transfer path in the Console.

This is necessary because:
- a Console must be able to receive compliant incoming transfers even with no Cartridge attached
- inbound policy is Console policy
- readiness is Console state
- critical Sign creation is a Console + ConsoleStorage responsibility

The Console is therefore the right place to finalize incoming compliant reception.

---

## 14. Why transfer orchestration is still modular

Although reception and core readiness are in the Console, full transfer orchestration is still modular.

Different Cartridges may want different:
- push semantics
- pull semantics
- app logic
- UX patterns
- sequencing
- preconditions

That is why Apeiron keeps the orchestration layer open to cartridges such as Pong.

This preserves flexibility without moving critical trust boundaries out of the Console.

---

## 15. Why a user can still destroy their own setup

Apeiron is not trying to prevent a user from taking local risk.

If someone deliberately plugs a destructive Cartridge into their own Console, that is their decision.

The standard is not trying to be paternalistic.

Instead, it aims to:
- isolate critical state in a separate contract
- make trust boundaries explicit
- make compliance deterministic
- make remote counterparties safer
- make recovery possible when appropriate

The blockchain remains a source of truth, not a moral babysitter.

---

## 16. Why recovery exists

A Console can hold meaningful state and policy.

Without recovery, a lost owner key can permanently destroy usability.

That is why Apeiron includes:
- pre-approved owner change
- social recovery / multisig-like recovery profile
- reset-to-safe-default behavior after successful recovery

Recovery should be real recovery, not symbolic recovery.

---

## 17. Why recovery resets critical delays

A recovered Console that remains trapped behind impossible delay or lock settings is not actually recovered.

So Apeiron recommends that successful recovery resets:
- `gstate`
- `istate`
- `ostate`
- relevant delays

to a safe default, such as one week.

This reduces the chance of permanent deadlock after recovery.

---

## 18. Why metadata distinguishes stored and resolved outputs

Apeiron supports both:
- stored metadata
- resolved metadata

This is needed because:
- some users will store metadata directly
- some will self-host and use base URI resolution
- some may choose not to persist incoming metadata locally
- offchain tools still need a canonical resolution path

This split keeps Apeiron flexible without undermining identity.

---

## 19. Why Pong is not the whole protocol

Pong is only the first reference transfer Cartridge.

It is important, but it is not Apeiron itself.

Apeiron Core should remain:
- narrower
- more general
- more reusable

Pong can then demonstrate:
- push flows
- pull flows
- reusable orchestration
- practical transfer semantics

without collapsing the whole protocol into one cartridge.

---

## 20. Why Apeiron is a self-sovereign token architecture

Apeiron is designed to give the holder or governing entity a sovereign onchain control surface.

A Console is not just a token container.

It is a policy and execution boundary controlled by its owner.

This means the holder can define:
- which remote paths are acceptable
- which incoming transfers are expected
- which outgoing transfers are allowed
- which execution module is active
- how recovery is configured

This is a different model from passive token reception systems where assets may be delivered without meaningful local policy.

In Apeiron, self-sovereignty is not only about custody.

It is also about policy, acceptance, recovery, and deterministic trust boundaries.

Not your contract, not your tokens!

---

## 21. Why Apeiron resists token spam and spoofed delivery

Apeiron is not based on an always-open passive token inbox model.

A compliant incoming Sign path must pass:
- Console policy
- waiting expectations when required
- EXTCODEHASH-based compliance checks
- the Console-controlled incoming reception path

This makes unsolicited delivery, token spam, and spoofed counterparties much harder to express within the compliant Apeiron model.

Apeiron therefore behaves closer to an onchain policy vault than to a passive token receiver.

This is one of the strongest practical consequences of the architecture.

---

## 22. Why Apeiron is useful for RWA and compliance-heavy systems

Apeiron separates critical Sign identity from descriptive metadata.

Its identity core is simple:
- `tokenId`
- `key`

Its metadata is complementary:
- mutable
- descriptive
- useful for presentation and annexed information
- not determinant of the protected identity core

This is especially useful for RWA and compliance-heavy systems because it provides:
- a simple identity model
- clearer auditability
- easier reasoning about what is critical
- easier attachment of future schemas without redefining the core of the represented Sign

---

## 23. What should remain outside the ERC

The formal ERC should stay focused.

The following are better kept outside the core ERC:
- historical narratives
- early discarded alternatives
- extended comparisons with other token models
- ecosystem strategy
- future cartridge ideas
- website messaging
- talk framing

This file exists partly so the ERC can stay formal and concise.

---

## 24. Historical note

Apeiron has roots in earlier work and hackathon-era thinking dating back to Devcon Bogotá in October 2022.

Those older materials may still be useful for:
- provenance
- historical context
- explaining discarded alternatives
- tracing the evolution of the design

But they should remain archived, not confused with the canonical current specification.