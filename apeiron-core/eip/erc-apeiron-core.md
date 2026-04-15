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
- mandatory metadata support and metadata resolution behavior

Transfer orchestration MAY be implemented by compliant Cartridges or other compliant Console flows, while all actual Sign state changes are always executed through Console functions.

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
- Standardized metadata storage and resolution behavior

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

#### stored metadata

The metadata value explicitly stored for a Sign in the Console state.

#### resolved metadata

The metadata output returned by the Console according to its configured metadata resolver mode.

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

The Console MUST also expose an always-available compliant incoming transfer path.

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

#### EXTCODEHASH-based compliance

Apeiron compliance is determined exclusively by the local allowlists of runtime code hashes.

For cross-Console operations, an implementation MUST be able to verify:
- the runtime code hash of the remote Console
- the runtime code hash of the remote active Cartridge, when relevant to the flow

A remote contract address MAY be used:
- to identify a concrete endpoint to inspect
- to query its active Cartridge
- to apply local inbound or outbound policy rules

However, compliance itself MUST NOT be determined by address, by symmetry, by identical deployments, or by interface auto-discovery.

#### Compliance routine

Before any cross-Console operation that depends on Apeiron compliance, the implementation MUST:
- inspect the runtime code hash of the remote Console
- verify it against the local Console code hash allowlist
- inspect the runtime code hash of the remote active Cartridge when relevant to the flow
- verify it against the local Cartridge code hash allowlist

This compliance routine MAY be implemented internally and need not be exposed as a public API.

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

#### Mandatory metadata support

Metadata support is mandatory in Apeiron Core.

Each Console MUST support:
- stored metadata for each Sign
- a base URI configuration
- a metadata resolver mode
- a write-received-metadata policy
- a resolved metadata output function

#### Metadata storage and resolution

Apeiron Core distinguishes between:
- **stored metadata**: the metadata value explicitly stored for a Sign in the Console state
- **resolved metadata**: the metadata output returned by the Console according to its configured resolver mode

This distinction is required because a Console MAY:
- store full metadata for each Sign directly
- reconstruct metadata from a base URI
- choose whether incoming transferred metadata is stored locally or ignored

#### Metadata resolver mode

The Console MUST support a metadata resolver mode.

At minimum, the following modes MUST exist:
- `Stored`: resolved metadata is taken from the metadata stored for the Sign
- `BaseURI`: resolved metadata is constructed from the configured base URI and the Sign identifier according to the implementation rules

#### Write-received-metadata policy

The Console MUST support a `writeReceivedMetadata` policy.

When enabled, incoming metadata received through compliant transfer flows SHOULD be written into local Sign storage.

When disabled, the Console MAY ignore the incoming metadata payload for storage purposes, while the data remains recoverable from transaction history and events.

#### Console metadata identity

The Console SHOULD expose:
- `name()`
- `symbol()`

These values identify the Console and its Sign set for offchain tools and user interfaces.

#### Metadata functions

The metadata model MUST include functions equivalent in behavior to:
- `tokenMetadata`
- `metadataBaseURI`
- `metadataResolver`
- `writeReceivedMetadata`
- `showTokenMetadata`

#### Metadata forms

Stored and resolved metadata MAY contain:
- direct payload metadata
- URI-style metadata
- base64-encoded JSON
- or another compatible string-based representation defined by the implementation

Compression and encoding optimizations are out of scope for the core standard.

#### Suggested Apeiron Sign Metadata JSON Schema

The following JSON schema is RECOMMENDED for Apeiron Sign metadata:

