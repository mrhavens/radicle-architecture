# Radicle Collaborative Objects (COBs)

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

**Collaborative Objects (COBs)** are Radicle's decentralized system for issues, patches, and reviews. Unlike traditional issue trackers that use databases, COBs are stored as **Git objects** and use **CRDTs** (Conflict-free Replicated Data Types) for eventual consistency.

This document details how COBs work at the protocol level.

---

## Core Concept

### What are COBs?

COBs are **collaborative objects** — distributed data structures that can be edited by multiple people without conflicts:

- **Issues** — Bug reports, feature requests
- **Patches** — Proposed code changes
- **Reviews** — Code reviews on patches

### CRDT Foundation

From the source:

```rust
//! Collaborative objects are graphs of CRDTs.
```

CRDTs guarantee:
- **Eventual consistency** — All replicas converge
- **No conflicts** — No merge conflicts in traditional sense
- **Offline-first** — Work offline, sync later

---

## COB Architecture

### Basic Types

| Type | Purpose |
|------|---------|
| `CollaborativeObject` | The computed object itself |
| `ObjectId` | Content-address for a COB |
| `TypeName` | Collection name (e.g., "issue", "patch") |
| `History` | Traversable history of changes |

### CRU Interface

COBs use a **CRU** interface (Create, Read, Update — no Delete):

```rust
// Create a new COB
pub fn create(...) -> CollaborativeObject

// Read COBs
pub fn get(...) -> CollaborativeObject
pub fn list(...) -> Vec<CollaborationalObject>

// Update a COB
pub fn update(...) -> Updated
```

---

## Storage

### Git Backend

COBs are stored in **Git**, not a database:

```
refs/heads/rad/issue/1   → Issue #1
refs/heads/rad/patch/1   → Patch #1  
refs/heads/rad/review/1   → Review #1
```

### Storage Abstraction

```rust
pub trait Store
where
    Self: object::Storage + change::Storage
{
    // Objects + Changes storage
}
```

### History as DAG

```rust
pub struct History {
    graph: Dag<EntryId, Entry>,
    root: EntryId,
}
```

The history is a **directed acyclic graph** (DAG) where:
- Nodes are `Entry` objects (changes)
- Edges are dependencies
- Tips are the current state

---

## Object Types

### Issue

```
refs/heads/rad/issue/{id}
```

Contains:
- Title
- Description
- State (open, closed)
- Comments
- Timestamps

### Patch

```
refs/heads/rad/patch/{id}
```

Contains:
- Title
- Description
- Base branch
- Head branch
- Revisions
- State (draft, open, merged, closed)

### Review

```
refs/heads/rad/review/{id}
```

Contains:
- Review comments
- Approvals/Rejections
- Suggestions

---

## Data Flow

### Creating an Issue

```
1. User runs: rad issue new "Bug in auth"
2. CLI creates Issue object
3. CLI creates initial Entry (CRDT operation)
4. Entry signed by user's key
5. Entry stored in Git
6. refs/heads/rad/issue/1 updated
7. Next gossip: Announcement spreads
```

### Updating an Issue

```
1. User runs: rad issue comment 1 "Found fix"
2. CLI creates new Entry with comment
3. Entry added to History DAG
4. refs/heads/rad/issue/1 updated to new tip
5. Gossip propagates update
6. All replicas merge automatically (CRDT)
```

### Merge Process

```
1. Receive new Entry from peer
2. Add Entry to local DAG
3. Create dependency edges from tips
4. Traverse to compute new state
5. No conflict — CRDT guarantees convergence
```

---

## Collaboration Model

### Identity

COBs are tied to **Radicle identities** (Ed25519 keys):

```rust
// Entry includes author identity
pub struct Entry {
    pub author: Author,
    pub resource: Resource,
    pub timestamp: Timestamp,
    // ...
}
```

### Signatures

Every change is **cryptographically signed**:

```rust
pub struct ExtendedSignature {
    // Ed25519 signature over the change content
}
```

This provides:
- **Authorship verification** — Who made the change
- **Tamper evidence** — Cannot modify history
- **Non-repudiation** — Author cannot deny

---

## Comparison to Traditional Systems

| Aspect | GitHub Issues | GitLab Issues | Radicle COBs |
|--------|--------------|--------------|--------------|
| Storage | Database | Database | Git |
| Offline | No | No | Yes (CRDT) |
| Sync | Central server | Central server | P2P Gossip |
| Identity | Email | Email | Ed25519 key |
| Deletion | Can delete | Can delete | CRU (no delete) |
| Conflicts | Merge conflicts | Merge conflicts | Auto-merge (CRDT) |

---

## Key Properties

### 1. Eventual Consistency

```
Alice (offline) adds comment A
Bob (offline) adds comment B
Both come online
→ Both see A + B (CRDT merge)
```

### 2. No Delete

```
"CRU Interface (No Delete)"
- Create
- Read  
- Update
- (No Delete)
```

This is a design choice — history is immutable.

### 3. Offline-First

```
Work without internet
Sync when connected
Always consistent
```

### 4. Cryptographic Provenance

```
Every change signed
Verifiable authorship
Tamper-evident
```

---

## Git References

COBs use specific Git ref patterns:

```
refs/heads/rad/              → COB namespace
refs/heads/rad/issue/       → Issues
refs/heads/rad/issue/1      → Issue #1
refs/heads/rad/issue/1/0    → Entry 0 of Issue #1

refs/heads/rad/patch/        → Patches
refs/heads/rad/patch/2      → Patch #2

refs/heads/rad/review/       → Reviews
refs/heads/rad/review/3      → Review #3
```

---

## Source References

| File | Purpose |
|------|---------|
| `radicle-cob/src/lib.rs` | Main COB interface |
| `radicle-cob/src/object.rs` | Object types |
| `radicle-cob/src/history.rs` | DAG history |
| `radicle-cob/src/change.rs` | Change/entry types |

---

## CLI Commands

```bash
# Issues
rad issue new "Bug in auth"
rad issue list
rad issue show 1
rad issue comment 1 "Found the issue"

# Patches
rad patch create
rad patch list
rad patch review 1 approve

# COB-specific
rad cob list issue
rad cob list patch
```

---

## Security Model

### 1. Signing Required

All COB changes must be signed by a valid Radicle identity.

### 2. Verifiable Authorship

```rust
// Verify signature on entry
entry.author.verify(&entry.signature)
```

### 3. No Tampering

Since history is a DAG of signed entries, tampering is detectable.

---

## Comparison to Centralized Systems

| Feature | GitHub | GitLab | Radicle |
|---------|--------|--------|---------|
| **Storage** | Database | Database | Git |
| **Offline** | ❌ | ❌ | ✅ |
| **Identity** | Email | Email | Key |
| **Delete** | Yes | Yes | No (CRU) |
| **Conflicts** | Merge | Merge | Auto-merge |
| **Discovery** | Search | Search | Follow |

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
