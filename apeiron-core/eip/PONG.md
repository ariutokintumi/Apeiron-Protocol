# PONG.md

## Apeiron Protocol Cartridge "Pong"

## Summary

Pong is the first reference Cartridge for Apeiron Protocol Core.

Pong standardizes transfer-oriented Sign flows, including push and pull semantics, while relying on Apeiron Console primitives for:
- readiness
- compliance
- policy
- critical state mutation
- incoming reception

Pong does not own critical Sign state and MUST NOT directly call or mutate `ConsoleStorage`.

## Purpose

Pong exists to provide a concrete, reusable, and auditable transfer module for Apeiron Consoles.

Pong primarily defines:
- transfer-oriented orchestration semantics
- push flows
- pull flows
- how a Cartridge should consume the canonical Console handshake
- how a Cartridge should use Console-native readiness and policy primitives
- how a Cartridge should preserve Apeiron transfer semantics

Pong is not the source of critical Sign truth.

The authoritative responsibilities remain:
- **Console** for policy, handshake, and compliant remote communication
- **ConsoleStorage** for critical Sign state

## Relationship to Apeiron Core

Apeiron Core already defines and requires the following Console-native primitives:
- `consoleStorage()`
- `activeCartridge()`
- `txHandshake(...)`
- `receiveTransfer(...)`
- `pushWaitingList`
- `pullTokenPreApprovedList`
- `istate`
- `idelay`
- inbound allowlist
- inbound blocklist
- `ostate`
- `odelay`
- outbound allowlist
- outbound blocklist
- metadata storage and resolution behavior

Pong consumes those primitives.

Pong SHOULD NOT duplicate them as independent authoritative state.

A Pong implementation MAY keep helper or orchestration state, but it MUST NOT override, replace, or contradict the authoritative Console and ConsoleStorage state used for:
- compliance
- readiness
- policy
- final Sign mutation

## Design goals

- Reusable across many Consoles
- Minimal logic surface
- No direct critical Sign storage authority
- Prefer clear separation between transfer-oriented orchestration and authoritative Console state
- Safe interaction with Console-native readiness and policy controls
- Compatibility with a Console that can still receive compliant transfers even when no Cartridge is connected
- Compatibility with the three-contract Apeiron architecture

## Non-goals

Pong does not standardize:
- collection minting
- ERC-721 compatibility
- ERC-1155 compatibility
- marketplace rules
- royalties
- auction logic
- validator logic
- metadata compression
- factory deployment

## Terminology

### push

A transfer flow initiated from the source side and delivered toward a destination Console.

### pull

A transfer flow initiated from the destination side against a Sign held by a remote source Console.

### source Console

The Console that currently governs the Sign and whose bound ConsoleStorage holds the critical Sign state.

### destination Console

The Console that will receive and recreate the Sign through its own bound ConsoleStorage.

### compliant path

A remote path in which:
1. remote Console codehash passes
2. remote ConsoleStorage codehash passes
3. remote active Cartridge codehash passes, if a Cartridge exists

## Scope of Pong

Pong defines:
- outgoing push transfer initiation
- incoming push helper flows
- incoming pull request initiation
- outgoing pull fulfillment
- recommended sequencing and revert behavior
- how transfer provenance should be emitted
- how Pong uses Console-native readiness, handshake, and policy state

Pong assumes:
- all critical Sign creation happens through Console-authorized calls into ConsoleStorage
- all critical Sign deletion happens through Console-authorized calls into ConsoleStorage
- compliant remote communication is Console-to-Console
- incoming reception happens through the destination Console `receiveTransfer(...)` primitive

## Core rule

Pong MUST NOT directly create, delete, or mutate critical Sign state in its own storage, in Console storage variables, or in ConsoleStorage.

All authoritative Sign state changes MUST happen through Console-defined paths.

## Recommended implementation profile

The RECOMMENDED Pong profile is:
- reusable by many Consoles
- as stateless as practical
- detached from direct Sign ownership
- consumed by Consoles as a transfer-oriented logic module
- dependent on the Console handshake and policy model

A Pong implementation MAY keep helper state if needed, but such state MUST NOT become authoritative over:
- compliance
- readiness
- critical Sign lifecycle
- final acceptance or rejection of incoming compliant transfers

## Transfer semantics

## Push model

In a push flow:
- the source-side local logic decides to send a Sign
- the source Console verifies outbound policy and remote compliance
- the source Console performs the compliant remote call
- the destination Console accepts or rejects through its incoming transfer path

Pong defines the orchestration semantics for this flow.

## Pull model

In a pull flow:
- the destination side requests a remote Sign
- the source side fulfills only if pull approval and outbound policy allow it
- the source Console performs the compliant remote call
- the destination Console accepts or rejects through its incoming transfer path

Pong defines the orchestration semantics for this flow.

## Compliance model in Pong

Pong MUST NOT redefine Apeiron compliance.

Pong SHOULD consume the canonical Console `txHandshake(...)` helper whenever it needs to determine whether a remote path is compliant.

A Pong implementation MAY implement its own compliance routine, but SHOULD NOT do so unless strictly necessary, since Apeiron Core defines the Console handshake as the canonical compliance path.

Pong MUST assume that compliant remote communication is:
- source Console -> destination Console

Pong SHOULD NOT be the direct authoritative remote caller in compliant flows.

## Policy model in Pong

Pong MUST respect Console-native policy.

### Inbound

Pong and the destination Console incoming transfer path MUST respect:
- `istate`
- `idelay`
- inbound allowlist
- inbound blocklist
- `pushWaitingList` when required by local rules

### Outbound

Pong MUST respect:
- `ostate`
- `odelay`
- outbound allowlist
- outbound blocklist
- `pullTokenPreApprovedList` when fulfilling pulls

