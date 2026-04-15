# PONG.md

## Apeiron Protocol Cartridge "Pong"

## Summary

Pong is the first reference Cartridge for Apeiron Protocol Core. It standardizes transfer-oriented Sign flows, including push and pull semantics, while relying on Apeiron Console primitives for storage, readiness, compliance, and policy enforcement.

Pong does not own Sign state and MUST NOT directly modify Console storage.

## Purpose

Pong exists to provide a concrete, reusable, and auditable transfer module for Apeiron Consoles.

Pong is responsible for:
- transfer orchestration
- push flows
- pull flows
- invoking the proper Console actions
- preserving Apeiron transfer semantics

Pong is not responsible for:
- owning Sign state
- directly mutating Sign storage
- replacing Console policy
- replacing Console compliance checks
- redefining Console waiting-list or approval state

## Relationship to Apeiron Core

Apeiron Core already defines and requires the following Console-native primitives:
- `receiveTransfer`
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
- EXTCODEHASH-based compliance
- metadata storage and resolution behavior

Pong consumes those primitives.

Pong MUST NOT duplicate them as independent authoritative state.

## Design goals

- Reusable across many Consoles
- Minimal logic surface
- No direct Sign storage authority
- Clean separation between transfer orchestration and Console state
- Safe interaction with Console-native readiness and policy controls
- Compatibility with a Console that can still receive compliant transfers even when no Cartridge is connected

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

A transfer flow initiated by the source side and sent toward a destination Console.

### pull

A transfer flow initiated by the destination side against an eligible Sign at a remote Console.

### source Console

The Console that currently holds the Sign.

### destination Console

The Console that will receive the recreated Sign.

### compliant path

A transfer path in which required EXTCODEHASH-based checks pass according to local Apeiron policy.

## Scope of PONG

Pong defines:
- outgoing push transfer initiation
- incoming push helper flows
- incoming pull request initiation
- outgoing pull fulfillment
- recommended sequencing and revert behavior
- how transfer provenance should be emitted
- how Pong uses Console-native readiness and policy state

Pong assumes:
- all Sign creation happens through Console functions
- all Sign deletion happens through Console functions
- incoming reception happens through the Console `receiveTransfer` primitive or an equivalent Console-native reception path defined by Apeiron Core

## Core rule

Pong MUST NOT directly create, delete, or mutate Sign state in its own storage or in Console storage.

All such changes MUST happen through Console functions.

## Recommended implementation profile

The RECOMMENDED Pong profile is:
- storage-less or near-storage-less
- reusable by many Consoles
- disconnected from Sign ownership semantics beyond allowed orchestration
- callable only through an active Cartridge relationship when required by the Console

A Pong implementation MAY keep minimal helper state if needed by a particular implementation, but such state MUST NOT replace or shadow the authoritative Console state.

## Transfer semantics

### Push model

In a push flow, the source side initiates the transfer and attempts to deliver the Sign to the destination Console.

The destination Console receives the transfer through its always-available compliant incoming transfer path.

### Pull model

In a pull flow, the destination side initiates a request against a Sign held at a source Console.

The source side fulfills the request only if:
- the Sign is eligible in `pullTokenPreApprovedList`
- outbound policy allows the flow
- required EXTCODEHASH-based compliance checks succeed

## Normative flows

## Outgoing Push

### Function purpose

The outgoing push flow sends a Sign from a source Console to a destination Console.

### Requirements

Before initiating a push flow, the implementation MUST:
- verify outbound policy on the source Console
- verify that the destination path is compliant under EXTCODEHASH-based local rules
- preserve `tokenId`
- preserve `metadata`
- route destination reception through the destination Console incoming transfer primitive

### Required behavior

A successful outgoing push flow MUST:
1. identify the local Sign by `key`
2. read the Sign `tokenId`
3. read or resolve the Sign metadata according to implementation needs
4. verify outbound policy
5. verify remote compliance
6. emit transfer initiation events
7. delete the local Sign only through the source Console
8. call the destination Console incoming transfer path
9. rely on full transaction revert if the destination flow fails
10. emit transfer completion events on success

### Failure conditions

