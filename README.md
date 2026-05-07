# Apeiron Protocol

Apeiron Protocol is a token architecture for onchain representation built around three components:

- **Console**: immutable policy and execution contract
- **ConsoleStorage**: isolated contract holding critical Sign state
- **Cartridge**: optional execution module connected to a Console

Its token unit is called a **Sign**.

Apeiron is currently being specified as an ERC-style standard together with companion cartridge specifications and reference implementations.

## Current status

This repository is under active design and standardization.

The current canonical draft files are:

- `apeiron-core/eip/erc-apeiron-core.md`
- `apeiron-core/eip/PONG.md`

Historical and archived materials are kept separately and are not the canonical specification.

## Core concepts

### Sign

A Sign is the Apeiron token unit.

Each Sign has:
- `tokenId`: immutable representation anchor
- `key`: immutable local identifier
- `metadata`: mutable descriptive data

Global uniqueness is determined by:

`(chainId, consoleAddress, key)`

Multiple Signs may share the same `tokenId`.

### Console

The Console is the main contract controlled by the user or entity.

It is responsible for:
- policy
- compliance
- recovery
- locks and delays
- transfer-readiness
- interaction with ConsoleStorage
- optional Cartridge connection

### ConsoleStorage

ConsoleStorage holds the critical Sign state and only accepts restricted calls from its bound Console.

This separation exists so that Cartridges can add logic without receiving direct authority over critical storage.

### Cartridge

A Cartridge is an optional execution module connected to a Console.

Cartridges may define local execution logic and orchestration, but must follow Apeiron compliance and state authority rules.

## Compliance model

Apeiron compliance is based on **EXTCODEHASH** allowlists.

In compliant remote interactions, verification is based on:
1. remote Console codehash
2. remote ConsoleStorage codehash
3. remote active Cartridge codehash, if a Cartridge exists

Compliance is not based on addresses, even though addresses are still used as endpoints for routing and local policy.

## Repository structure

```text
apeiron-core/
  eip/
    erc-apeiron-core.md
    PONG.md
    RATIONALE.md   # planned / optional companion rationale
archive/
  ... historical drafts and old notes
README.md
```

## Main documents

### `erc-apeiron-core.md`

The formal core Apeiron draft.

It defines:
- Console
- ConsoleStorage
- Cartridge
- Sign model
- compliance
- policy controls
- recovery
- metadata behavior
- canonical interfaces

### `PONG.md`

The first reference Cartridge specification.

It defines:
- push and pull transfer-oriented behavior
- orchestration rules
- interaction with Apeiron Core primitives

## Design goals

- protect critical Sign storage
- allow modular execution
- preserve explicit trust boundaries
- support advanced policy and recovery rules
- keep transfer-readiness available even without a Cartridge connected
- support reusable cartridge logic

## Notes

This repository contains evolving drafts.

Until the ERC and reference implementations stabilize:
- wording may change
- interfaces may be refined
- companion specifications may be added

## License

- ERC/specification text: CC0
- reference implementation code: MIT