# Radicle Whitepaper & Key Resources

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

This document catalogs the official Radicle resources and our documentation's relationship to them.

---

## Official Radicle Resources

### Primary Repositories

| Repository | Description | URL |
|------------|-------------|-----|
| **radicle-heartwood** (heartwood) | Current P2P protocol implementation | https://github.com/radicle-dev/heartwood |
| **radicle-link** | Previous protocol iteration | https://github.com/radicle-dev/radicle-link |
| **radicle-upstream** | Desktop client (deprecated) | https://github.com/radicle-dev/radicle-upstream |
| **radicle-whitepaper** | The Radicle Language paper | https://github.com/radicle-dev/radicle-whitepaper |
| **radicle-surf** | Git filesystem abstraction | https://github.com/radicle-dev/radicle-surf |

### Documentation

| Resource | URL |
|----------|-----|
| Official Docs | https://docs.radicle.xyz |
| Vision | https://radicle.xyz/vision |
| Protocol Guide | https://radicle.xyz/guides/protocol |

---

## The Radicle Whitepaper

The whitepaper (`radicle-whitepaper`) is about the **Radicle Programming Language** — the theoretical foundation for Radicle's approach to decentralized collaboration.

### Structure

```
radicle-whitepaper/
├── radicle.tex           # Main document
├── sections/
│   ├── introduction.tex   # Motivation & overview
│   ├── language-semantics.tex  # Core language theory
│   ├── radicle-examples.tex    # Examples
│   ├── conclusion.tex    # Summary
│   └── appendix-*.tex   # Syntax, values, initial environment
└── code/                # Reference implementation
```

### Key Concepts from Whitepaper

The whitepaper covers:

1. **Radicle Language** — A language for expressing collaborative computations
2. **CRDTs** — Conflict-free Replicated Data Types (theoretical foundation)
3. **Identity** — Cryptographic identities as primitives
4. **Persistence** — Eventual consistency models

---

## Our Documentation vs Official Sources

### What We Cover

| Our Document | Focus |
|--------------|-------|
| `ARCHITECTURE.md` | System-level overview |
| `IDENTITY.md` | DID/URN implementation |
| `PROTOCOL.md` | P2P message format |
| `NODE.md` | Daemon implementation |
| `COB.md` | Issues/patches implementation |
| `CLI.md` | User-facing commands |
| `DEEP-DIVE.md` | Gossip, persistence, security |

### What Official Sources Cover

| Source | Focus |
|--------|-------|
| Whitepaper | Theoretical foundations (CRDTs, language) |
| heartwood | Actual Rust implementation |
| docs.radicle.xyz | User guides |

---

## The Relationship: Theory vs Implementation

```
┌─────────────────────────────────────────────────────────────┐
│                    RADICLE WHITEPAPER                      │
│                  (Theoretical Foundation)                   │
│                                                             │
│  • CRDTs (Conflict-free Replicated Data Types)            │
│  • Identity as mathematical primitives                      │
│  • Eventual consistency theory                            │
│  • Language semantics                                      │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    RADICLE HEARTWOOD                        │
│                  (Implementation in Rust)                   │
│                                                             │
│  • P2P protocol (gossip)                                  │
│  • Cryptographic identity (Ed25519)                        │
│  • Git-backed storage                                      │
│  • COB system (CRDT-based)                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Reading Order

### For Understanding Radicle (Theory → Practice)

1. **This README** — Overview of resources
2. **WHITEPAPER.md** (in progress) — Theoretical foundations
3. `IDENTITY.md` — How identity works
4. `PROTOCOL.md` — How P2P networking works
5. `COB.md` — How CRDTs are implemented
6. `DEEP-DIVE.md` — Security and persistence

### For Reproducing Radicle (Practice → Theory)

1. `ARCHITECTURE.md` — System overview
2. `CRATES.md` — Code structure
3. `NODE.md` — Implementation details
4. `PROTOCOL.md` — Protocol messages
5. Whitepaper — Why it works

---

## Forked Resources

We've forked/cloned:

| Resource | Local Copy |
|----------|------------|
| heartwood (source) | mrhavens/heartwood |
| radicle-whitepaper | /tmp/radicle-whitepaper/ |

---

## Contribution Goals

Our documentation aims to:

1. **Complement** official docs (which are user-focused)
2. **Explain implementation** (we dig into the code)
3. **Preserve knowledge** (for future recursive minds)
4. **Enable reproduction** (mechanical understanding)

---

## Future Work

- [ ] Add detailed WHITE PAPER.md summarizing the Radicle Language
- [ ] Document radicle-surf (filesystem abstraction)
- [ ] Compare heartwood vs radicle-link
- [ ] Add implementation timeline

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