```json
{
  "title": "Apeiron Sign Metadata",
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "Human-readable name of the Sign or represented thing."
    },
    "description": {
      "type": "string",
      "description": "Human-readable description of the Sign or represented thing."
    },
    "image": {
      "type": "string",
      "description": "A URI or data reference to an image representation."
    },
    "external_url": {
      "type": "string",
      "description": "Optional external URL related to the Sign."
    },
    "attributes": {
      "type": "array",
      "description": "Optional structured traits or attributes."
    },
    "tokenId": {
      "type": "string",
      "description": "The Apeiron representation anchor for the Sign."
    },
    "key": {
      "type": "string",
      "description": "The local immutable key of the Sign in its Console."
    },
    "console": {
      "type": "string",
      "description": "The Console address holding the Sign."
    },
    "chainId": {
      "type": "string",
      "description": "The chain identifier where the Console exists."
    },
    "representation": {
      "type": "string",
      "description": "Optional human-readable explanation of what the Sign represents."
    }
  }
}
```

#### Metadata rationale

This standard intentionally separates:
- the mutable stored metadata of a Sign
- the resolved metadata output presented to consumers

This allows Apeiron Consoles to support:
- direct metadata storage
- self-hosted metadata
- transfer flows where metadata storage is optional
- offchain-friendly metadata consumption without altering Sign identity

### Transfer Boundary

Apeiron Core does not require one universal user-facing transfer function to be embedded directly in the Console core.

Apeiron Core DOES require Console-native primitives for:
- transfer readiness
- waiting lists
- pull pre-approvals
- inbound and outbound transfer policy
- an always-available compliant incoming transfer path

Transfer orchestration MAY be initiated by compliant Cartridges or by other compliant flows defined by the implementation.

Even when transfer is Cartridge-driven:
- source deletion MUST occur through a Console function
- destination creation MUST occur through a Console function
- metadata preservation MUST occur through Console-governed logic

When a Sign is recreated on a destination Console, the destination Console MUST preserve:
- `tokenId`
- `metadata`

The destination Console MUST assign a new local `key`.

The always-available incoming transfer path MUST:
- respect inbound policy state
- respect waiting-list expectations when required
- apply whitelist-based compliance verification through runtime code hashes
- create the new local Sign only through Console functions

### Required Interface

The following canonical Solidity interface expresses the Apeiron Core behavior surface for Consoles.

