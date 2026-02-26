# Radicle Deep Dive — Protocol, Persistence & Security

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

This document provides deep technical analysis of three critical aspects of Radicle:

1. **Gossip Protocol** — How discovery and replication work
2. **Persistence** — How data is stored and checkpointed
3. **Security Model** — Cryptographic guarantees

---

# Part 1: Gossip Protocol — Deep Analysis

## The Discovery Problem

Centralized systems solve discovery with databases:

```
GitHub: User searches → Database query → Results
```

Radicle has **no central database**. Instead, it uses **gossip** — peers tell each other what they know.

## Gossip Mechanics

### What is Gossip?

**Gossip protocols** are a class of peer-to-peer protocols where:

1. Nodes periodically share information with random peers
2. Information spreads exponentially through the network
3. Eventually, all honest nodes converge to the same state

### Radicle's Gossip Implementation

From `gossip.rs`:

```rust
pub fn inventory(
    timestamp: Timestamp,
    inventory: impl IntoIterator<Item = RepoId>,
) -> InventoryAnnouncement {
    InventoryAnnouncement {
        inventory: BoundedVec::truncate(inventory),
        timestamp,
    }
}
```

### Three Announcement Types

| Type | Content | Frequency |
|------|---------|-----------|
| **Node** | Node's addresses, features, alias | On connect + hourly |
| **Inventory** | List of repository IDs | On connect + periodic |
| **Refs** | Branch/tag refs for a repo | On push + periodic |

### Gossip Flow

```
1. Node A pushes new commit to repo R
2. A updates local refs
3. A creates RefsAnnouncement(R, refs, timestamp)
4. A signs with private key
5. A sends to connected peers (B, C, D)
6. B, C, D verify signature
7. B, C, D store announcement
8. B, C, D relay to their peers
9. Eventually: all interested nodes have announcement
```

### Interest Filtering

Nodes don't relay everything — they filter based on **interest**:

```rust
pub struct Subscribe {
    pub filter: Filter,      // What to subscribe to
    pub since: Timestamp,    // Start time
    pub until: Timestamp,    // End time
}
```

Filter by:
- Specific repository IDs
- Time windows

### Fetch Protocol

When a node sees a RefsAnnouncement for a repo it wants:

```
1. Node A receives: "B has repo R at refs {main: abc123}"
2. A checks local storage
3. A wants: main, develop (local is behind)
4. A sends Request(rid=R, refs=[main, develop])
5. B responds with Git objects
6. A integrates into local storage
```

From `session.rs`:

```rust
pub struct QueuedFetch {
    pub rid: RepoId,           // Repo being fetched
    pub from: NodeId,         // Peer being fetched from
    pub refs_at: Vec<RefsAt>, // Refs being fetched
    pub timeout: time::Duration,
}
```

### Timings

| Event | Interval |
|-------|----------|
| Gossip broadcast | Every 6 seconds |
| Inventory announcement | On connect + periodic |
| Node announcement | Every 60 minutes |
| Connection prune | Every 30 minutes |

---

## Gossip Properties

### Properties Guaranteed

1. **Eventual delivery** — If peers stay connected, info eventually spreads
2. **Integrity** — Messages are signed, tampering detected
3. **Origin authentication** — Verifiable who announced what

### Properties NOT Guaranteed

1. **Ordering** — Gossip doesn't guarantee message order
2. **Delivery time** — No latency guarantees
3. **Exactly-once** — May receive duplicate announcements

### Convergence

Gossip guarantees **eventual consistency**:
- If all nodes eventually receive the same announcements
- And apply them in timestamp order
- Then all replicas converge to same state

---

# Part 2: Persistence & Checkpointing

## Storage Architecture

### Git as Database

Radicle uses Git as its **sole storage backend**:

```
Git Objects:     ──→ Commit ──→ Tree ──→ Blob
                    │
Radicle Ext:    ──→ Rad ──→ Identity Document
                    │
COBs:           ──→ refs/heads/rad/issue/1
```

### What Gets Stored

| Data Type | Storage | Format |
|-----------|---------|--------|
| Git objects | `objects/` | Git native |
| Radicle refs | `refs/rad/` | Git refs |
| Identity | `refs/radicle.json` | JSON |
| COBs | `refs/heads/rad/issue/*` | Git objects |

## Checkpointing

### Traditional Checkpoints

Typical systems save periodic snapshots:

```
checkpoint_001.tar.gz
checkpoint_002.tar.gz
...
```

### Radicle's Approach

Radicle uses **Git's built-in checkpointing**:

1. **Commits are snapshots** — Every commit is a full snapshot
2. **Refs are pointers** — Branch tips point to commits
3. **History is immutable** — You can't change past commits

### The "Refs" Checkpoint

```rust
pub struct RefsAnnouncement {
    pub rid: RepoId,
    pub refs: BoundedVec<RefsAt, REF_REMOTE_LIMIT>,
    pub timestamp: Timestamp,
}
```

