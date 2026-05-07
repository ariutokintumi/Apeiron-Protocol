---
eip: <TBD>
title: Apeiron Protocol Core
author: German Abal Bazzano (@ariutokintumi)
discussions-to: https://ethereum-magicians.org/t/
status: Draft
type: Standards Track
category: ERC
created: 2022-10-16
updated: 2026-05-06
requires:
---

# Apeiron Protocol Core

## Simple Summary

A standard for Console-managed onchain Signs using a three-contract architecture: `Console`, `ConsoleStorage`, and optional `Cartridge`, where critical Sign state is isolated in `ConsoleStorage`, all compliant remote interactions are Console-to-Console, and cross-Console compatibility is determined by whitelisted runtime code hashes.

## Abstract

This standard defines Apeiron Protocol Core, a token architecture for managing onchain Signs through three components:

- **Console**: the immutable policy and execution contract
- **ConsoleStorage**: the isolated contract holding critical Sign state
- **Cartridge**: an optional execution module connected to a Console

A **Sign** is the Apeiron token unit associated with:
- `tokenId`, an immutable representation anchor of type `string`
- `key`, an immutable autoincrement local identifier
- `metadata`, mutable while the Sign exists

The global unique Sign reference is `(chainId, consoleAddress, key)`. Multiple Signs MAY share the same `tokenId`.

Apeiron Core standardizes:
- Console-owned policy and execution rules
- isolated critical Sign state in ConsoleStorage
- owner/operator permissions
- recovery flows
- mandatory lock and delay controls
- transfer-readiness primitives
- inbound and outbound transfer policy controls
- mandatory metadata support and metadata resolution behavior
- EXTCODEHASH-based compliance for Console, ConsoleStorage, and active Cartridge when present

Compliant remote communication is Console-to-Console. Cartridges MAY define orchestration logic and local execution behavior, but they MUST NOT directly access or mutate the critical Sign state stored in ConsoleStorage.

## Motivation

### Why this standard exists

Existing token standards do not directly model:
- sovereign per-user policy contracts
- isolated critical token state in a dedicated storage contract
- direct onchain representation anchors
- modular execution through pluggable Cartridges
- whitelist-based interoperability
- always-available receive and policy primitives at the Console level

Apeiron Core is designed for systems where:
- a user or entity controls a dedicated Console
- trust in counterparties is explicit
- critical Sign state must remain protected from external execution logic
- transfer availability must not depend on one Cartridge being permanently connected
- complex local execution modules can exist without being granted direct critical state authority

### Design goals

- Immutable Console logic
- Strict critical storage sovereignty
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
- a Pong reference Cartridge specification
- a Collection/Asteroids reference Cartridge specification
- a reference implementation repository

## Specification

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Terminology

#### Console

An immutable contract that:
- governs Signs
- holds whitelist and policy configuration
- manages owner/operator/recovery controls
- exposes authorized execution functions
- may hold one active Cartridge address at a time
- points to exactly one immutable `ConsoleStorage`
- performs compliant remote Console-to-Console communication

#### ConsoleStorage

A dedicated contract that:
- holds the critical Sign state
- is bound permanently to exactly one Console
- only accepts critical-state mutation calls from that Console
- exposes minimal state getters and restricted mutation functions

#### Cartridge

An optional execution module connected to a Console.

A Cartridge MAY:
- provide local execution logic
- be invoked by the Console
- be invoked by users through Console-defined flows
- consume Console helpers and policy primitives

A Cartridge MUST NOT:
- directly call or mutate `ConsoleStorage`
- become the authoritative remote caller in compliant cross-Console flows
- redefine Apeiron compliance

#### Sign

The Apeiron token unit governed by a Console and stored in ConsoleStorage.

A Sign has:
- `tokenId`
- `key`
- `metadata`

#### tokenId

An immutable `string` representation anchor chosen by the use case. It MAY be a hash, explanation, description, identifier, or other direct reference. It is not required to be unique.

#### key

An immutable autoincrement identifier local to one Console. It MUST NOT be reused.

#### metadata

Mutable while the Sign exists. The critical stored value is held in `ConsoleStorage` and its effective output may be resolved by Console rules.

#### stored metadata

The metadata value explicitly stored for a Sign in `ConsoleStorage`.

#### resolved metadata

