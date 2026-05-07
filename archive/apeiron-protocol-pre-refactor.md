---
eip: <TBD>
title: Apeiron Protocol Core
author: German Abal Bazzano (@ariutokintumi)
discussions-to: https://ethereum-magicians.org/t/<apeiron-core-thread-placeholder>
status: Draft
type: Standards Track
category: ERC
created: 2022-10-16
updated: 2026-04-14
requires:
---

# Apeiron Protocol Core

## Simple Summary

A standard for Console-managed onchain Signs, where all Sign state changes occur through immutable Console functions, and cross-Console interactions rely on whitelist-based compliance checks of remote Consoles and their active Cartridges.

## Abstract

- Define the Apeiron Protocol Core standard.
- Define the three core concepts:
  - **Console**
  - **Cartridge**
  - **Sign**
- State that:
  - Sign state is stored and mutated only through Console functions.
  - Consoles are immutable after deployment.
  - A Console may have one active Cartridge at a time.
  - Cross-Console operations rely on whitelist-based compliance checks using allowed runtime code hashes.
- State that:
  - `tokenId` is the Sign representation anchor.
  - `key` is the immutable autoincrement local identifier.
  - The global unique Sign reference is `(chainId, consoleAddress, key)`.
- State that:
  - Multiple Signs MAY share the same `tokenId`.
  - Metadata is mutable while the Sign exists.
- State that:
  - transfer orchestration is expected to be initiated by compliant Cartridges or compliant Console flows,
  - while all actual Sign state changes are always executed through Console functions.
- State that:
  - the Console core includes transfer-readiness, waiting-list, pull-authorization, and inbound/outbound policy controls to guarantee service availability even when no Cartridge is currently connected.

## Motivation

### Why this standard exists

- Existing token standards do not directly model the Apeiron architecture of:
  - sovereign per-user Console storage,
  - representation anchors stored directly onchain,
  - pluggable execution modules,
  - whitelist-based interoperability between compliant counterparties,
  - always-available receive and policy primitives at the Console level.
- Apeiron Core is designed for systems where:
  - the user or controlling entity owns a dedicated Console,
  - trust in counterparties is explicit and curated through whitelist policy,
  - state integrity must remain protected from external execution logic,
  - transfer-related availability cannot depend on permanently keeping one Cartridge connected.

### Design goals

- Immutable Console logic.
- Strict storage sovereignty.
- Safe modularity through Cartridges.
- Explicit cross-Console trust boundaries.
- Standardized recovery and locking controls.
- Support for both unique and same-representation Signs.
- Always-available core transfer-preparation and policy primitives.

### Non-goals

- This ERC does **not** standardize:
  - a factory,
  - NFT migration,
  - ERC-721 or ERC-1155 compatibility,
  - a marketplace,
  - metadata compression formats,
  - frontend behavior,
  - deployment tooling.
- This ERC may be accompanied by:
  - a Pong reference Cartridge specification,
  - a Collection/Asteroids reference Cartridge specification,
  - a reference implementation repository.

## Specification

### Conventions

- The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in RFC 2119.

### Terminology

#### Console

- Immutable contract that stores and governs Signs.
- Owns the Sign storage model.
- Exposes all authorized state-changing functions.
- Holds whitelist and policy configuration.
- Holds one active Cartridge address at a time.
- Holds always-available transfer-readiness and policy state.

#### Cartridge

- External execution module used by a Console.
- May request actions through Console functions.
- MUST NOT directly modify Console storage.
- MUST NOT rely on fallback-driven storage mutation.
- Advanced execution logic is expected to be implemented through Cartridges.

#### Sign

- Apeiron token unit stored by a Console.
- A Sign has:
  - `tokenId`
  - `key`
  - `metadata`
- A Sign is globally identified by `(chainId, consoleAddress, key)`.

#### tokenId

- Immutable while the Sign exists.
- Type: `string`.
- A representation anchor chosen by the use case.
- MAY be a hash, explanation, description, identifier, or other direct reference.
- Is not required to be unique.

#### key

- Immutable local identifier.
- Autoincremented by the Console.
- Unique within one Console.
- MUST NOT be reused.

#### metadata

