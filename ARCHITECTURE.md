# Radicle Heartwood Architecture Overview

**Version:** 1.0.0  
**Radicle Version:** Heartwood (latest)  
**Last Updated:** 2026-02-26

---

## Executive Summary

Radicle Heartwood is a peer-to-peer code collaboration stack built on Git. It provides decentralized hosting, cryptographic identity, and collaborative objects without relying on central servers.

This document provides the architectural blueprint for understanding, reproducing, or modifying Radicle.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RADICLE HEARTWOOD ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                           CLI LAYER                                     │
├─────────────────────────────────────────────────────────────────────────┤
│  rad CLI (radicle-cli)                                                │
│  ├── rad auth     — Identity management                                │
│  ├── rad init     — Initialize repository                              │
│  ├── rad push    — Push to network                                   │
│  ├── rad pull    — Pull from network                                  │
│  ├── rad issue   — Create/manage issues (COBs)                       │
│  ├── rad patch   — Create/manage patches                               │
│  └── rad remote — Manage remotes                                      │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      GIT REMOTE HELPER                                  │
├─────────────────────────────────────────────────────────────────────────┤
│  radicle-remote-helper                                                │
│  Handles `rad://` protocol scheme                                      │
│  Translates Git operations to Radicle protocol                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         NODE LAYER                                      │
├─────────────────────────────────────────────────────────────────────────┤
│  radicle-node daemon                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ Runtime (async)                                                  │  │
│  │  ├── Peer management                                             │  │
│  │  ├── Gossip protocol                                             │  │
│  │  ├── Replication engine                                          │  │
│  │  └── Storage manager                                             │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ Protocol Service (state machine)                                 │  │
│  │  ├── Initial handshake                                           │  │
│  │  ├── Announcement                                                │  │
│  │  ├── Dissemination                                              │  │
│  │  └── Replication                                                 │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       PROTOCOL LAYER                                    │
├─────────────────────────────────────────────────────────────────────────┤
│  radicle-protocol                                                      │
│                                                                         │
│  Wire Format: Custom binary protocol                                   │
│  Transport: TCP + Tor (optional)                                      │
│                                                                         │
│  Message Types:                                                        │
│  ├── Announcement — "I have this repo at this revision"              │
│  ├── Gossip — "Here's what I've heard"                                │
│  ├── Request — "Give me this object"                                 │
│  └── Response — "Here's the object"                                   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        IDENTITY LAYER                                   │
├─────────────────────────────────────────────────────────────────────────┤
│  radicle-crypto                                                        │
│                                                                         │
│  Key Types:                                                            │
│  ├── Node ID — Ed25519 public key                                    │
│  ├── Signing Key — Ed25519 (can be delegated)                         │
│  └── Encryption Key — X25519 (for sealed boxes)                      │
│                                                                         │
│  Identity Format: rad:z<base32-encoded-key>                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      STORAGE LAYER                                      │
├─────────────────────────────────────────────────────────────────────────┤
│  Git + Radicle Extensions                                              │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ Standard Git Objects                                            │  │
│  │  ├── Commit — Versioned history                                  │  │
│  │  ├── Tree — File listings                                       │  │
│  │  ├── Blob — File contents                                       │  │
│  │  └── Tag — Annotated tags                                      │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ Radicle-Native Objects                                          │  │
│  │  ├── Rad — Project identity                                      │  │
│  │  ├── Delegate — Key delegation                                   │  │
│  │  ├── Issue — Collaborative object                                │  │
│  │  ├── Patch — Proposed change                                     │  │
│  │  └── Review — Patch review                                      │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Crate Architecture

### Core Crates

| Crate | Purpose |
|-------|---------|
| `radicle` | Standard library, shared types |
| `radicle-crypto` | Ed25519 signing, X25519 encryption |
| `radicle-oid` | Object ID handling |

### Git Integration

| Crate | Purpose |
|-------|---------|
| `radicle-git-metadata` | Git metadata handling |
| `radicle-git-ref-format` | Reference format |
| `radicle-remote-helper` | Git remote helper for `rad://` |

### Networking

| Crate | Purpose |
|-------|---------|
| `radicle-node` | P2P daemon |
| `radicle-protocol` | Protocol messages and state machine |
| `radicle-fetch` | Fetching logic |
| `radicle-ssh` | SSH functionality |

### Collaboration

| Crate | Purpose |
|-------|---------|
| `radicle-cob` | Collaborative Objects (issues, patches) |
| `radicle-dag` | Directed acyclic graph for histories |
| `radicle-signals` | Signal handling |

---

## Key Concepts

### 1. Identity (URN)

Radicle uses cryptographic identities:

```
rad:z3hgcXBHBhc6Lie6R1UKiL4jSpnG1nqLZkYzLG5N2e1M
└─────────────────────────────────────────────────────┘
         │
         └── Base32-encoded Ed25519 public key
```

### 2. Repository (RID)

Repositories have Radicle IDs:

```
rad:z3hgcXBHBhc6Lie6R1UKiL4jSpnG1nqLZkYzLG5N2e1M/my-project
└─────────────────────────────────────────────────────┘
         │
         └── Project namespace (owner's URN)
```

### 3. Gossip Protocol

1. **Announce** — Node announces repository at specific revision
2. **Gossip** — Peers propagate announcements
3. **Request** — Node requests missing objects
4. **Response** — Peer sends objects

### 4. Collaborative Objects (Cobs)

Issues, patches, and reviews are stored as Git objects:

```
refs/heads/rad/issue/1  → Issue #1
refs/heads/rad/patch/1  → Patch #1
```

---

## Data Flow

### Push to Network

```
User (rad push)
    │
    ▼
Git Remote Helper (rad://)
    │
    ▼
Radicle CLI (validates, signs)
    │
    ▼
Radicle Node (announces to peers)
    │
    ▼
Gossip Protocol (propagates)
```

### Clone from Network

```
User (git clone rad://...)
    │
    ▼
Git Remote Helper
    │
    ▼
Radicle Node (fetches via protocol)
    │
    ▼
Gossip (locates peers with repo)
    │
    ▼
Fetch (retrieves objects)
    │
    ▼
Git (assembles repository)
```

---

## Security Model

### Trust

- Identity is cryptographic — tied to key pair
- Changes are signed
- Verifiable authorship

### Verification

```bash
# Verify commit signature
git verify-commit <commit>

# View signing key
rad self
```

---

## Comparison to Centralized Systems

| Feature | GitHub | Radicle |
|---------|--------|---------|
| Hosting | Centralized server | P2P |
| Identity | Email + password | Cryptographic key |
| Discovery | Search | Follow users |
| Forking | Server-side copy | Local fork |
| Issues | Database | Git objects |
| Access Control | Repo settings | Key delegation |

---

## Further Reading

- [Protocol Guide](https://radicle.xyz/guides/protocol)
- [Heartwood Source](https://github.com/radicle-dev/heartwood)
- [Radicle CLI Docs](https://docs.radicle.xyz)

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