The metadata output returned by the Console according to its configured metadata resolver mode.

### Core Invariants

#### Critical Sign state authority

Critical Sign state MUST be stored in `ConsoleStorage`.

Critical Sign state changes MUST occur only through the restricted API of `ConsoleStorage`, callable only by its bound Console.

No Cartridge MAY directly call or mutate `ConsoleStorage`.

No external module MAY bypass the Console-to-Console or Console-to-ConsoleStorage rules that protect critical Sign state.

#### Console immutability

Console logic MUST be immutable after deployment.

Console ownership MAY change.

Console policy state MAY change according to the standard rules.

#### ConsoleStorage immutability of binding

Each `ConsoleStorage` MUST be permanently bound to one Console.

That bound Console address MUST NOT be changeable after deployment.

#### Cartridge connection model

A Console MAY have at most one active Cartridge at a time.

The active Cartridge MAY be plugged, changed, or unplugged only through Console functions and only if allowed by local policy.

#### Sign identity

`tokenId` MUST remain immutable while the Sign exists.

`key` MUST remain immutable after creation.

Deleting a Sign removes the Sign from the active state.

Recreated Signs on remote Consoles receive a new local `key`.

#### Representation multiplicity

Multiple Signs MAY share the same `tokenId`.

This standard does not require one `tokenId` to map to only one Sign.

#### Metadata mutability

`metadata` MAY be changed while the Sign exists.

Metadata changes MUST occur through Console-authorized paths and the restricted `ConsoleStorage` API.

Metadata MAY be changed by authorized owner, operator, or active Cartridge flows, subject to local policy.

#### Availability invariants

The Console MUST expose transfer-readiness and transfer-policy primitives without requiring a Cartridge to remain connected at all times.

Waiting lists, pull pre-approvals, and inbound/outbound transfer policy state MUST be part of the Console core.

The Console MUST also expose an always-available compliant incoming transfer path.

#### Fallback restrictions

Fallback and receive paths MUST NOT be used as implicit critical-state mutation mechanisms.

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

Being active MUST NOT grant direct critical storage authority over `ConsoleStorage`.

### Compliance Model

#### Local whitelist policy

Each Console MUST maintain:
- a whitelist of allowed Console runtime code hashes
- a whitelist of allowed ConsoleStorage runtime code hashes
- a whitelist of allowed Cartridge runtime code hashes

#### Console identity getters

The Console MUST expose:
- the current active Cartridge address
- the bound `ConsoleStorage` address

#### EXTCODEHASH-based compliance

Apeiron compliance is determined exclusively by local allowlists of runtime code hashes.

For compliant cross-Console operations, an implementation MUST verify:
1. the runtime code hash of the remote Console
2. the runtime code hash of the remote ConsoleStorage
3. the runtime code hash of the remote active Cartridge, if a remote active Cartridge exists

A remote contract address MAY be used:
- to identify a concrete endpoint to inspect
- to query its bound ConsoleStorage
- to query its active Cartridge
- to apply local inbound or outbound policy rules

However, compliance itself MUST NOT be determined by address, by symmetry, by identical deployments, or by interface auto-discovery.

#### Canonical handshake

The Console MUST expose a public read-only handshake helper.

The canonical handshake MUST:
- inspect the runtime code hash of the remote Console
- verify it against the local Console code hash allowlist
- query the remote Console for its bound ConsoleStorage
- inspect the runtime code hash of that remote ConsoleStorage
- verify it against the local ConsoleStorage code hash allowlist
- query the remote Console for its active Cartridge
- if the remote active Cartridge is not zero, inspect its runtime code hash
- verify it against the local Cartridge code hash allowlist
- return enough information for the caller to determine whether the remote path is compliant

Cartridges MAY consume this helper.

Cartridges MAY implement their own compliance routine, but SHOULD NOT do so unless strictly necessary, since Apeiron Core defines the Console handshake as the canonical compliance path.

#### Compliant remote communication rule

In compliant cross-Console flows, the remote caller MUST be the source Console.

Cartridges MUST NOT be the direct authoritative remote caller in compliant flows.

#### Policy responsibility

The Console owner is responsible for the trust and risk of configured whitelists and active Cartridge choices.

### Mandatory Policy Controls

#### Global lock state

The Console MUST implement a mandatory global lock state.