- Mutable while the Sign exists.
- Stored and changed only through Console functions.
- May contain direct payload data, URI-style data, or other implementation-defined metadata content.

#### active Cartridge

- The single Cartridge address currently connected to the Console.
- Used as the execution counterpart for Cartridge-driven behavior.

#### compliant Console

- A remote Console whose runtime code hash is allowed by the local Console’s whitelist policy.

#### compliant Cartridge

- A remote active Cartridge whose runtime code hash is allowed by the local Console’s whitelist policy.

### Core invariants

#### Sign state authority

- All Sign state changes MUST occur through Console functions.
- No Cartridge MAY directly mutate Console storage.
- No external module MAY bypass Console mutation rules.

#### Console immutability

- Console logic MUST be immutable after deployment.
- Console ownership MAY change.
- Console policy state MAY change according to the standard rules.

#### Cartridge connection model

- A Console MAY have at most one active Cartridge at a time.
- The active Cartridge MAY be plugged, changed, or unplugged only through Console functions and only if allowed by local policy.

#### Sign identity

- `tokenId` MUST remain immutable while the Sign exists.
- `key` MUST remain immutable forever after creation.
- Deleting a Sign removes the Sign from the Console state.
- Recreated Signs on remote Consoles receive a new local `key`.

#### Representation multiplicity

- Multiple Signs MAY share the same `tokenId`.
- This standard does not require one `tokenId` to map to only one Sign.

#### Metadata mutability

- `metadata` MAY be changed while the Sign exists.
- Metadata changes MUST occur through Console functions.
- Metadata MAY be changed by authorized owner, operator, or active Cartridge flows, subject to the implementation and policy rules defined by this standard.

#### Availability invariants

- The Console MUST expose transfer-readiness and transfer-policy primitives without requiring a Cartridge to remain connected at all times.
- Waiting lists, pull approvals, and inbound/outbound policy state MUST be part of the Console core.

#### Fallback restrictions

- Fallback and receive paths MUST NOT be used as implicit Sign mutation mechanisms.
- Implementations SHOULD minimize unexpected execution surfaces.

### Sign model

#### Required Sign state

- Each Sign MUST have:
  - `tokenId`
  - `key`
  - `metadata`

#### Identity model

- Local identity: `key`
- Global identity: `(chainId, consoleAddress, key)`
- Representation anchor: `tokenId`

#### Same-representation Signs

- This standard explicitly permits multiple Signs to represent the same thing by sharing the same `tokenId`.
- Applications MAY interpret such Signs as same-representation units or distinct units, depending on their own logic.

### Authorization model

#### Owner

- The Console MUST have an owner.
- The owner MUST be able to:
  - configure policy,
  - manage whitelist rules,
  - manage the active Cartridge,
  - manage recovery configuration,
  - manage operators,
  - perform owner-authorized Sign actions.

#### Operator

- The owner MAY approve operators.
- Operators MAY perform only the actions allowed by the standard and implementation.
- Operator approval and revocation MUST occur through Console functions.

#### Active Cartridge

- The active Cartridge MAY call only the Console functions allowed for Cartridge-driven behavior.
- A Cartridge MUST NOT gain general storage authority by being active.

### Compliance model

#### Local whitelist policy

- Each Console MUST maintain:
  - a whitelist of allowed Console runtime code hashes,
  - a whitelist of allowed Cartridge runtime code hashes.

#### Active Cartridge getter

- The Console MUST expose the current active Cartridge address through a getter.

#### Whitelist-based compliance checks

- Cross-Console operations defined by this standard or by compliant Cartridges MUST require whitelist-based compliance checks.
- A Console MUST be able to verify:
  - the remote Console runtime code hash,
  - the remote Console’s active Cartridge runtime code hash when relevant to the flow.
- A given flow MAY involve checks by one side or by both sides, depending on the operation semantics and implementation.

#### Compliance meaning

- Compliance in Apeiron Core is based on local whitelist policy.
- Compliance does not require identical addresses.
- Compliance does not require factory deployment.
- Compliance is not based on interface auto-discovery.

#### Policy responsibility

- The Console owner is responsible for the trust and risk of the configured whitelist and active Cartridge choices.

### Mandatory policy controls

#### Global lock state

