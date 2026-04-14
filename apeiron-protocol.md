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

A standard for Console-managed onchain Signs, where all Sign state changes occur through immutable Console functions, and cross-Console interactions require compliance checks of the remote Console and its active Cartridge.

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
  - Cross-Console operations require compliance checks based on whitelisted runtime code hashes.
- State that:
  - `tokenId` is the Sign representation anchor.
  - `key` is the immutable autoincrement local identifier.
  - The global unique Sign reference is `(chainId, consoleAddress, key)`.
- State that:
  - Multiple Signs MAY share the same `tokenId`.
  - Metadata is mutable while the Sign exists.
- State that transfer behavior is not native to the Console core and is expected to be implemented by compliant Cartridges, while the actual Sign state changes are always executed through Console functions.

## Motivation

### Why this standard exists

- Existing token standards do not directly model the Apeiron architecture of:
  - sovereign per-user Console storage,
  - representation anchors stored directly onchain,
  - pluggable execution modules,
  - handshake-based interoperability between compliant counterparties.
- Apeiron Core is designed for systems where:
  - the user or controlling entity owns a dedicated Console,
  - trust in counterparties is explicit and curated through whitelist policy,
  - state integrity must remain protected from external execution logic.

### Design goals

- Immutable Console logic.
- Strict storage sovereignty.
- Safe modularity through Cartridges.
- Explicit cross-Console trust boundaries.
- Standardized recovery and locking controls.
- Support for both unique and same-representation Signs.

### Non-goals

- This ERC does **not** standardize:
  - a factory,
  - NFT migration,
  - ERC-721 or ERC-1155 compatibility,
  - a collection minting model,
  - the Pong Cartridge,
  - a marketplace,
  - metadata compression formats,
  - frontend behavior,
  - deployment tooling.

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

#### Cartridge

- External execution module used by a Console.
- May request actions through Console functions.
- MUST NOT directly modify Console storage.
- MUST NOT rely on fallback-driven storage mutation.
- Transfer logic and other advanced behaviors are expected to be implemented through Cartridges.

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
- Used as the execution counterpart for Cartridge-based behavior.

#### compliant Console

- A remote Console whose runtime code hash is whitelisted by the local Console.

#### compliant Cartridge

- A remote active Cartridge whose runtime code hash is whitelisted by the local Console.

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
- The active Cartridge MAY be plugged, changed, or unplugged only through Console functions and only if compliant with local policy.

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

#### Handshake requirement

- Cross-Console operations defined by compliant Cartridges MUST require compliance checks.
- At minimum, each side MUST verify:
  - the remote Console runtime code hash,
  - the remote Console’s active Cartridge runtime code hash.

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
- The lock state MUST govern the protected control surface defined by the standard.

#### Lock transition delay

- The Console MUST implement a delay model for protected state transitions.
- Lock release behavior and delay behavior MUST be standardized.

#### Console compliance delay

- The Console MUST implement delayed activation for newly added Console runtime code hashes.

#### Cartridge compliance delay

- The Console MUST implement delayed activation for newly added Cartridge runtime code hashes.

### Recovery and owner change

#### Pre-approved owner change

- The Console MUST support a pre-approved new owner flow.

#### Multi-party social recovery

- The standard SHOULD define a standardized optional recovery extension for multi-party social recovery.
- This extension SHOULD allow a guardian or multiparty process to recover or transfer Console control according to explicit policy.

#### Recovery safety goals

- Prevent irreversible loss of control.
- Reduce risks from key compromise.
- Preserve Console immutability while allowing safe ownership recovery.

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

#### Transfer is not native Console behavior

- Apeiron Core does not require a native transfer function in the Console core model.

#### Cartridge-driven transfer

- Transfer behavior is expected to be implemented by compliant Cartridges.

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
- lock state query
- delay configuration queries
- Console code hash whitelist query
- Cartridge code hash whitelist query
- Sign existence / query functions
- metadata resolver / metadata query functions
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

##### Owner and recovery

- pre-approve new owner
- owner pull request / owner claim flow
- recovery configuration functions
- optional social recovery functions

##### Operator management

- approve operator
- revoke operator

##### Cartridge execution entrypoint

- standardized execution / forwarding entrypoint used by active Cartridge to trigger compliant Console actions

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
- DelayUpdated

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

#### Additional policy extension

- advanced policy controls beyond the required global lock model

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
- Easier compliance checks.
- Lower policy complexity.
- Better security review surface.

### Why `tokenId` is a string

- Flexible enough for hashes, descriptions, and other direct representation anchors, what makes compliance easy and regulations friendly.

### Why `tokenId` is not required to be unique

- Multiple Signs may represent the same thing or same representation anchor.
- Global uniqueness is carried by `(chainId, consoleAddress, key)`.

### Why metadata is mutable

- Some represented things evolve over time.
- The standard permits mutable metadata while preserving Sign identity.

### Why compliance is whitelist-based

- Apeiron relies on explicit trust decisions by the Console owner.
- Runtime code hash whitelisting provides deterministic local policy.

### Why transfer is outside the Console core

- Transfer logic is application behavior.
- Keeping it in Cartridges reduces Console attack surface and preserves modularity.

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

- Incorrectly whitelisting Console or Cartridge code hashes can lead to loss, corruption, or unwanted behavior.

### Active Cartridge risk

- Plugging a new active Cartridge changes the allowed execution surface and must be treated as high risk.

### Locking and delay safety

- Delays are intended to reduce harm from mistaken or malicious policy changes.
- Implementations SHOULD emit sufficient events for offchain monitoring.

### Reentrancy and transfer ordering

- Reference implementations SHOULD delete local Sign state before making remote external calls in transfer flows.
- If remote creation fails, the whole transaction MUST revert.

### Recovery risk

- Recovery features can create their own attack surfaces if poorly configured.
- Social recovery thresholds and guardian policies should be chosen carefully.

### Fallback and implicit execution

- Unexpected fallback paths increase attack surface and should be minimized or avoided.

## Reference Implementation

- A reference implementation SHOULD be published alongside this ERC.
- The reference implementation SHOULD include:
  - Console contract
  - basic policy model
  - recovery model
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
  - waiting list model,
  - inbound/outbound policy model,
  - Pong-specific locking semantics.
- Collection minting (Asteroids) and other Cartridges MAY be documented in companion specifications or implementation docs.

## References

- Ethereum Magicians discussion thread placeholder
- Informational repository links
- Optional informational link to project X account

## Copyright

Copyright and related rights waived via CC0.