#### Lock transition delay

The Console MUST implement a delay model for protected state transitions.

#### Console compliance delay

The Console MUST implement delayed activation for newly added Console runtime code hashes.

#### ConsoleStorage compliance delay

The Console MUST implement delayed activation for newly added ConsoleStorage runtime code hashes.

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
- reset `glockdelay`, `consoleDelay`, `consoleStorageDelay`, `cartridgeDelay`, `idelay`, and `odelay` to a recovery-safe default

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
- **stored metadata**: the metadata value explicitly stored for a Sign in `ConsoleStorage`
- **resolved metadata**: the metadata output returned by the Console according to its configured resolver mode

This distinction is required because a Console MAY:
- store full metadata for each Sign directly
- reconstruct metadata from a base URI
- choose whether incoming transferred metadata is stored locally or ignored

#### Metadata resolver mode

The Console MUST support a metadata resolver mode.

At minimum, the following modes MUST exist:
- `Stored`
- `BaseURI`

#### Write-received-metadata policy

The Console MUST support a `writeReceivedMetadata` policy.

When enabled, incoming metadata received through compliant transfer flows SHOULD be written into local Sign storage.

When disabled, the Console MAY ignore the incoming metadata payload for storage purposes, while the data remains recoverable from transaction history and events.

#### Console metadata identity

The Console SHOULD expose:
- `name()`
- `symbol()`

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

### Transfer Boundary

#### No mandatory generic user-facing transfer function in Console core

Apeiron Core does not require one universal user-facing transfer function to be embedded directly in the Console core.

#### Console-native transfer support primitives

Apeiron Core DOES require Console-native primitives for:
- transfer readiness
- waiting lists
- pull pre-approvals
- inbound/outbound transfer policy
- a canonical handshake helper
- an always-available compliant incoming transfer path

#### Cartridge-driven orchestration

Transfer orchestration MAY be initiated by compliant Cartridges.

Specific transfer models such as Pong SHOULD be specified in companion documents and reference implementations.

#### Console-executed transfer state changes

Even when transfer is Cartridge-driven:
- source deletion MUST occur through a Console-authorized call into ConsoleStorage
- destination creation MUST occur through a Console-authorized call into ConsoleStorage
- metadata preservation MUST occur through Console-governed logic

#### Recreated Sign state

When a Sign is recreated on a destination Console, the destination Console MUST preserve:
- `tokenId`
- `metadata`

The destination Console MUST assign a new local `key`.

#### Incoming transfer path

The always-available incoming transfer path MUST:
- be callable from a remote Console
- inspect `msg.sender` as the remote Console in compliant flows
- use the canonical handshake helper or an equivalent internal Console routine
- respect inbound policy state
- respect waiting-list expectations when required
- create the new local Sign only through Console-authorized calls into ConsoleStorage

### Required Interface

The following canonical Solidity interfaces express the Apeiron Core behavior surface.

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

interface IApeironConsoleStorage {
    // =============================================================
    // Events
    // =============================================================

    event BoundConsole(address indexed console);

    event SignRecordCreated(
        uint256 indexed key,
        string tokenId
    );

    event SignRecordDeleted(
        uint256 indexed key,
        string tokenId
    );

    event StoredMetadataUpdated(
        uint256 indexed key
    );

    // =============================================================
    // Binding
    // =============================================================

    function boundConsole() external view returns (address);

    // =============================================================
    // Critical Sign state getters
    // =============================================================

    function exists(uint256 key) external view returns (bool);

    function tokenIdOf(uint256 key) external view returns (string memory);

    function storedMetadataOf(uint256 key) external view returns (string memory);

    function signCount() external view returns (uint256);

    function representationCount(
        string calldata tokenId
    ) external view returns (uint256);

    function nextKey() external view returns (uint256);

    // =============================================================
    // Restricted critical Sign state mutations
    // =============================================================

    function createSignRecord(
        string calldata tokenId,
        string calldata metadata
    ) external returns (uint256 key);

    function deleteSignRecord(uint256 key) external;

    function setStoredMetadata(
        uint256 key,
        string calldata metadata
    ) external;
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
        uint256 newConsoleStorageDelay,
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

    event ConsoleStorageCodehashAllowed(
        bytes32 indexed codehash,
        uint256 activatesAt,
        address indexed caller
    );