- The Console MUST implement a mandatory global lock state.
- The lock state MUST govern the protected Console control surface defined by the standard.

#### Lock transition delay

- The Console MUST implement a delay model for protected state transitions.
- Lock release behavior and delay behavior MUST be standardized.

#### Console compliance delay

- The Console MUST implement delayed activation for newly added Console runtime code hashes.

#### Cartridge compliance delay

- The Console MUST implement delayed activation for newly added Cartridge runtime code hashes.

### Transfer-readiness and availability controls

#### pushWaitingList

- The Console MUST support an expected-incoming-transfer registry.
- This registry is used to record transfers that the Console is awaiting from remote Consoles.
- Registry entries SHOULD minimally reference:
  - remote `tokenId`,
  - remote `key`,
  - remote Console address,
  - and any additional policy fields required by the implementation.

#### pullTokenPreApprovedList

- The Console MUST support a pull pre-approval registry.
- This registry is used to authorize remote Consoles, specific remote addresses, or wildcard policies to pull or claim eligible Signs according to the allowed flow.

#### Inbound transfer policy

- The Console MUST support:
  - `istate`
  - `idelay`
  - inbound allowlist / waitlist
  - inbound blocklist
- The inbound blocklist MUST prevail over other inbound allowances.

#### Outbound transfer policy

- The Console MUST support:
  - `ostate`
  - `odelay`
  - outbound allowlist / waitlist
  - outbound blocklist
- The outbound blocklist MUST prevail over other outbound allowances.

#### Service availability

- These transfer-readiness and transfer-policy controls are part of the Console core to preserve operability even when the active Cartridge is changed, unplugged, or temporarily absent.

### Recovery and owner change

#### Pre-approved owner change

- The Console MUST support a pre-approved new owner flow.

#### Multi-party social recovery

- The standard SHOULD define a standardized multi-party social recovery extension.
- This extension SHOULD allow a guardian or multiparty process to recover or transfer Console control according to explicit policy.

#### Recovery reset policy

- If a Console ownership transfer is successfully executed through the standardized social recovery flow, the Console SHOULD:
  - reset `gstate` to unlocked,
  - reset `istate` to unlocked,
  - reset `ostate` to unlocked,
  - reset `glockdelay`, `consoleDelay`, `cartridgeDelay`, `idelay`, and `odelay` to a recovery-safe default.
- The RECOMMENDED recovery-safe default delay is 7 days.

#### Recovery safety goals

- Prevent irreversible loss of control.
- Reduce risks from key compromise.
- Preserve Console immutability while allowing safe ownership recovery.
- Avoid permanent lock misconfiguration after successful recovery.

### Metadata model

#### Mandatory metadata support

- Metadata support is mandatory in Apeiron Core.

#### Allowed forms

- Metadata MAY be:
  - directly stored payload data,
  - URI-style content,
  - resolver-derived output,
  - or another implementation-defined format compatible with this standard.

#### Mutable metadata

- Metadata MAY be updated while the Sign exists.

#### Base URI / resolver support

- The standard MAY define resolver-related functions as mandatory or standardized optional sections, depending on final interface scope.

#### Storage optimization note

- Compression and encoding optimizations are out of scope for the core standard.

### Transfer boundary

#### No mandatory generic user-facing transfer function in Console core

- Apeiron Core does not require one universal user-facing transfer function to be embedded directly in the Console core.

#### Console-native transfer support primitives

- Apeiron Core DOES require Console-native primitives for:
  - transfer readiness,
  - waiting lists,
  - pull pre-approvals,
  - inbound/outbound transfer policy,
  - compliance-aware receive and forwarding flows.

#### Cartridge-driven orchestration

- Transfer orchestration MAY be initiated by compliant Cartridges.
- Specific transfer models such as Pong SHOULD be specified in companion documents and reference implementations.

#### Console-executed transfer state changes

- Even when transfer is Cartridge-driven:
  - source deletion MUST occur through a Console function,
  - destination creation MUST occur through a Console function,
  - metadata preservation MUST occur through Console-governed logic.

#### Recreated Sign state

- When a Sign is recreated on a destination Console, the destination Console MUST preserve:
  - `tokenId`
  - `metadata`
- The destination Console MUST assign a new local `key`.

#### Event-level provenance

