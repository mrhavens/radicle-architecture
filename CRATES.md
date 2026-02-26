# Radicle Crate Reference

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

Radicle Heartwood consists of 21 Rust crates. This document catalogs each crate and its purpose.

---

## Core Crates

### radicle

The Radicle standard library — shared types and interfaces used across the project.

**Location:** `crates/radicle/src`

**Key Types:**
- `RadUrl` — Radicle URL scheme (`rad://`)
- `Urn` — Uniform Resource Name for identity
- `Identity` — Project or user identity
- `Remote` — Remote peer reference

**Purpose:** Foundation that all other crates depend on.

---

### radicle-crypto

Cryptographic primitives for identity and signing.

**Location:** `crates/radicle-crypto/src`

**Key Types:**
- `PublicKey` — Ed25519 public key
- `SecretKey` — Ed25519 secret key  
- `Signer` — Trait for signing operations
- ` encrypt` — X25519 encryption

**Purpose:** Identity foundation. All Radicle identity is based on Ed25519.

**Security:** Pure Rust — no external crypto dependencies.

---

### radicle-oid

Object ID handling for Git objects.

**Location:** `crates/radicle-oid/src`

**Key Types:**
- `Oid` — Git object ID (SHA1)
- `ObjectId` — Generic object reference

**Purpose:** Git interoperability.

---

## Git Integration

### radicle-git-metadata

Handling Git metadata in Radicle contexts.

**Location:** `crates/radicle-git-metadata/src`

**Purpose:** Extract and process Git metadata (commits, trees, blobs).

---

### radicle-git-ref-format

Reference format validation and handling.

**Location:** `crates/radicle-git-ref-format/src`

**Purpose:** Validate Git ref names (`refs/heads/`, `refs/tags/`, etc.).

---

### radicle-remote-helper

Git remote helper for `rad://` protocol.

**Location:** `crates/radicle-remote-helper/src`

**Purpose:** Allows Git to interact with Radicle repositories using standard Git commands.

**Protocol:** 
- Registered as Git protocol helper
- Handles `rad://` URL scheme
- Translates Git operations to Radicle operations

---

## Networking

### radicle-node

The P2P daemon that runs the Radicle network.

**Location:** `crates/radicle-node/src`

**Key Files:**
- `lib.rs` — Main entry point
- `runtime.rs` — Async runtime
- `reactor.rs` — Event reactor
- `worker.rs` — Background worker tasks

**Purpose:** Core networking daemon. Runs continuously, connects to peers, replicates repos.

**Configuration:**
```json
{
  "node": {
    "alias": "my-node",
    "listen": [],
    "peers": { "type": "dynamic" },
    "relay": "auto"
  }
}
```

---

### radicle-protocol

The P2P protocol implementation.

**Location:** `crates/radicle-protocol/src`

**Key Files:**
- `service.rs` — Protocol state machine
- `wire.rs` — Wire format (binary encoding)
- `fetcher.rs` — Object fetching logic

**Message Types:**
- `message::Announce` — New repository revision
- `message::Gossip` — Information dissemination
- `message::Request` — Object request
- `message::Response` — Object response

**Purpose:** The gossip protocol that powers Radicle networking.

---

### radicle-fetch

Fetching logic for retrieving repository data.

**Location:** `crates/radicle-fetch/src`

**Purpose:** Efficient object transfer between peers.

---

### radicle-ssh

SSH functionality for remote connections.

**Location:** `crates/radicle-ssh/src`

**Purpose:** SSH key handling, `ssh-agent` integration.

---

## Collaboration (Cobs)

### radicle-cob

Collaborative Objects — issues, patches, reviews.

**Location:** `crates/radicle-cob/src`

**Key Types:**
- `Issue` — Issue tracking
- `Patch` — Proposed changes
- `Review` — Patch review
- `Timeline` — Object history

**Purpose:** Decentralized collaboration. All COBs are stored as Git objects.

**Storage Format:**
```
refs/heads/rad/issue/1      → Issue #1
refs/heads/rad/patch/1     → Patch #1
refs/heads/rad/review/1    → Review #1
```

---

### radicle-dag

Directed Acyclic Graph for tracking object histories.

**Location:** `crates/radicle-dag/src`

**Purpose:** Efficient traversal of commit graphs, merge detection.

---

### radicle-signals

Signal handling for the node.

**Location:** `crates/radicle-signals/src`

**Purpose:** Graceful shutdown, health checks.

---

## CLI

### radicle-cli

The `rad` command-line interface.

**Location:** `crates/radicle-cli/src`

**Commands:**
```
rad auth        — Authenticate/create identity
rad init        — Initialize repository
rad push        — Push to network
rad pull        — Pull from network
rad clone       — Clone repository
rad checkout    — Switch branches
rad branch      — Branch management
rad issue       — Issue management
rad patch       — Patch management
rad remote      — Remote management
rad self        — Show identity
rad peers       — List connected peers
```

**Purpose:** User-facing CLI for all Radicle operations.

---

### radicle-cli-test

Testing framework for CLI documentation.

**Location:** `crates/radicle-cli-test/src`

**Purpose:** Write documentation as tests.

---

### radicle-term

Terminal UI components.

**Location:** `crates/radicle-term/src`

**Purpose:** Reusable TUI components for CLI.

---

## Utility Crates

### radicle-schemars

JSON schema generation.

**Purpose:** Generate schemas for serialization.

---

### radicle-localtime

Local time handling.

**Purpose:** Time zone aware timestamps.

---

### radicle-systemd

Systemd integration.

**Location:** `crates/radicle-systemd/src`

**Purpose:** systemd service files and helpers.

---

### radicle-windows

Windows-specific code.

**Location:** `crates/radicle-windows/src`

**Purpose:** Windows compatibility.

---

### radicle

The meta-crate that re-exports everything.

**Location:** `crates/radicle/src/lib.rs`

**Purpose:** Convenience import for the entire Radicle stack.

---

## Crate Dependency Graph

```
                    ┌──────────────────┐
                    │   radicle-crypto  │ ← Identity foundation
                    └────────┬─────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
┌────────────────┐  ┌──────────────┐  ┌──────────────┐
│radicle-git-*   │  │  radicle     │  │ radicle-oid  │
│Git integration │  │  Standard lib │  │  Object IDs  │
└────────┬───────┘  └───────┬──────┘  └──────────────┘
         │                  │
         │         ┌────────┴────────┐
         │         │                 │
         ▼         ▼                 ▼
┌────────────────┐  ┌──────────────┐  ┌──────────────┐
│remote-helper  │  │radicle-node  │  │ radicle-cob  │
│Git protocol   │  │ P2P daemon  │  │ Collaboration│
└───────────────┘  └──────┬───────┘  └──────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
    ┌────────────────┐     ┌────────────────┐
    │radicle-protocol│     │radicle-fetch  │
    │ P2P gossip     │     │ Fetching      │
    └────────────────┘     └────────────────┘
              │
              ▼
    ┌────────────────┐
    │ radicle-cli    │
    │ User interface │
    └────────────────┘
```

---

## Further Reading

- Source: `https://github.com/mrhavens/heartwood`
- Docs: `https://docs.radicle.xyz`

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
