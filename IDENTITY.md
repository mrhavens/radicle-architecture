# Radicle Identity System

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

The Radicle identity system is cryptographic-first — identity is based on Ed25519 key pairs, not email addresses or usernames. This document details how identity works at the protocol level.

---

## Identity Fundamentals

### Key Types

| Type | Algorithm | Purpose |
|------|-----------|---------|
| **Node ID** | Ed25519 | Primary identity, network addressable |
| **Signing Key** | Ed25519 | Signs commits and changes |
| **Encryption Key** | X25519 | Sealed boxes, encrypted communication |

### Cryptographic Primitives

From `radicle-crypto`:

```rust
// Public key type
pub struct PublicKey(ed25519::PublicKey);

// Signature type  
pub struct Signature(ed25519::Signature);

// Signing trait
pub trait Signer: Send {
    fn public_key(&self) -> &PublicKey;
    fn sign(&self, msg: &[u8]) -> Signature;
    fn try_sign(&self, msg: &[u8]) -> Result<Signature, SignerError>;
}
```

---

## Identity Formats

### DID (Decentralized Identifier)

Radicle uses the **DID key method**:

```
did:key:MULTIBASE(base58-btc, MULTICODEC(public-key-type, raw-public-key-bytes))
```

Example:
```
did:key:z6MkrLMMsiPWUcNPHcRajuMi9mDfYckSoJyPwwnknocNYPm7
```

### Radicle URN

**URN (Uniform Resource Name)** is the Radicle-specific identifier:

```
rad:z<36-char-base32>/<path>
```

Components:
- `rad:` — Radicle scheme prefix
- `z<36-char-base32>` — Base32-encoded Ed25519 public key
- `/<path>` — Optional path (repository name, etc.)

**Full Examples:**

| Type | Format | Example |
|------|--------|---------|
| **Node ID** | `rad:z...` | `rad:z3hgcXBHBhc6Lie6R1UKiL4jSpnG1nqLZkYzLG5N2e1M` |
| **Repository** | `rad:z.../name` | `rad:z3hgcXBHBhc6Lie6R1UKiL4jSpnG1nqLZkYzLG5N2e1M/my-project` |
| **COB** | `rad:z.../name/rad/issue/1` | `rad:z3hgcXBHBhc6.../my-project/rad/issue/1` |

---

## Identity Document

Each identity has a **document** stored in Git:

### Document Structure (`radicle.json`)

```json
{
  "version": 1,
  "type": "project",  // or "user"
  "payload": {
    "name": "my-project",
    "description": "A cool project"
  },
  "delegates": [
    "did:key:z6Mkr..."
  ],
  "threshold": 1,
  "visibility": "public"
}
```

### Key Fields

| Field | Purpose |
|-------|---------|
| `version` | Document format version |
| `type` | Identity type (project/user) |
| `payload` | Metadata (name, description) |
| `delegates` | Keys authorized to make changes |
| `threshold` | Signatures required for updates |
| `visibility` | Public or private |

---

## Delegation

Radicle supports **multi-signature** identities:

1. **Single key** — One key controls identity (most common)
2. **Multiple delegates** — Several keys share control
3. **Threshold** — Require M of N signatures

```rust
pub struct Doc {
    pub delegates: Vec<Did>,    // Authorized keys
    pub threshold: usize,        // Required signatures
    // ...
}
```

---

## Identity Verification

### Verifying a Signature

```rust
use radicle_crypto::{PublicKey, Signature,Signer};

// Verify signature
let public_key = PublicKey::from_str(z6Mkr...)?;
let signature = Signature::from_str(sig_string)?;
let is_valid = public_key.verify(message, &signature);
```

### Verifying Commit Authorship

```bash
# Git verify-commit shows Radicle signature
git verify-commit <commit-hash>
```

---

## Storage

Identity documents are stored in Git:

```
refs/heads/radicle.json
```

The document is signed by delegate keys, providing cryptographic proof of identity.

---

## Comparison

| Aspect | GitHub | GitLab | Radicle |
|--------|--------|--------|---------|
| Identity | Email + password | Email + password | Ed25519 key |
| Verification | Password/token | Password/token | Cryptographic signature |
| Delegation | Org membership | Org membership | Key delegation |
| Recovery | Email reset | Email reset | Key backup |

---

## Key Security Properties

### 1. No Central Authority

- Identity is a key pair, not an account
- No "password reset" — key is the identity
- Loss of key = loss of identity

### 2. Cryptographic Proof

- Every change is signed
- Verifiable without trusted third party
- Tamper-evident history

### 3. Multi-Sig Support

- Threshold signatures enable:
  - Key rotation
  - Shared custody
  - Redundancy

---

## Node vs User Identity

### Node Identity

- The P2P node's cryptographic identity
- Used for networking (gossip, replication)
- URN: `rad:z...`

### User Identity

- The human's identity
- Delegates to node for signing
- Can have multiple nodes

### Repository Identity

- Owned by user identity
- Controlled via delegation
- Has its own URN

---

## Source References

| File | Crate | Purpose |
|------|-------|---------|
| `radicle-crypto/src/lib.rs` | radicle-crypto | Signer, PublicKey, Signature |
| `radicle/src/identity/did.rs` | radicle | Did (DID format) |
| `radicle/src/identity/doc.rs` | radicle | Identity document |
| `radicle/src/rad.rs` | radicle | Repository initialization |

---

## Further Reading

- [DID Core Specification](https://www.w3.org/TR/did-core/)
- [Ed25519 Signatures](https://ed25519.cr.yp.to/)

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
