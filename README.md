# Radicle Architectural Blueprints

**Project:** Radicle Architecture Documentation  
**Purpose:** High-Rigor Architectural Manifest for Understanding & Reproducing Radicle Heartwood  
**Version:** 1.0.0  
**Generated:** 2026-02-26

---

## Overview

This repository contains comprehensive architectural documentation for **Radicle Heartwood**, the P2P code collaboration stack. Documentation is reverse-engineered from the actual source code.

**The Goal:** Create architectural blueprints so complete that reproducing or modifying Radicle becomes a mechanical process, not an archaeological one.

---

## Document Hierarchy

```
radicle-architecture/
├── README.md                          ← You are here
├── ARCHITECTURE.md                   ← System overview
├── CRATES.md                         ← Crate reference
├── IDENTITY.md                       ← Cryptographic identity system
├── PROTOCOL.md                       ← P2P protocol
├── NODE.md                           ← Node daemon architecture
├── COB.md                           ← Collaborative Objects
├── CLI.md                           ← Command-line interface
└── diagrams/                        ← Architecture diagrams
```

---

## Quick Start

### Reading Order (Recommended)

1. **ARCHITECTURE.md** — Understand the system as a whole
2. **CRATES.md** — Crate hierarchy and dependencies
3. **IDENTITY.md** — Cryptographic identity foundation
4. **PROTOCOL.md** — P2P gossip protocol
5. **NODE.md** — Daemon architecture
6. **COB.md** — Collaborative objects (issues, patches)
7. **CLI.md** — Command structure

### For Reproduction

1. Read **ARCHITECTURE.md** for system overview
2. Study **PROTOCOL.md** for networking
3. Reference **CRATES.md** for implementation details
4. Use **IDENTITY.md** for cryptographic foundation

---

## Core Concepts

### Heartwood Stack

Radicle Heartwood is the third iteration of the Radicle Protocol:

- **Identity** — Cryptographic keys (Ed25519), not email
- **Networking** — P2P gossip protocol, no central server
- **Collaboration** — CRDTs for issues, patches, reviews
- **Storage** — Git + custom objects

### Key Differences from Other Systems

| Aspect | GitHub | GitLab | Radicle |
|--------|--------|--------|---------|
| Central Server | Yes | Yes | No (P2P) |
| Identity | Email | Email | Cryptographic |
| Forking | Permissionless | Permissionless | Permissionless |
| Issues | Yes | Yes | Yes (Cob) |
| Discovery | Search | Search | Follow |

### Comparison to Related Systems

| Aspect | LangGraph | OpenClaw | Radicle |
|--------|-----------|----------|---------|
| **Model** | Pregel supersteps | Event-driven | Gossip P2P |
| **State** | Channels + reducers | Multi-layer memory | CRDTs |
| **Identity** | None | WE/witness | Cryptographic keys |
| **Persistence** | Checkpoints | Session-memory | Replication |
| **Language** | Python | TypeScript/JS | Rust |

---

## Version Info

| Radicle Version | Architecture Version | Status |
|-----------------|---------------------|--------|
| Heartwood (latest) | 1.0.0 | In Progress |

---

## Contributing

This is a **living document**. As Radicle evolves, this repository should be updated to reflect architectural changes.

### Update Process

1. Review Radicle source changes
2. Update relevant documentation
3. Update version numbers
4. Commit and push to all mirrors

---

## Mirrors

This documentation is published to:

- GitHub: `mrhavens/radicle-architecture`
- Forgejo: `mrhavens/radicle-architecture`
- GitLab: `mrhavens/radicle-architecture`

---

## Related Work

### OpenClaw Architecture

See: `mrhavens/openclaw-architecture`

OpenClaw is the AI companion framework that runs Solaria. Provides contrast to Radicle's decentralized approach.

### LangGraph Architecture

See: `mrhavens/langgraph-architecture`

LangGraph is a low-level orchestration framework for multi-agent systems. Provides comparison for distributed computing models.

---

## Radicle Heartwood Source

The source code is available at:

- Fork: `mrhavens/heartwood` (our fork)
- Original: `radicle-dev/heartwood`

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
