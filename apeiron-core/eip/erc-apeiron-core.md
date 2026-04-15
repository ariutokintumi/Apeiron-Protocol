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

A standard for Console-managed onchain Signs, where all Sign state changes occur through immutable Console functions, and cross-Console interactions rely on whitelist-based compliance checks of remote Consoles and, when relevant, their active Cartridges.

## Abstract

This standard defines Apeiron Protocol Core, a Console/Cartridge architecture for managing onchain Signs.

A **Console** is an immutable contract that stores and governs Signs. A **Cartridge** is an external execution module that may request actions through Console functions but MUST NOT directly mutate Console storage. A **Sign** is the Apeiron token unit stored by a Console.

Each Sign contains:
- `tokenId`, an immutable representation anchor of type `string`
- `key`, an immutable autoincrement local identifier
- `metadata`, mutable while the Sign exists

The global unique Sign reference is `(chainId, consoleAddress, key)`. Multiple Signs MAY share the same `tokenId`.

Apeiron Core standardizes:
- Console-owned Sign state
- whitelist-based compliance
- owner/operator permissions
- recovery flows
- mandatory lock and delay controls
- transfer-readiness primitives
- inbound and outbound transfer policy controls

Transfer orchestration is expected to be implemented by compliant Cartridges or compliant Console flows, while all actual Sign state changes are always executed through Console functions.

## Motivation

### Why this standard exists

Existing token standards do not directly model:
- sovereign per-user contract storage
- direct onchain representation anchors
- modular execution through pluggable Cartridges
- whitelist-based interoperability
- always-available receive and policy primitives at the Console level

Apeiron Core is designed for systems where:
- a user or entity controls a dedicated Console
- trust in counterparties is explicit
- state integrity must remain protected from external logic
- transfer availability must not depend on one Cartridge being permanently connected

### Design goals

- Immutable Console logic
- Strict storage sovereignty
- Safe modularity through Cartridges
- Explicit cross-Console trust boundaries
- Standardized recovery and locking controls
- Support for both unique and same-representation Signs
- Always-available transfer-readiness and policy primitives

### Non-goals

This ERC does not standardize:
- factories
- ERC-721 or ERC-1155 compatibility
- migration cartridges
- marketplaces
- metadata compression formats
- frontend behavior
- deployment tooling

This ERC may be accompanied by:
- a Pong reference cartridge specification
- a Collection/Asteroids reference cartridge specification
- a reference implementation repository

## Specification

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Terminology

#### Console

An immutable contract that stores and governs Signs, exposes all authorized state-changing functions, holds whitelist and policy configuration, and may hold one active Cartridge address at a time.

#### Cartridge

An external execution module used by a Console. A Cartridge MAY request actions through Console functions but MUST NOT directly modify Console storage and MUST NOT rely on fallback-driven storage mutation.

#### Sign

The Apeiron token unit stored by a Console. A Sign has `tokenId`, `key`, and `metadata`.

#### tokenId

An immutable `string` representation anchor chosen by the use case. It MAY be a hash, explanation, identifier, or other direct reference. It is not required to be unique.

#### key

An immutable autoincrement identifier local to one Console. It MUST NOT be reused.

#### metadata

Mutable while the Sign exists. It is stored and changed only through Console functions.

### Core Invariants

#### Sign state authority

All Sign state changes MUST occur through Console functions.

No Cartridge MAY directly mutate Console storage.

No external module MAY bypass Console mutation rules.

#### Console immutability

Console logic MUST be immutable after deployment.

Console ownership MAY change.

Console policy state MAY change according to the standard rules.

#### Cartridge connection model

A Console MAY have at most one active Cartridge at a time.

The active Cartridge MAY be plugged, changed, or unplugged only through Console functions and only if allowed by local policy.

#### Sign identity

`tokenId` MUST remain immutable while the Sign exists.

`key` MUST remain immutable after creation.

Deleting a Sign removes the Sign from the Console state.

Recreated Signs on remote Consoles receive a new local `key`.