Announcing refs is the checkpoint:
- "At time T, repo R is at state S"
- Signed by node identity
- Stored in gossip database

### Persistence Layers

| Layer | What | How |
|-------|------|-----|
| **Git objects** | All data | Native Git |
| **Refs** | Current state | Git refs |
| **Gossip DB** | Announcements | SQLite |
| **COB history** | Issues/patches | Git + CRDT |

### Recovery

To recover a node from scratch:

```
1. Connect to seed nodes
2. Receive inventory announcements
3. For each desired repo:
   a. Find peers who have it
   b. Request refs
   c. Fetch objects
   d. Verify signatures
   e. Integrate into local Git
```

---

# Part 3: Security Model

## Threat Model

Radicle assumes:

1. **Network attackers** — Can intercept, drop, delay packets
2. **Malicious peers** — Can send invalid data
3. **Eclipse attacks** — Can isolate a node from honest network

## Security Mechanisms

### 1. Cryptographic Identity

Every node has an Ed25519 key pair:

```rust
pub struct PublicKey(ed25519::PublicKey);
pub struct Signature(ed25519::Signature);
```

**Properties:**
- No central authority needed
- Identity is the key
- Loss of key = loss of identity

### 2. Message Signing

All announcements are signed:

```rust
pub struct Announcement {
    pub node: NodeId,           // Signing node
    pub signature: Signature,  // Ed25519 signature
    pub message: AnnouncementMessage,
}

impl Announcement {
    pub fn verify(&self) -> bool {
        let msg = self.message.encode_to_vec();
        self.node.verify(msg, &self.signature).is_ok()
    }
}
```

### 3. Proof-of-Work

Node announcements require **proof-of-work**:

```rust
impl NodeAnnouncement {
    pub fn work(&self) -> u32 {
        // scrypt on serialized announcement
        // Count leading zero bits
    }

    pub fn solve(mut self, target: u32) -> Option<Self> {
        // Iterate nonces until work >= target
    }
}
```

**Purpose:** Economic spam prevention.

**Parameters:**
- Release: `(15, 8, 1)` — ~1 second compute
- Debug: `(1, 1, 1)` — instant

### 4. Timestamps

Announcements include timestamps:

```rust
pub struct NodeAnnouncement {
    pub timestamp: Timestamp,
    // ...
}
```

**Rules:**
- Timestamps too far in future → rejected
- Timestamps in past → accepted (but may be ignored)

### 5. Address Validation

Nodes announce their addresses, but validation is minimal:

- No IP spoofing prevention at protocol level
- Relies on TCP connection for address verification
- Optional: Tor for anonymity

### 6. COB Signatures

Every COB change is signed:

```rust
pub struct ExtendedSignature {
    // Ed25519 signature over change content
}
```

Verifiable authorship for all issues/patches.

---

## Attack Vectors

### Eclipse Attack

**What:** Attacker isolates victim from honest network.

**Mitigation:**
- Connect to multiple peers
- Prefer diverse network
- Seed nodes provide bootstrap

### Sybil Attack

**What:** Attacker creates many fake identities.

**Mitigation:**
- Proof-of-work on announcements
- Network connections cost resources

### Data Pollution

**What:** Attacker sends invalid data.

**Mitigation:**
- Signature verification required
- Invalid signatures → message ignored
- COBs: must verify author signature

### Replay Attack

**What:** Attacker re-sends old announcements.

**Mitigation:**
- Timestamps checked (recent only)
- Gossip DB may deduplicate

---

## Comparison: Security Properties

| Aspect | GitHub | GitLab | Radicle |
|--------|--------|--------|---------|
| **Identity** | Email + password | Email + password | Ed25519 key |
| **Data integrity** | Central trust | Central trust | Cryptographic |
| **Authorship** | Server logs | Server logs | Signed commits |
| **Transport** | HTTPS | HTTPS | P2P + optional Tor |
| **Spam prevention** | Rate limits | Rate limits | PoW |

---

## Privacy

### Metadata

**GitHub/GitLab know:**
- Who you are (email)
- What you work on
- When you work
- Who you collaborate with

**Radicle knows:**
- Only what you announce
- IP address (if not using Tor)

### Tor Integration

Radicle supports **Tor** for anonymity:

```json
{
  "node": {
    "relay": "onion"
  }
}
```

Using Tor hides:
- Your IP address
- Your network activity

---

## Summary

### What Radicle Protects Against

| Threat | Protection |
|--------|------------|
| Tampering | Signed messages |
| Impersonation | Ed25519 keys |
| Spam | Proof-of-work |
| Forgery | COB signatures |
| Censorship | P2P replication |

### What Radicle Does NOT Protect Against

| Threat | Note |
|--------|------|
| Metadata analysis | Use Tor for anonymity |
| Network-level attacks | Depends on network |
| Key compromise | No recovery mechanism |

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
