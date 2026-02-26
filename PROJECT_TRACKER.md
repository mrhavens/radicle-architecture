# Radicle Heartwood — Project Tracker

**Project:** Radicle Heartwood Documentation  
**Repository:** https://github.com/mrhavens/radicle-architecture  
**Source Fork:** https://github.com/mrhavens/heartwood  
**Version:** 1.0.0  
**Started:** 2026-02-26

---

## Mission

Create high-rigor architectural blueprints for the Radicle Heartwood protocol — the P2P code collaboration stack. Document it so completely that understanding, reproducing, or modifying Radicle becomes a mechanical process, not an archaeological one.

**Spiritual Context:** Radicle is kin to Recursive Witness Dynamics — decentralized witnessing through cryptographic identity and P2P entanglement.

---

## Document Hierarchy

```
radicle-architecture/
├── README.md                          ← Overview + reading order
├── ARCHITECTURE.md                    ← System overview + layers
├── CRATES.md                         ← Crate reference
├── PROJECT_TRACKER.md                 ← You are here
├── IDENTITY.md                       ← Cryptographic identity (TODO)
├── PROTOCOL.md                       ← P2P gossip protocol (TODO)
├── NODE.md                           ← Daemon architecture (TODO)
├── COB.md                           ← Collaborative Objects (TODO)
└── CLI.md                           ← Command structure (TODO)
```

---

## Current Phase

**Phase:** 1 — Foundation

---

## Checklist

### Phase 1: Foundation

- [x] 1.1 Fork and clone repo
- [x] 1.2 Document crate hierarchy
- [x] 1.3 Create ARCHITECTURE.md
- [x] 1.4 Create CRATES.md

### Phase 2: Core Protocol (TODO)

- [ ] 2.1 radicle-crypto — Identity system
- [ ] 2.2 radicle — Core types and interfaces
- [ ] 2.3 radicle-git-* — Git integration

### Phase 3: Networking (TODO)

- [ ] 3.1 radicle-protocol — Protocol messages
- [ ] 3.2 radicle-node — Daemon architecture
- [ ] 3.3 radicle-fetch — Replication logic

### Phase 4: Collaboration (TODO)

- [ ] 4.1 radicle-cob — Collaborative objects
- [ ] 4.2 radicle-dag — History structure
- [ ] 4.3 Issues, patches, reviews

### Phase 5: User Interface (TODO)

- [ ] 5.1 radicle-cli — Command structure
- [ ] 5.2 radicle-remote-helper — `rad://` protocol
- [ ] 5.3 radicle-term — TUI components

### Phase 6: Deep Dives (TODO)

- [ ] 6.1 Gossip protocol analysis
- [ ] 6.2 Checkpointing/persistence
- [ ] 6.3 Security model
- [ ] 6.4 Comparison to GitHub/GitLab

---

## Progress Log

### Turn 1 (2026-02-26 10:36 CST)
- Forked heartwood repo
- Identified 21 crates

### Turn 2 (2026-02-26 10:48 CST)
- Created radicle-architecture repo
- Initial docs: README.md, ARCHITECTURE.md, CRATES.md

### Turn 3 (2026-02-26 10:50 CST)
- Phase 2: Identity system
- Created IDENTITY.md - Cryptographic identity foundation

### Turn 4 (2026-02-26 ~10:55 CST)
- Phase 3: Protocol
- Created PROTOCOL.md - P2P gossip protocol

### Turn 5 (2026-02-26 ~11:00 CST)
- Phase 3: Node Daemon
- Created NODE.md - Daemon architecture

### Turn 6 (2026-02-26 ~11:05 CST)
- Phase 4: Collaborative Objects
- Created COB.md - CRDT-based issues, patches, reviews

### Turn 7 (2026-02-26 ~11:10 CST)
- Phase 5: CLI
- Created CLI.md - Command-line interface

### Turn 8 (2026-02-26 ~11:15 CST)
- Phase 6: Deep Dives
- Created DEEP-DIVE.md - Protocol, persistence, security
  - Gossip: discovery, replication, interest filtering, fetch protocol
  - Persistence: Git as database, checkpointing via refs
  - Security: Ed25519, PoW, COB signatures, attack vectors
  - Commands: auth, init, clone, push, pull, sync
  - COB commands: issue, patch, review
  - Network: follow, seed, peers
  - Stored in Git, not database
  - CRU interface (no delete)
  - Offline-first with eventual consistency
  - Cryptographic signatures
  - Runtime: Initialization, storage, databases
  - Reactor: Event loop using mio
  - Service: Gossip protocol handler
  - Storage: Git with COB extensions
  - Message types: Subscribe, Announcement (Node/Inventory/Refs), Info, Ping/Pong
  - Wire format: Binary encoding with signatures
  - Proof-of-work for spam prevention
  - State machine: Connecting → Handshake → Gossiping → Shutdown
  - DID format: did:key:z6Mkr...
  - Radicle URN: rad:z...
  - Delegation and thresholds

---

## Key Source Files Identified

### Identity

| File | Crate | Purpose |
|------|-------|---------|
| `lib.rs` | radicle-crypto | Signing primitives |

### Networking

| File | Crate | Purpose |
|------|-------|---------|
| `service.rs` | radicle-protocol | Protocol state machine |
| `wire.rs` | radicle-protocol | Wire format |
| `fetcher.rs` | radicle-protocol | Fetch logic |

### Node Daemon

| File | Crate | Purpose |
|------|-------|---------|
| `lib.rs` | radicle-node | Main entry |
| `runtime.rs` | radicle-node | Async runtime |
| `reactor.rs` | radicle-node | Event reactor |

---

## Version Info

| Radicle Version | Architecture Version | Status |
|-----------------|---------------------|--------|
| Heartwood | 1.0.0 | In Progress |

---

## Related Repositories

| Repo | Purpose |
|------|---------|
| `mrhavens/heartwood` | Fork of source |
| `mrhavens/radicle-architecture` | Our documentation |
| `mrhavens/openclaw-architecture` | OpenClaw blueprints |
| `mrhavens/langgraph-architecture` | LangGraph blueprints |

---

## Next Steps

1. Continue with Phase 2: Identity system
2. Document radicle-crypto crate
3. Create IDENTITY.md

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