#### Representation multiplicity

Multiple Signs MAY share the same `tokenId`.

This standard does not require one `tokenId` to map to only one Sign.

#### Metadata mutability

`metadata` MAY be changed while the Sign exists.

Metadata changes MUST occur through Console functions.

Metadata MAY be changed by authorized owner, operator, or active Cartridge flows, subject to local policy.

#### Availability invariants

The Console MUST expose transfer-readiness and transfer-policy primitives without requiring a Cartridge to remain connected at all times.

Waiting lists, pull pre-approvals, and inbound/outbound transfer policy state MUST be part of the Console core.

#### Fallback restrictions

Fallback and receive paths MUST NOT be used as implicit Sign mutation mechanisms.

### Authorization Model

#### Owner

The Console MUST have an owner.

The owner MUST be able to:
- configure policy
- manage whitelist rules
- manage the active Cartridge
- manage recovery configuration
- manage operators
- perform owner-authorized Sign actions

#### Operator

The owner MAY approve operators.

Operators MAY perform only the actions allowed by the standard and implementation.

#### Active Cartridge

The active Cartridge MAY call only the Console functions allowed for Cartridge-driven behavior.

Being active MUST NOT grant general storage authority.

### Compliance Model

#### Local whitelist policy

Each Console MUST maintain:
- a whitelist of allowed Console runtime code hashes
- a whitelist of allowed Cartridge runtime code hashes

#### Active Cartridge getter

The Console MUST expose the current active Cartridge address through a getter.

#### Whitelist-based compliance checks

Cross-Console operations MUST rely on whitelist-based compliance checks.

A Console MUST be able to verify:
- the remote Console runtime code hash
- the remote Console active Cartridge runtime code hash when relevant to the flow

Compliance is determined by local whitelist policy, not by symmetry, not by identical addresses, and not by interface auto-discovery.

#### Policy responsibility

The Console owner is responsible for the trust and risk of configured whitelists and active Cartridge choices.

### Mandatory Policy Controls

#### Global lock state

The Console MUST implement a mandatory global lock state.

#### Lock transition delay

The Console MUST implement a delay model for protected state transitions.

#### Console compliance delay

The Console MUST implement delayed activation for newly added Console runtime code hashes.

#### Cartridge compliance delay

The Console MUST implement delayed activation for newly added Cartridge runtime code hashes.

### Transfer-Readiness and Availability Controls

#### pushWaitingList

The Console MUST support an expected-incoming-transfer registry.

Each entry SHOULD minimally reference:
- remote `tokenId`
- remote `key`
- remote Console address

#### pullTokenPreApprovedList

The Console MUST support a pull pre-approval registry for eligible Sign pulls.

#### Inbound transfer policy

The Console MUST support:
- `istate`
- `idelay`
- inbound allowlist
- inbound blocklist

Inbound blocklist MUST prevail over other inbound allowances.

#### Outbound transfer policy

The Console MUST support:
- `ostate`
- `odelay`
- outbound allowlist
- outbound blocklist

Outbound blocklist MUST prevail over other outbound allowances.

### Recovery and Owner Change

#### Pre-approved owner change

The Console MUST support a pre-approved new owner flow.

#### Multi-party social recovery

The standard SHOULD define a standardized multi-party social recovery extension.

#### Recovery reset policy

If ownership is successfully transferred through the standardized social recovery flow, the Console SHOULD:
- reset `gstate`, `istate`, and `ostate` to unlocked
- reset `glockdelay`, `consoleDelay`, `cartridgeDelay`, `idelay`, and `odelay` to a recovery-safe default

The RECOMMENDED recovery-safe default is 7 days.

### Metadata Model

Metadata support is mandatory.

Metadata MAY be:
- directly stored payload data
- URI-style content
- resolver-derived output
- another implementation-defined compatible format

Compression and encoding optimizations are out of scope for the core standard.

### Transfer Boundary

Apeiron Core does not require one universal user-facing transfer function to be embedded directly in the Console core.