- Source Console address and source key SHOULD be emitted in events for transfer-related flows.
- They are not required as persistent core state.

### Required interface

> Note: this section is the interface outline only. Final function names and signatures will be refined in the formal draft.

#### Console read functions

- owner getter
- active Cartridge getter
- operator status query
- global lock state query
- inbound state query
- outbound state query
- delay configuration queries
- Console code hash whitelist query
- Cartridge code hash whitelist query
- Sign existence / query functions
- metadata resolver / metadata query functions
- pushWaitingList query functions
- pullTokenPreApprovedList query functions
- inbound allowlist / blocklist query functions
- outbound allowlist / blocklist query functions
- policy configuration getters

#### Console write functions

##### Sign management

- create Sign
- delete Sign
- update metadata

##### Metadata configuration

- set metadata base URI
- set metadata resolver mode
- set write-received-metadata policy

##### Compliance configuration

- add/remove Console code hash
- add/remove Cartridge code hash
- plug/change/unplug active Cartridge

##### Policy and locking

- configure global lock state
- configure lock delay
- configure Console compliance delay
- configure Cartridge compliance delay
- configure inbound state
- configure inbound delay
- configure inbound allowlist / blocklist
- configure outbound state
- configure outbound delay
- configure outbound allowlist / blocklist

##### Transfer-readiness configuration

- register expected incoming transfer
- remove or resolve expected incoming transfer entry
- pre-approve pull rights
- revoke pull rights

##### Owner and recovery

- pre-approve new owner
- owner pull request / owner claim flow
- recovery configuration functions
- social recovery functions
- recovery reset handling

##### Operator management

- approve operator
- revoke operator

##### Compliance and forwarding entrypoint

- `txFunction` or equivalent standardized execution / forwarding entrypoint used by the active Cartridge or compliant transfer flow to trigger compliance checks and Console-governed actions

##### Receive-side availability entrypoint

- standardized receive / finalize function for compliant incoming transfer flows when local waiting-list and policy conditions are satisfied

### Events

> Note: event names are placeholders and should be finalized consistently in the formal draft.

#### Sign lifecycle events

- SignCreated
- SignDeleted
- SignMetadataUpdated

#### Policy events

- ConsoleCodehashUpdated
- CartridgeCodehashUpdated
- GlobalLockUpdated
- InboundPolicyUpdated
- OutboundPolicyUpdated
- DelayUpdated

#### Availability and transfer-preparation events

- PushWaitingRegistered
- PushWaitingResolved
- PullPreApprovalUpdated

#### Cartridge events

- CartridgeConnected
- CartridgeDisconnected
- CartridgeChanged

#### Operator and owner events

- OperatorUpdated
- OwnerPreApproved
- OwnerTransferred
- RecoveryConfigured
- RecoveryExecuted
- RecoveryResetApplied

#### Transfer-related provenance events

- TransferInitiated
- TransferCompleted
- TransferRejected

### Required behavior rules

#### Sign creation

- MUST assign a fresh `key`.
- MUST store the provided `tokenId`.
- MUST initialize metadata according to the implementation-defined rules.

#### Sign deletion

- MUST remove the Sign from active existence.
- MUST NOT permit `key` reuse.

#### Metadata update

- MUST preserve `tokenId`.
- MUST preserve `key`.
- MUST only update metadata.

#### Active Cartridge change

- MUST require Cartridge compliance with local whitelist and delay rules.

#### Compliance update

- Newly added code hashes MUST respect delay rules before becoming active for compliance.

#### Lock update

- Protected state changes MUST respect the configured lock rules.

#### Transfer-preparation update

- Waiting-list, pull pre-approval, and inbound/outbound policy changes MUST respect the configured policy and delay rules where applicable.

#### Recovery execution

- A successful social recovery flow SHOULD apply the standardized recovery reset policy.

### Optional standardized extensions

#### Metadata resolver extension

- Base URI model
- resolver mode selection
- write-received-metadata policy

#### Multi-party social recovery extension

- guardian set
- threshold logic
- recovery proposal / execution
- safety delays
- recovery reset policy

#### Additional policy extension

- advanced policy controls beyond the required global, inbound, and outbound lock models

## Rationale

### Why immutable Consoles