### Precedence

Inbound blocklist MUST prevail over inbound allowlist.

Outbound blocklist MUST prevail over outbound allowlist.

Global Console policy MUST prevail where Apeiron Core defines broader lock semantics.

## Metadata behavior in Pong

Pong MUST preserve:
- `tokenId`
- `metadata`

Pong MUST NOT mutate Sign identity.

Pong MAY use:
- stored metadata
- resolved metadata
- implementation-defined local read paths

But the destination recreation MUST preserve Apeiron Core semantics.

## Normative flows

## Outgoing Push

### Purpose

The outgoing push flow sends a Sign from a source Console to a destination Console.

### Requirements

Before initiating a push flow, Pong SHOULD ensure that the source Console:
- verifies outbound policy
- executes the canonical handshake against the destination Console
- confirms that the destination path is compliant
- preserves `tokenId`
- preserves `metadata`
- routes destination reception through `destinationConsole.receiveTransfer(...)`

### Required behavior

A successful outgoing push flow MUST:
1. identify the local Sign by `key`
2. read the local `tokenId`
3. read stored or resolved metadata according to the chosen flow
4. rely on the source Console to verify outbound policy
5. rely on the source Console to execute the canonical handshake
6. emit initiation events
7. delete the local Sign only through the source Console / ConsoleStorage path
8. call the destination Console incoming transfer path from the source Console
9. rely on full transaction revert if the destination flow fails
10. emit completion events on success

### Failure conditions

An outgoing push flow MUST revert when:
- the local Sign does not exist
- outbound block policy denies the flow
- outbound state denies the flow
- remote Console compliance fails
- remote ConsoleStorage compliance fails
- remote active Cartridge compliance fails when a remote Cartridge exists
- local deletion fails
- destination reception fails

## Incoming Push Helper

### Purpose

Pong MAY expose helper logic for preparing or coordinating an incoming push.

This helper is not the authoritative destination acceptance mechanism.

The authoritative acceptance mechanism remains the destination Console incoming transfer path.

### Relationship to `pushWaitingList`

Pong MAY offer a helper alias for registering expected incoming push data, but the authoritative state MUST remain in the Console `pushWaitingList`.

## Incoming Pull Request

### Purpose

The pull request flow begins from the destination side, asking a remote source Console to release a specific Sign.

### Requirements

Before initiating a pull request, Pong SHOULD ensure that:
- destination readiness is checked locally
- the expected remote path is verified through the source or destination Console handshake
- waiting expectations are registered locally when desired

### Required behavior

A pull request flow SHOULD:
1. identify the remote Sign to be requested
2. prepare local waiting expectations when desired
3. route the request to the remote source-side Pong fulfillment logic
4. rely on the source Console for pull approval checks, outbound policy, and remote compliant delivery

## Outgoing Pull Fulfillment

### Purpose

The source side fulfills a valid pull request for a local Sign.

### Requirements

A pull fulfillment flow MUST require that:
- the local Sign exists
- the Sign is eligible under `pullTokenPreApprovedList`
- the requesting remote Console is authorized by local pull pre-approval rules
- outbound policy permits the flow
- the source Console canonical handshake confirms that the destination path is compliant

### Required behavior

A successful outgoing pull fulfillment MUST:
1. identify the local Sign
2. verify pull pre-approval through the source Console
3. verify outbound policy through the source Console
4. verify remote compliance through the source Console handshake
5. emit initiation events
6. delete the local Sign only through the source Console / ConsoleStorage path
7. call the destination Console incoming transfer path from the source Console
8. rely on full transaction revert if the destination flow fails
9. emit completion events on success

### Failure conditions

A pull fulfillment flow MUST revert when:
- the local Sign does not exist
- no pull pre-approval exists
- outbound block policy denies the flow
- outbound state denies the flow
- remote Console compliance fails
- remote ConsoleStorage compliance fails
- remote active Cartridge compliance fails when a remote Cartridge exists
- local deletion fails
- destination reception fails

## Event expectations

Pong implementations SHOULD emit cartridge-level orchestration events in addition to Console lifecycle events.

Suggested Pong events:
- `PongPushInitiated`
- `PongPushDelivered`
- `PongPullRequested`
- `PongPullFulfilled`
- `PongTransferRejected`

The authoritative Sign lifecycle remains represented by:
- Console events
- ConsoleStorage lifecycle events

## Security considerations

- Pong MUST NOT directly call or mutate ConsoleStorage
- Pong MUST NOT rely on fallback-driven mutation
- Pong MUST NOT bypass Console policy
- Pong SHOULD rely on the Console `txHandshake(...)` helper for compliance verification
- Reimplementing Apeiron compliance inside Pong SHOULD be treated as high risk
- Pong SHOULD remain as stateless as practical
- Pong flows SHOULD delete the local Sign before the remote external call
- Pong flows MUST rely on transaction-wide revert if remote reception fails
- Pong MUST NOT become a hidden substitute for Console-native policy, compliance, or critical state authority

## Reference interface suggestion

The following function families are RECOMMENDED for a Pong reference implementation:
- outgoing push
- incoming push helper / waiting helper
- incoming pull request
- outgoing pull fulfillment

Exact function names MAY vary, but the semantics defined in this document SHOULD be preserved.

## Reference implementation note

A reference Pong implementation SHOULD:
- be reusable by many Consoles
- keep no authoritative Sign state
- integrate only through Apeiron Console interfaces
- rely on ConsoleStorage only indirectly through Console APIs
- include tests for push and pull flows
- include tests for waiting-list use
- include tests for pull pre-approval use
- include tests for inbound/outbound block precedence
- include tests for full revert on failed destination reception
- include tests for 2-way and 3-way compliance paths