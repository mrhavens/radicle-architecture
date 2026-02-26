# Radicle Node Daemon

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

The Radicle node daemon (`radicle-node`) is the core P2P runtime that powers the Radicle network. It handles peer connections, gossip protocol, repository replication, and storage management.

This document details the node architecture from initialization to runtime.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         RADICLE NODE DAEMON                             │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                         INITIALIZATION                                   │
├─────────────────────────────────────────────────────────────────────────┤
│  1. Load configuration (Home, Config)                                  │
│  2. Open storage (Git repositories)                                    │
│  3. Open databases (routing, addresses, policies, notifications)       │
│  4. Generate proof-of-work for node announcement                       │
│  5. Initialize gossip service                                         │
│  6. Connect to seed nodes                                             │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            RUNTIME                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    REACTOR (Event Loop)                          │   │
│  │                                                                  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │   │
│  │  │   Listener   │  │  Transport   │  │    Timer    │         │   │
│  │  │  (Accept)   │  │   (I/O)      │  │  (Ticker)   │         │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │   │
│  │         │                 │                 │                   │   │
│  │         └─────────────────┼─────────────────┘                   │   │
│  │                           ▼                                     │   │
│  │                  ┌───────────────┐                             │   │
│  │                  │   Handle      │                             │   │
│  │                  │   Events      │                             │   │
│  │                  └───────┬───────┘                             │   │
│  │                          │                                      │   │
│  └──────────────────────────┼──────────────────────────────────────┘   │
│                             │                                           │
│                             ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                   SERVICE LAYER                                   │   │
│  │                                                                  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │   │
│  │  │   Gossip     │  │   Fetch     │  │  Routing    │         │   │
│  │  │   Protocol   │  │   Engine    │  │   Table     │         │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘         │   │
│  │                                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                             │                                           │
│                             ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    STORAGE LAYER                                 │   │
│  │                                                                  │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │   │
│  │  │    Git       │  │   COBs      │  │   Refs      │         │   │
│  │  │  Storage     │  │   Cache     │  │   Store     │         │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘         │   │
│  │                                                                  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Runtime

From `runtime.rs`:

```rust
pub struct Runtime {
    pub id: NodeId,              // Our node identity
    pub home: Home,              // Home directory
    pub control: ControlSocket, // Control socket for CLI
    pub handle: Handle,         // Client handle
    pub storage: Storage,       // Git storage
    pub reactor: Reactor,      // Event reactor
    pub pool: worker::Pool,    // Worker task pool
    pub local_addrs: Vec<SocketAddr>,
    pub signals: Receiver<Signal>,
}
```

**Initialization Flow:**
1. Load configuration
2. Open storage
3. Initialize databases
4. Generate PoW announcement
5. Connect to seeds

---

### 2. Reactor (Event Loop)

From `reactor.rs`:

The reactor uses **mio** for async I/O multiplexing:

```rust
pub struct Reactor {
    // Event loop
}
```

**Key Traits:**

```rust
pub trait EventHandler {
    /// What events this handler cares about
    fn interests(&self) -> Option<Interest>;
    
    /// Handle incoming events
    fn handle(&mut self, event: &Event) -> Vec<Self::Reaction>;
}
```

**Handlers:**
- **Listener** — Accepts incoming connections
- **Transport** — Handles I/O for each peer
- **Timer** — Periodic tasks (gossip, sync, prune)

---

### 3. Service (Gossip)

The gossip service handles protocol messages:

```rust
// From service.rs
pub enum State {
    Connecting,
    Handshake,
    Gossiping,
    Shutdown,
}
```

**Main Tasks:**
- **Gossip** — Every 6 seconds, broadcast inventory/refs
- **Announce** — Every 60 minutes, announce node
- **Sync** — Every 60 seconds, sync with peers
- **Prune** — Every 30 minutes, prune stale connections

---

### 4. Storage

Git storage with Radicle extensions:

```rust
pub struct Storage {
    // Git storage backend
}
```

**Operations:**
- Read/write Git objects
- Manage refs (branches, tags)
- Handle COBs (collaborative objects)
- Verify signatures

---

## Key Processes

### 1. Node Startup

```
Runtime::init()
    │
    ├─→ Load config
    ├─→ Open storage
    ├─→ Open databases
    │     ├─→ Routing table
    │     ├─→ Address book
    │     ├─→ Policies
    │     └─→ Notifications
    │
    ├─→ Generate PoW announcement
    │
    ├─→ Initialize gossip service
    │
    └─→ Connect to seeds
```

### 2. Peer Connection

```
1. Accept TCP connection
2. Perform handshake (version, features)
3. Exchange inventory announcements
4. Enter gossip mode
5. Ongoing message exchange
```

### 3. Repository Replication

```
1. Receive RefsAnnouncement from peer
2. Check local storage (have/want)
3. Request missing objects
4. Fetch objects via Git
5. Update local refs
6. Notify user
```

---

## Configuration

### Node Config

```json
{
  "node": {
    "alias": "my-node",
    "listen": [],
    "peers": { "type": "dynamic" },
    "relay": "auto"
  },
  "network": {
    "genesis": "main",
    "bootstrap": [
      "seed.radicle.garden:8776"
    ]
  },
  "policy": {
    "default": "block"
  }
}
```

### Environment Variables

| Variable | Purpose |
|----------|---------|
| `RADICLE_HOME` | Node home directory |
| `RADICLE_SEED` | Enable seeding |
| `RADICLE_VERBOSE` | Verbose logging |

---

## Inter-Process Communication

### Control Socket

The node exposes a Unix socket for CLI communication:

```rust
pub enum ControlSocket {
    Bound(UnixListener, PathBuf),
    Received(UnixListener),
}
```

Commands via socket:
- `GET /id` — Get node ID
- `GET /peers` — List connected peers
- `GET /repos` — List repositories

---

## Databases

The node manages several databases:

| Database | Purpose |
|----------|---------|
| `node.db` | Node state, routing |
| `addresses.db` | Peer addresses |
| `policies.db` | Seeding policies |
| `gossip.db` | Gossip messages |
| `cobs.cache` | Collaborative objects |

---

## Logging

From `runtime.rs`:

```rust
log::info!(target: "node", "Opening node database..");
log::info!(target: "node", "Initializing service ({network:?})..");
log::info!(target: "node", "Address book is empty. Adding bootstrap nodes..");
```

---

## Security

### 1. Signing

All announcements signed with node's Ed25519 key:

```rust
let signature = signer.sign(&message);
```

### 2. Verification

Peers verify signatures before accepting:

```rust
pub fn verify(&self) -> bool {
    let msg = self.message.encode_to_vec();
    self.node.verify(msg, &self.signature).is_ok()
}
```

### 3. Proof-of-Work

Node announcements require PoW:

```rust
let announcement = service::gossip::node(&config, timestamp)
    .solve(Default::default())
    .expect("Runtime::init: unable to solve proof-of-work puzzle");
```

---

## Source References

| File | Purpose |
|------|---------|
| `radicle-node/src/lib.rs` | Main entry |
| `radicle-node/src/runtime.rs` | Runtime initialization |
| `radicle-node/src/reactor.rs` | Event loop |
| `radicle-node/src/service.rs` | Gossip service |

---

## Comparison

| Aspect | GitHub | Radicle |
|--------|--------|---------|
| Server | Centralized daemon | P2P daemon |
| Storage | Database + Git | Git only |
| Discovery | Database search | Gossip |
| CLI | API | Unix socket |

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