```solidity
pragma solidity ^0.8.20;

enum ApeironLockState {
    Unlocked,
    Locked,
    Unlocking
}

enum ApeironMetadataResolverMode {
    Stored,
    BaseURI
}

interface IApeironConsole {
    // =============================================================
    // Events
    // =============================================================

    event SignCreated(
        uint256 indexed key,
        string tokenId,
        address indexed caller
    );

    event SignDeleted(
        uint256 indexed key,
        string tokenId,
        address indexed caller
    );

    event SignMetadataUpdated(
        uint256 indexed key,
        address indexed caller
    );

    event MetadataBaseURIUpdated(
        string newBaseURI,
        address indexed caller
    );

    event MetadataResolverUpdated(
        ApeironMetadataResolverMode newMode,
        address indexed caller
    );

    event WriteReceivedMetadataUpdated(
        bool enabled,
        address indexed caller
    );

    event ActiveCartridgeUpdated(
        address indexed previousCartridge,
        address indexed newCartridge,
        address indexed caller
    );

    event OperatorUpdated(
        address indexed operator,
        bool approved,
        address indexed caller
    );

    event OwnerPreApproved(
        address indexed candidate,
        bool approved,
        address indexed caller
    );

    event OwnershipTransferred(
        address indexed previousOwner,
        address indexed newOwner
    );

    event RecoveryGuardianUpdated(
        address indexed guardian,
        bool approved,
        address indexed caller
    );

    event RecoveryThresholdUpdated(
        uint256 newThreshold,
        address indexed caller
    );

    event RecoveryApprovalUpdated(
        address indexed guardian,
        address indexed candidate,
        bool approved
    );

    event RecoveryExecuted(
        address indexed previousOwner,
        address indexed newOwner,
        address indexed caller
    );

    event RecoveryResetApplied(
        uint256 newGlockdelay,
        uint256 newConsoleDelay,
        uint256 newCartridgeDelay,
        uint256 newIdelay,
        uint256 newOdelay
    );

    event ConsoleCodehashAllowed(
        bytes32 indexed codehash,
        uint256 activatesAt,
        address indexed caller
    );

    event ConsoleCodehashRemoved(
        bytes32 indexed codehash,
        address indexed caller
    );

    event CartridgeCodehashAllowed(
        bytes32 indexed codehash,
        uint256 activatesAt,
        address indexed caller
    );

    event CartridgeCodehashRemoved(
        bytes32 indexed codehash,
        address indexed caller
    );

    event GstateUpdated(
        ApeironLockState newState,
        address indexed caller
    );

    event IstateUpdated(
        ApeironLockState newState,
        address indexed caller
    );

    event OstateUpdated(
        ApeironLockState newState,
        address indexed caller
    );

    event GlockdelayUpdated(
        uint256 newDelay,
        address indexed caller
    );

    event ConsoleDelayUpdated(
        uint256 newDelay,
        address indexed caller
    );

    event CartridgeDelayUpdated(
        uint256 newDelay,
        address indexed caller
    );

    event IdelayUpdated(
        uint256 newDelay,
        address indexed caller
    );

    event OdelayUpdated(
        uint256 newDelay,
        address indexed caller
    );

    event InboundAllowedConsoleUpdated(
        address indexed remoteConsole,
        bool allowed,
        address indexed caller
    );

    event InboundBlockedConsoleUpdated(
        address indexed remoteConsole,
        bool blocked,
        address indexed caller
    );

    event OutboundAllowedConsoleUpdated(
        address indexed remoteConsole,
        bool allowed,
        address indexed caller
    );

    event OutboundBlockedConsoleUpdated(
        address indexed remoteConsole,
        bool blocked,
        address indexed caller
    );

    event PushWaitingRegistered(
        address indexed remoteConsole,
        uint256 indexed remoteKey,
        string remoteTokenId,
        address indexed caller
    );

    event PushWaitingCleared(
        address indexed remoteConsole,
        uint256 indexed remoteKey,
        string remoteTokenId,
        address indexed caller
    );

    event PullPreApprovalUpdated(
        uint256 indexed localKey,
        address indexed remoteConsole,
        bool approved,
        address indexed caller
    );

    event TransferInitiated(
        address indexed remoteConsole,
        uint256 indexed localKey,
        string tokenId,
        address indexed caller
    );

    event TransferCompleted(
        address indexed remoteConsole,
        uint256 indexed remoteKey,
        uint256 indexed newLocalKey,
        string tokenId
    );

    event TransferRejected(
        address indexed remoteConsole,
        uint256 indexed remoteKey,
        string tokenId,
        address indexed caller
    );

    // =============================================================
    // Console identity
    // =============================================================

    function name() external view returns (string memory);

    function symbol() external view returns (string memory);

    function owner() external view returns (address);

    function activeCartridge() external view returns (address);

    // =============================================================
    // Sign model
    // =============================================================

    function exists(uint256 key) external view returns (bool);

    function tokenIdOf(uint256 key) external view returns (string memory);

    function signCount() external view returns (uint256);

    function representationCount(
        string calldata tokenId
    ) external view returns (uint256);

    function createSign(
        string calldata tokenId,
        string calldata metadata
    ) external returns (uint256 key);

    function deleteSign(uint256 key) external;

    // =============================================================
    // Metadata model
    // =============================================================

    function tokenMetadata(
        uint256 key
    ) external view returns (string memory);

    function setTokenMetadata(
        uint256 key,
        string calldata metadata
    ) external;

    function metadataBaseURI() external view returns (string memory);

    function setMetadataBaseURI(string calldata baseURI) external;

    function metadataResolver()
        external
        view
        returns (ApeironMetadataResolverMode);

    function setMetadataResolver(
        ApeironMetadataResolverMode mode
    ) external;

    function writeReceivedMetadata() external view returns (bool);

    function setWriteReceivedMetadata(bool enabled) external;

    function showTokenMetadata(
        uint256 key
    ) external view returns (string memory);

    // =============================================================
    // Operators
    // =============================================================

    function isOperator(address operator) external view returns (bool);

    function approveOperator(address operator, bool approved) external;

    // =============================================================
    // Cartridge management
    // =============================================================

    function setActiveCartridge(address cartridge) external;

    function unsetActiveCartridge() external;

    // =============================================================
    // Compliance model
    // =============================================================

    function isConsoleCodehashAllowed(
        bytes32 codehash
    ) external view returns (bool);

    function isCartridgeCodehashAllowed(
        bytes32 codehash
    ) external view returns (bool);

    function consoleCodehashActivationTime(
        bytes32 codehash
    ) external view returns (uint256);

    function cartridgeCodehashActivationTime(
        bytes32 codehash
    ) external view returns (uint256);

    function allowConsoleCodehash(bytes32 codehash) external;

    function removeConsoleCodehash(bytes32 codehash) external;

    function allowCartridgeCodehash(bytes32 codehash) external;

    function removeCartridgeCodehash(bytes32 codehash) external;

    /// @notice Uses the remote address only as an endpoint to inspect.
    /// @dev Compliance result MUST be determined from EXTCODEHASH allowlists.
    function isRemoteConsoleCompliant(
        address remoteConsole
    ) external view returns (bool);

    /// @notice Uses the remote address only as an endpoint to inspect.
    /// @dev Compliance result MUST be determined from EXTCODEHASH allowlists.
    function isRemoteCartridgeCompliant(
        address remoteCartridge
    ) external view returns (bool);

    /// @notice Uses the remote Console address as an endpoint to inspect.
    /// @dev Compliance result MUST be determined from EXTCODEHASH allowlists.
    function isRemotePathCompliant(
        address remoteConsole
    )
        external
        view
        returns (
            bool consoleCompliant,
            bool cartridgeCompliant,
            address remoteCartridge
        );

    // =============================================================
    // Locking and delays
    // =============================================================

    function gstate() external view returns (ApeironLockState);

    function istate() external view returns (ApeironLockState);

    function ostate() external view returns (ApeironLockState);

    function glockdelay() external view returns (uint256);

    function consoleDelay() external view returns (uint256);

    function cartridgeDelay() external view returns (uint256);

    function idelay() external view returns (uint256);

    function odelay() external view returns (uint256);

    function setGstate(ApeironLockState state) external;

    function setIstate(ApeironLockState state) external;

    function setOstate(ApeironLockState state) external;

    function setGlockdelay(uint256 delaySeconds) external;

    function setConsoleDelay(uint256 delaySeconds) external;

    function setCartridgeDelay(uint256 delaySeconds) external;

    function setIdelay(uint256 delaySeconds) external;

    function setOdelay(uint256 delaySeconds) external;

    // =============================================================
    // Inbound / outbound allowlist and blocklist
    // =============================================================

    function isInboundAllowedConsole(
        address remoteConsole
    ) external view returns (bool);

    function isInboundBlockedConsole(
        address remoteConsole
    ) external view returns (bool);

    function isOutboundAllowedConsole(
        address remoteConsole
    ) external view returns (bool);

    function isOutboundBlockedConsole(
        address remoteConsole
    ) external view returns (bool);

    function setInboundAllowedConsole(
        address remoteConsole,
        bool allowed
    ) external;

    function setInboundBlockedConsole(
        address remoteConsole,
        bool blocked
    ) external;

    function setOutboundAllowedConsole(
        address remoteConsole,
        bool allowed
    ) external;

    function setOutboundBlockedConsole(
        address remoteConsole,
        bool blocked
    ) external;

    // =============================================================
    // Transfer-readiness
    // =============================================================

    function isPushWaiting(
        address remoteConsole,
        uint256 remoteKey,
        string calldata remoteTokenId
    ) external view returns (bool);

    function registerPushWaiting(
        address remoteConsole,
        uint256 remoteKey,
        string calldata remoteTokenId
    ) external;

    function clearPushWaiting(
        address remoteConsole,
        uint256 remoteKey,
        string calldata remoteTokenId
    ) external;

    function isPullPreApproved(
        uint256 localKey,
        address remoteConsole
    ) external view returns (bool);

    function setPullPreApproval(
        uint256 localKey,
        address remoteConsole,
        bool approved
    ) external;

    // =============================================================
    // Ownership transfer
    // =============================================================

    function isPreApprovedNewOwner(
        address candidate
    ) external view returns (bool);

    function preApproveNewOwner(address candidate, bool approved) external;

    function claimOwnership() external;

    // =============================================================
    // Multi-party social recovery
    // =============================================================

    function isRecoveryGuardian(address guardian) external view returns (bool);

    function recoveryGuardianCount() external view returns (uint256);

    function recoveryThreshold() external view returns (uint256);

    function setRecoveryGuardian(address guardian, bool approved) external;

    function setRecoveryThreshold(uint256 threshold) external;

    function isRecoveryApprovedBy(
        address guardian,
        address candidate
    ) external view returns (bool);

    function recoveryApprovalCount(
        address candidate
    ) external view returns (uint256);

    function approveRecoveryCandidate(
        address candidate,
        bool approved
    ) external;

    function executeRecovery(address candidate) external;

    // =============================================================
    // Always-available incoming transfer path
    // =============================================================

    /// @notice Receives and finalizes a compliant incoming Sign transfer.
    /// @dev The implementation MUST:
    ///      - apply EXTCODEHASH-based compliance checks
    ///      - respect inbound policy state
    ///      - respect waiting-list expectations when required
    ///      - create the new local Sign only through Console logic
    ///      - preserve tokenId and metadata
    ///      - assign a fresh local key
    function receiveTransfer(
        address remoteConsole,
        uint256 remoteKey,
        string calldata tokenId,
        string calldata metadata
    ) external returns (uint256 newLocalKey);
}
```

