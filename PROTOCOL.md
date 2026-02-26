# Radicle P2P Protocol

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

The Radicle protocol is a P2P gossip protocol for replicating Git repositories. It uses cryptographic signatures, proof-of-work, and a custom wire format to enable decentralized code collaboration.

This document details the protocol at the message level.

---

## Protocol Constants

| Constant | Value | Purpose |
|----------|-------|---------|
| `ADDRESS_LIMIT` | 16 | Max addresses per node announcement |
| `REF_REMOTE_LIMIT` | 1024 | Max refs in announcement |
| `INVENTORY_LIMIT` | 2973 | Max inventory items |

---

## Protocol Version

- Current: **7** (Heartwood)
- The version is announced in `NodeAnnouncement`

---

## Message Types

### 1. Subscribe

Subscribe to gossip messages matching a filter and time range.

```rust
pub struct Subscribe {
    pub filter: Filter,      // What to subscribe to
    pub since: Timestamp,    // Start time
    pub until: Timestamp,    // End time
}
```

**Purpose:** Tell peers what messages you want to receive.

---

### 2. Announcement

The core gossip message. Three types:

#### 2a. Node Announcement

```rust
pub struct NodeAnnouncement {
    pub version: u8,           // Protocol version
    pub features: Features,   // Node capabilities
    pub timestamp: Timestamp, // Monotonic time
    pub alias: Alias,          // Display name
    pub addresses: Vec<Address>, // Network addresses
    pub nonce: u64,           // PoW nonce
    pub agent: UserAgent,     // Client identifier
}
```

**Features:**
- `SEED` — Node seeds repositories
- `ANONYMOUS` — Anonymous peer
- Protocol version support

#### 2b. Inventory Announcement

```rust
pub struct InventoryAnnouncement {
    pub inventory: Vec<RepoId>, // Repos this node has
    pub timestamp: Timestamp,
}
```

**Purpose:** Announce what repositories you have.

#### 2c. Refs Announcement

```rust
pub struct RefsAnnouncement {
    pub rid: RepoId,              // Repository ID
    pub refs: Vec<RefsAt>,        // Updated refs
    pub timestamp: Timestamp,
}
```

**Purpose:** Announce updated branches/tags for a repo.

---

### 3. Info

Informational messages between peers (not relayed):

```rust
pub enum Info {
    // Tell peer they've already synced this ref
    RefsAlreadySynced { rid: RepoId, at: Oid },
}
```

---

### 4. Ping/Pong

Connection keep-alive and liveness checking:

```rust
pub struct Ping {
    pub ponglen: Size,   // Requested pong size
    pub zeroes: ZeroBytes, // Ignored bytes
}

pub struct Pong {
    pub zeroes: ZeroBytes,
}
```

---

## Wire Format

### Encoding

Messages use a custom binary encoding:

```rust
// NodeAnnouncement encoding (example)
impl wire::Encode for NodeAnnouncement {
    fn encode(&self, buf: &mut impl BufMut) {
        self.version.encode(buf);
        self.features.encode(buf);
        self.timestamp.encode(buf);
        self.alias.encode(buf);
        self.addresses.encode(buf);
        self.nonce.encode(buf);
        self.agent.encode(buf);
    }
}
```

### Message Envelope

All messages wrap in an `Announcement`:

```rust
pub struct Announcement {
    pub node: NodeId,           // Signing node
    pub signature: Signature,   // Cryptographic signature
    pub message: AnnouncementMessage, // The actual message
}
```

**Verification:**
```rust
impl Announcement {
    pub fn verify(&self) -> bool {
        let msg = self.message.encode_to_vec();
        self.node.verify(msg, &self.signature).is_ok()
    }
}
```

---

## Proof-of-Work

Node announcements include **proof-of-work** to prevent spam:

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

**Parameters:**
- Debug: `(1, 1, 1)` — Easy to compute
- Release: `(15, 8, 1)` — Standard difficulty

**Purpose:** Economic barrier against spam.

---

## Protocol Flow

### Connection Establishment

```
1. Connect (TCP)
2. Exchange NodeAnnouncement
   - Version negotiation
   - Feature advertisement
3. Exchange InventoryAnnouncement
   - Share what repos you have
4. Ongoing Gossip
```

### Gossip Protocol

```
1. Node A has new refs for repo R
2. A creates RefsAnnouncement(R, refs)
3. A signs with private key
4. A sends to connected peers
5. Peers verify signature
6. Peers relay to their peers
7. Interested nodes fetch missing objects
```

### Fetch Protocol

```
1. Node A sees RefsAnnouncement for repo R
2. A checks local storage: needs refs X, Y
3. A sends Request(rid=R, refs=[X,Y])
4. Peer responds with objects
5. A integrates into local Git storage
```

---

## State Machine (Service)

From `service.rs`:

```rust
pub enum State {
    /// Initial state, connecting
    Connecting,
    /// Connected, handshaking
    Handshake,
    /// Synced and gossiping
    Gossiping,
    /// Shutting down
    Shutdown,
}
```

---

## Security Model

### 1. Cryptographic Identity

- Every node has Ed25519 key pair
- Messages signed with private key
- Verifiable without trusted third party

### 2. Proof-of-Work

- Node announcements require PoW
- Economic spam prevention
- Configurable difficulty

### 3. Signature Verification

- Every announcement signed
- Invalid signatures dropped
- Fork detection via signed refs

---

## Network

### Transport

- **Primary:** TCP
- **Optional:** Tor for anonymity

### Port

- Default: `8776`

### Seed Nodes

Radicle has permanent seed nodes:

```
seed.radicle.garden:8776
ash.radicle.garden:8776
```

---

## Message Limits

| Type | Limit |
|------|-------|
| Addresses per node | 16 |
| Refs per announcement | 1024 |
| Inventory items | 2973 |
| Ping zeroes | Variable |
| Message size | MAX_SIZE |

---

## Source References

| File | Purpose |
|------|---------|
| `radicle-protocol/src/service/message.rs` | Message types |
| `radicle-protocol/src/service.rs` | State machine |
| `radicle-protocol/src/wire.rs` | Encoding/decoding |

---

## Comparison to GitHub/GitLab

| Aspect | GitHub | Radicle |
|--------|--------|---------|
| Protocol | HTTPS | Custom P2P |
| Authentication | Token | Ed25519 key |
| Identity | Account | Cryptographic |
| Discovery | Search | Gossip |
| Reliability | Centralized | Distributed |

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