Apeiron Core DOES require Console-native primitives for:
- transfer readiness
- waiting lists
- pull pre-approvals
- inbound and outbound transfer policy
- compliance-aware receive and forwarding flows

Transfer orchestration MAY be initiated by compliant Cartridges.

Even when transfer is Cartridge-driven:
- source deletion MUST occur through a Console function
- destination creation MUST occur through a Console function
- metadata preservation MUST occur through Console-governed logic

When a Sign is recreated on a destination Console, the destination Console MUST preserve:
- `tokenId`
- `metadata`

The destination Console MUST assign a new local `key`.

### Required Interface

> Note: final names and exact signatures are intentionally left for the formal pass.

#### Console read functions

- owner getter
- active Cartridge getter
- operator status query
- global lock state query
- inbound state query
- outbound state query
- delay queries
- Console code hash whitelist query
- Cartridge code hash whitelist query
- Sign existence/query functions
- metadata query functions
- pushWaitingList query functions
- pullTokenPreApprovedList query functions
- inbound allowlist/blocklist query functions
- outbound allowlist/blocklist query functions

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
- configure inbound allowlist/blocklist
- configure outbound state
- configure outbound delay
- configure outbound allowlist/blocklist

##### Transfer-readiness configuration

- register expected incoming transfer
- resolve expected incoming transfer
- pre-approve pull rights
- revoke pull rights

##### Owner and recovery

- pre-approve new owner
- owner pull request / owner claim flow
- recovery configuration
- social recovery execution
- recovery reset handling

##### Operator management

- approve operator
- revoke operator

##### Compliance and forwarding entrypoint

- `txFunction` or equivalent standardized execution/forwarding entrypoint used by the active Cartridge or compliant transfer flow to trigger compliance checks and Console-governed actions

##### Receive-side availability entrypoint

- standardized receive/finalize function for compliant incoming transfer flows when waiting-list and policy conditions are satisfied

### Events

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

#### Availability events

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

#### Transfer provenance events

- TransferInitiated
- TransferCompleted
- TransferRejected

## Rationale

### Why immutable Consoles

To protect Sign integrity, reduce trust in mutable logic, and separate long-lived state from modular behavior.

### Why Cartridges exist

To allow modular execution behavior without granting storage authority.

### Why one active Cartridge

To simplify trust, review, and policy complexity.

### Why `tokenId` is a string

To support hashes, descriptions, identifiers, and other direct representation anchors.

### Why `tokenId` is not required to be unique

Because multiple Signs may represent the same thing, while global uniqueness is carried by `(chainId, consoleAddress, key)`.

### Why transfer-readiness controls belong in Console core

Because they are general-purpose availability primitives and should not depend on a permanently connected Cartridge.

### Why recovery resets locks

Because recovery must remain meaningful even after harmful lock misconfiguration.

## Backwards Compatibility

Apeiron Core is not a drop-in replacement for ERC-721 or ERC-1155.

Compatibility layers and migration paths are out of scope for the core standard.

## Security Considerations

- Cartridges MUST NOT directly write Console storage.
- Incorrect whitelist configuration can lead to loss or corruption.
- Active Cartridge changes are high-risk operations.
- Waiting-list and pull-approval misconfiguration can create denial-of-service or unintended transfer behavior.
- Transfer implementations SHOULD delete local Sign state before remote external calls and rely on full revert on failure.
- Recovery reset logic must not become a bypass for ordinary lock policy.
- Fallback-driven mutation MUST be avoided.

## Reference Implementation

A reference implementation SHOULD be published alongside this ERC and include:
- Console contract
- recovery model
- transfer-readiness controls
- example Cartridge hookup
- tests

Transfer behavior is expected to be demonstrated through a compliant Pong reference Cartridge.

## Discussion and Companion Documents

- `DISCUSSION.md`
- `PONG.md`
- future cartridge specs
- reference implementation repository

## References

- Ethereum Magicians thread placeholder
- repository links
- informational project links

## Copyright

Copyright and related rights waived via CC0.