    event ConsoleStorageCodehashRemoved(
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

    event ConsoleStorageDelayUpdated(
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

    function consoleStorage() external view returns (address);

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

    function isConsoleStorageCodehashAllowed(
        bytes32 codehash
    ) external view returns (bool);

    function isCartridgeCodehashAllowed(
        bytes32 codehash
    ) external view returns (bool);

    function consoleCodehashActivationTime(
        bytes32 codehash
    ) external view returns (uint256);

    function consoleStorageCodehashActivationTime(
        bytes32 codehash
    ) external view returns (uint256);

    function cartridgeCodehashActivationTime(
        bytes32 codehash
    ) external view returns (uint256);

    function allowConsoleCodehash(bytes32 codehash) external;

    function removeConsoleCodehash(bytes32 codehash) external;

    function allowConsoleStorageCodehash(bytes32 codehash) external;

    function removeConsoleStorageCodehash(bytes32 codehash) external;

    function allowCartridgeCodehash(bytes32 codehash) external;

    function removeCartridgeCodehash(bytes32 codehash) external;

    function txHandshake(
        address remoteConsole
    )
        external
        view
        returns (
            bool consoleCompliant,
            bool storageCompliant,
            bool cartridgePresent,
            bool cartridgeCompliant,
            address remoteStorage,
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

    function consoleStorageDelay() external view returns (uint256);

    function cartridgeDelay() external view returns (uint256);

    function idelay() external view returns (uint256);

    function odelay() external view returns (uint256);

    function setGstate(ApeironLockState state) external;

    function setIstate(ApeironLockState state) external;

    function setOstate(ApeironLockState state) external;

    function setGlockdelay(uint256 delaySeconds) external;

    function setConsoleDelay(uint256 delaySeconds) external;

    function setConsoleStorageDelay(uint256 delaySeconds) external;

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
    /// @dev In compliant flows msg.sender MUST be the remote Console.
    function receiveTransfer(
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

Newly added Console, ConsoleStorage, or Cartridge code hashes MUST respect the corresponding activation delay before becoming active for compliance.

#### Lock update

Protected state changes MUST respect the configured lock rules and delays.

#### Incoming transfer reception

The always-available `receiveTransfer` path MUST:
- treat `msg.sender` as the remote Console in compliant flows
- execute the canonical handshake against `msg.sender`
- apply EXTCODEHASH-based compliance checks for Console, ConsoleStorage, and Cartridge when present
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

### Why split Console and ConsoleStorage

Because it lets Apeiron preserve a strong guarantee: the Cartridge never needs direct critical-state authority.

### Why immutable Consoles

To protect Sign integrity, reduce trust in mutable logic, and separate long-lived policy from modular behavior.

### Why Cartridges exist

To allow modular application behavior without granting direct critical Sign storage authority.

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

- Cartridges MUST NOT directly call or mutate ConsoleStorage.
- Incorrect whitelist configuration can lead to loss or corruption.
- Active Cartridge changes are high-risk operations.
- Waiting-list and pull-approval misconfiguration can create denial-of-service or unintended transfer behavior.
- Transfer implementations SHOULD delete local Sign state before remote external calls and rely on full revert on failure.
- Recovery reset logic must not become a bypass for ordinary lock policy.
- Fallback-driven mutation MUST be avoided.
- Metadata resolver behavior MUST NOT alter Sign identity.
- Implementations SHOULD clearly distinguish between stored metadata and resolved metadata in offchain tooling.
- `receiveTransfer` MUST remain subject to EXTCODEHASH-based compliance and inbound policy controls.
- The bound Console address of ConsoleStorage MUST be immutable.

## Reference Implementation

A reference implementation SHOULD be published alongside this ERC and include:
- Console contract
- ConsoleStorage contract
- recovery model
- transfer-readiness controls
- metadata storage and resolver behavior
- example Cartridge hookup
- tests

Transfer behavior is expected to be demonstrated through a compliant Pong reference Cartridge.

## Discussion and companion documents

- `RATIONALE.md`
- `PONG.md`
- future cartridge specs
- reference implementation repository

## References

- https://ethereum-magicians.org/t/
- https://github.com/ariutokintumi/apeiron-protocol
- https://x.com/ApeironProtocol

## Copyright

Copyright and related rights waived via CC0.