- Protect Sign integrity.
- Reduce trust in mutable execution logic.
- Separate long-lived state from changeable application behavior.

### Why Cartridges exist

- Allow modular application behavior without granting storage authority.
- Keep Console storage protected while enabling evolving use cases.

### Why one active Cartridge

- Simpler trust model.
- Easier policy review.
- Lower policy complexity.
- Better security review surface.

### Why `tokenId` is a string

- Flexible enough for hashes, descriptions, and other direct representation anchors.

### Why `tokenId` is not required to be unique

- Multiple Signs may represent the same thing or same representation anchor.
- Global uniqueness is carried by `(chainId, consoleAddress, key)`.

### Why metadata is mutable

- Some represented things evolve over time.
- The standard permits mutable metadata while preserving Sign identity.

### Why compliance is whitelist-based

- Apeiron relies on explicit trust decisions by the Console owner.
- Runtime code hash whitelisting provides deterministic local policy.

### Why transfer-readiness controls belong in the Console core

- These controls are general-purpose availability primitives.
- They allow the Console to remain operable even when no Cartridge is currently connected.
- They reduce dependence on permanent Cartridge attachment.

### Why transfer orchestration remains outside the minimal Console core

- Transfer logic is application behavior.
- Keeping orchestration modular reduces Console attack surface while preserving flexibility.

### Why recovery resets locks

- Recovery must remain meaningful even if the prior owner misconfigured lock timing or lost keys while the Console was tightly locked.
- Resetting critical delays to a safe default prevents irrecoverable deadlock.

## Backwards Compatibility

- Apeiron Core is not a drop-in replacement for ERC-721.
- Apeiron Core is not a drop-in replacement for ERC-1155.
- Similarities may exist at the application layer, but semantics differ.
- Compatibility layers and migration paths are out of scope for the core standard.

## Security Considerations

### Direct storage access

- Cartridges MUST NOT directly write Console storage.
- Implementations MUST avoid delegatecall-style designs that violate this guarantee.

### Whitelist risk

- Incorrectly allowing Console or Cartridge code hashes can lead to loss, corruption, or unwanted behavior.

### Active Cartridge risk

- Plugging a new active Cartridge changes the allowed execution surface and must be treated as high risk.

### Availability policy risk

- Misconfigured waiting-list, pull-approval, inbound, or outbound rules can create denial-of-service conditions or unintended transfer acceptance/rejection.

### Locking and delay safety

- Delays are intended to reduce harm from mistaken or malicious policy changes.
- Implementations SHOULD emit sufficient events for offchain monitoring.

### Reentrancy and transfer ordering

- Reference implementations SHOULD delete local Sign state before making remote external calls in transfer flows.
- If remote creation fails, the whole transaction MUST revert.

### Recovery risk

- Recovery features can create their own attack surfaces if poorly configured.
- Social recovery thresholds and guardian policies should be chosen carefully.

### Recovery reset risk

- Recovery reset logic must be narrowly scoped to successful standardized recovery execution.
- It must not become an unintended bypass for ordinary lock policy.

### Fallback and implicit execution

- Unexpected fallback paths increase attack surface and should be minimized or avoided.

## Reference Implementation

- A reference implementation SHOULD be published alongside this ERC.
- The reference implementation SHOULD include:
  - Console contract
  - basic policy model
  - recovery model
  - transfer-readiness controls
  - example active Cartridge hookup
  - tests
- Transfer behavior is expected to be demonstrated through a compliant Pong reference Cartridge published alongside the Apeiron Core reference implementation.

## Discussion and companion documents

- `DISCUSSION.md` SHOULD contain:
  - broader narrative,
  - comparison to existing token models,
  - cartridge reuse rationale,
  - metadata optimization notes,
  - use case examples,
  - social and recovery discussion.
- `PONG.md` SHOULD contain:
  - transfer behavior,
  - push/pull model,
  - waiting-list model,
  - inbound/outbound policy usage,
  - Pong-specific execution semantics.
- Collection minting (Asteroids) and other Cartridges MAY be documented in companion specifications or implementation docs.

## References

- Ethereum Magicians discussion thread placeholder
- Informational repository links
- Optional informational link to project X account

## Copyright

Copyright and related rights waived via CC0.