An outgoing push flow MUST revert when:
- the local Sign does not exist
- outbound block policy denies the flow
- outbound state denies the flow
- remote Console compliance fails
- remote active Cartridge compliance fails when relevant to the flow
- local deletion fails
- destination reception fails

## Incoming Push Helper

### Function purpose

Pong MAY expose a helper for preparing or coordinating an incoming push.

This helper is not the authoritative destination acceptance mechanism. The authoritative acceptance mechanism remains the destination Console incoming transfer path.

### Relationship to `pushWaitingList`

Pong MAY offer a helper alias for registering expected incoming push data, but the authoritative state MUST remain in the Console `pushWaitingList`.

## Incoming Pull Request

### Function purpose

The pull request flow begins from the destination side, asking a remote source Console to release a specific Sign.

### Requirements

Before initiating a pull request, the implementation SHOULD:
- verify destination readiness
- verify that the expected remote path is compliant
- optionally register waiting expectations locally

### Required behavior

A pull request flow SHOULD:
1. identify the remote Sign to be requested
2. prepare local waiting expectations when desired
3. route the request to the remote source-side Pong fulfillment flow
4. rely on remote-side approval and outbound policy checks

## Outgoing Pull Fulfillment

### Function purpose

The source side fulfills a valid pull request for a local Sign.

### Requirements

A pull fulfillment flow MUST require that:
- the local Sign exists
- the Sign is eligible under `pullTokenPreApprovedList`
- the requesting remote Console is authorized by the local pull pre-approval rules
- outbound policy permits the flow
- EXTCODEHASH-based compliance checks pass for the destination path

### Required behavior

A successful outgoing pull fulfillment MUST:
1. identify the local Sign
2. verify pull pre-approval
3. verify outbound policy
4. verify remote compliance
5. emit transfer initiation events
6. delete the local Sign only through the source Console
7. call the destination Console incoming transfer path
8. rely on full transaction revert if the destination flow fails
9. emit transfer completion events on success

### Failure conditions

A pull fulfillment flow MUST revert when:
- the local Sign does not exist
- no pull pre-approval exists
- outbound block policy denies the flow
- outbound state denies the flow
- remote compliance fails
- local deletion fails
- destination reception fails

## Compliance rules in Pong

Pong MUST NOT redefine Apeiron compliance.

Pong MUST rely on Apeiron Core EXTCODEHASH-based compliance.

When a Pong flow depends on a remote Console path, the implementation MUST:
- inspect the remote Console runtime code hash
- verify it against the local Console allowlist
- inspect the remote active Cartridge runtime code hash when relevant to the flow
- verify it against the local Cartridge allowlist

The use of remote addresses in Pong is only for:
- identifying the endpoint to inspect
- routing the transfer
- applying Console-native inbound/outbound or waiting-list policy

Compliance itself remains determined exclusively by runtime code hash allowlists.

## Policy rules in Pong

Pong MUST respect Console-native transfer policy.

### Inbound

Pong and the destination Console incoming reception path MUST respect:
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
- local implementation-defined metadata read paths

But the destination recreation MUST preserve Apeiron Core semantics.

## Event expectations

Pong implementations SHOULD emit cartridge-level orchestration events in addition to Console lifecycle events.

Suggested Pong events:
- `PongPushInitiated`
- `PongPushRequested`
- `PongPushDelivered`
- `PongPullRequested`
- `PongPullFulfilled`
- `PongTransferRejected`

The authoritative Sign lifecycle remains represented by Console events.

## Security considerations

- Pong MUST NOT directly mutate Console storage
- Pong MUST NOT rely on fallback-driven mutation
- Pong MUST NOT bypass Console policy
- Pong MUST treat remote compliance failure as fatal to the flow
- Pong SHOULD remain as stateless as practical
- Pong flows SHOULD delete the local Sign before the remote external call
- Pong flows MUST rely on transaction-wide revert if remote reception fails
- Pong MUST NOT become a hidden substitute for Console-native policy or compliance logic

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
- include tests for push and pull flows
- include tests for waiting-list use
- include tests for pull pre-approval use
- include tests for inbound/outbound block precedence
- include tests for full revert on failed destination reception