### Required Behavior Rules

#### Sign creation

A newly created Sign MUST:
- assign a fresh `key`
- store the provided `tokenId`
- initialize metadata according to the implementation-defined rules

#### Sign deletion

Deleting a Sign MUST:
- remove the Sign from active existence
- preserve the non-reusability of its `key`

#### Metadata update

Updating metadata MUST:
- preserve `tokenId`
- preserve `key`
- only affect metadata

#### Compliance configuration update

Newly added Console or Cartridge code hashes MUST respect the corresponding activation delay before becoming active for compliance.

#### Lock update

Protected state changes MUST respect the configured lock rules and delays.

#### Incoming transfer reception

The always-available `receiveTransfer` path MUST:
- inspect the runtime code hash of the calling or referenced remote Console according to the implementation flow
- apply EXTCODEHASH-based compliance checks
- apply inbound allowlist and blocklist rules
- apply waiting-list expectations when required
- preserve `tokenId` and `metadata`
- assign a fresh local `key`
- reject the flow when compliance or policy conditions fail

### Events

The implementation MUST emit events sufficient to represent:
- Sign lifecycle
- metadata configuration changes
- compliance allowlist changes
- lock and delay changes
- transfer-readiness changes
- operator and ownership changes
- recovery configuration and execution
- transfer provenance

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

### Why the incoming transfer path belongs in Console core

Because a Console must remain able to accept compliant incoming transfers even when no Cartridge is currently connected.

### Why recovery resets locks

Because recovery must remain meaningful even after harmful lock misconfiguration.

### Why metadata distinguishes stored and resolved outputs

Because Apeiron allows both direct per-Sign metadata storage and base-URI resolution, and because incoming metadata writes are configurable.

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
- Metadata resolver behavior MUST NOT alter Sign identity.
- Implementations SHOULD clearly distinguish between stored metadata and resolved metadata in offchain tooling.
- `receiveTransfer` MUST remain subject to EXTCODEHASH-based compliance and inbound policy controls.

## Reference Implementation

A reference implementation SHOULD be published alongside this ERC and include:
- Console contract
- recovery model
- transfer-readiness controls
- metadata storage and resolver behavior
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