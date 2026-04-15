# Pong Cartridge Specification

## Purpose

Pong is the first reference Apeiron Cartridge for transfer-oriented Sign flows.

Pong standardizes:
- push-based outgoing transfer flows
- push-based incoming finalization flows
- pull request flows
- pull execution flows
- how Cartridge orchestration interacts with Console-native waiting lists, pull pre-approvals, and inbound/outbound policy controls

Pong does not own Sign storage and MUST NOT directly mutate Console storage.

## Scope

Pong depends on Apeiron Protocol Core.

All Sign creation, deletion, and metadata mutation remain Console-governed.

Pong only orchestrates valid transfer flows by calling Console functions and compliance entrypoints.

## Relationship to Apeiron Core

The following transfer-readiness and policy primitives are defined by Apeiron Core and consumed by Pong:
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
- whitelist-based compliance checks
- `txFunction` or equivalent forwarding/compliance entrypoint
- receive/finalize entrypoint

Pong does not redefine those primitives as independent state.

## Terminology

### push

A transfer flow initiated by the source side toward a destination Console.

### pull

A transfer flow initiated by the destination side against an eligible Sign at a remote Console.

### push_i_token

Incoming push finalization flow.

### push_o_token

Outgoing push initiation flow.

### pull_i_token

Incoming pull request flow against a remote Console.

### pull_o_token

Outgoing fulfillment of a previously authorized pull flow.

## Normative Behavior

### push_o_token

The source side initiates a push-oriented transfer to a destination Console.

The Pong implementation MUST:
1. verify outbound policy on the source Console
2. verify whitelist-based compliance for the destination Console and, when relevant, its active Cartridge
3. call the appropriate Console compliance/forwarding entrypoint
4. cause source-side deletion only through a Console function
5. cause destination-side creation only through a Console function
6. preserve `tokenId` and `metadata`
7. rely on full revert if the destination-side flow fails

### push_i_token

The destination side finalizes an expected incoming push.

The Pong implementation MUST require that at least one of the following is satisfied:
- inbound state is unlocked
- the remote Console is allowed by inbound allowlist policy
- the incoming transfer matches an entry in `pushWaitingList`

Inbound blocklist MUST override inbound allowlist and waiting expectations where applicable by Console policy.

### pull_i_token

The destination side requests a pull from a remote Console.

The Pong implementation MUST:
1. identify the requested remote Sign
2. verify local readiness for the expected incoming Sign
3. route the request through the compliant remote flow
4. rely on remote-side pre-approval checks

### pull_o_token

The source side fulfills a pull request.

The Pong implementation MUST require that:
- the Sign is eligible in `pullTokenPreApprovedList`
- the requesting remote Console, remote address, or wildcard policy is authorized
- outbound policy permits the flow
- remote compliance requirements pass

The source-side Sign MUST be deleted only through a Console function.

The destination-side Sign MUST be created only through a Console function.

## Helper Flows

### tokenWaiting

A helper flow or UX alias that writes expected incoming transfer information into the Console `pushWaitingList`.

Normatively, the state belongs to the Console core.

### tokenUnlock

A helper flow or UX alias that writes pull eligibility into the Console `pullTokenPreApprovedList`.

Normatively, the state belongs to the Console core.

## Policy Interactions

### Inbound controls

Pong MUST respect:
- `istate`
- `idelay`
- inbound allowlist
- inbound blocklist

### Outbound controls

Pong MUST respect:
- `ostate`
- `odelay`
- outbound allowlist
- outbound blocklist

### Precedence

Inbound blocklist MUST prevail over inbound allowlist.

Outbound blocklist MUST prevail over outbound allowlist.

Global Console policy MUST prevail where the core standard defines a broader lock condition.

## Events

Suggested Pong-specific event family:
- PongPushInitiated
- PongPushFinalized
- PongPullRequested
- PongPullFulfilled
- PongTransferRejected

Core transfer provenance events SHOULD also be emitted by the Console-side lifecycle.

## Failure Conditions

Pong flows MUST revert when:
- source outbound policy denies the flow
- destination inbound policy denies the flow
- waiting-list expectations do not match when required
- pull pre-approval does not exist
- Console whitelist-based compliance fails
- relevant Cartridge whitelist-based compliance fails
- source deletion fails
- destination creation fails

## Ordering and Atomicity

Pong reference implementations SHOULD:
1. perform all local and remote compliance checks
2. delete the local Sign through the Console
3. perform the remote external call
4. recreate the Sign remotely through the destination Console
5. rely on transaction-wide revert if the remote flow fails

## Security Considerations

- Pong MUST NOT directly mutate Console storage
- Pong MUST NOT rely on fallback-driven mutation
- Pong MUST NOT bypass core Console policy
- Pong implementers MUST treat active Cartridge changes as high-risk
- Pong flows MUST be reentrancy-aware and revert atomically on failure

## Reference Implementation Notes

A reference Pong implementation SHOULD:
- integrate only through Apeiron Console interfaces
- include Foundry tests
- include at least one working push flow
- include at least one working pull flow
- demonstrate waiting-list and pre-approval usage
- demonstrate inbound/outbound block precedence