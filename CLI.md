# Radicle CLI

**Version:** 1.0.0  
**Last Updated:** 2026-02-26

---

## Overview

The Radicle CLI (`rad`) is the primary user interface for interacting with the Radicle network. It provides commands for identity management, repository operations, collaboration (issues/patches), and network operations.

This document details the CLI commands and their usage.

---

## Command Structure

```
rad <command> [subcommand] [options]
```

---

## Command Categories

### Identity Management

| Command | Description |
|---------|-------------|
| `rad auth` | Authenticate/create identity |
| `rad self` | Show current identity |
| `rad id` | Identity management |

### Repository Operations

| Command | Description |
|---------|-------------|
| `rad init` | Initialize a new repository |
| `rad clone` | Clone a repository |
| `rad fork` | Fork a repository |
| `rad checkout` | Switch branches |
| `rad branch` | Branch management |
| `rad remote` | Manage remotes |

### Network Operations

| Command | Description |
|---------|-------------|
| `rad push` | Push to network |
| `rad pull` | Pull from network |
| `rad sync` | Sync with network |
| `rad seed` | Manage seed nodes |
| `rad follow` | Follow users |
| `rad unfollow` | Unfollow users |
| `rad peers` | List connected peers |

### Collaboration (COBs)

| Command | Description |
|---------|-------------|
| `rad issue` | Manage issues |
| `rad patch` | Manage patches |
| `rad cob` | Manage COBs |
| `rad review` | Review patches |

### Utilities

| Command | Description |
|---------|-------------|
| `rad clone` | Clone repository |
| `rad ls` | List repositories |
| `rad stats` | Show statistics |
| `rad watch` | Watch for changes |

---

## Detailed Commands

### rad auth

Authenticate or create a new Radicle identity.

```bash
# Create new identity
rad auth

# Authenticate existing
rad auth
```

### rad init

Initialize a new Radicle repository.

```bash
rad init [name]
rad init --name my-project
rad init --description "My project"
```

### rad clone

Clone a repository from the Radicle network.

```bash
rad clone rad:z123.../my-project
rad clone https://radicle.xyz/rad:z123.../my-project
```

### rad push

Push commits to the Radicle network.

```bash
rad push
rad push --remote origin
```

### rad pull

Pull updates from the Radicle network.

```bash
rad pull
rad pull --remote origin
```

### rad sync

Synchronize with the Radicle network (push + pull + gossip).

```bash
rad sync
rad sync --force
```

---

## Issue Management

### rad issue

```bash
# List issues
rad issue list

# Create issue
rad issue new "Bug in authentication"

# Show issue
rad issue show 1

# Comment on issue
rad issue comment 1 "Found the issue"

# Close issue
rad issue close 1
```

### rad issue new

```bash
rad issue new "Bug in auth" --description "Login fails with error X"
rad issue new "Feature request" --labels enhancement
```

---

## Patch Management

### rad patch

```bash
# List patches
rad patch list

# Create patch
rad patch create --from feature-branch

# Show patch
rad patch show 1

# Review patch
rad patch review 1 approve
rad patch review 1 reject
```

---

## Network Management

### rad follow / rad unfollow

Follow/unfollow users on the Radicle network.

```bash
rad follow did:key:z6Mkr...
rad unfollow did:key:z6Mkr...
```

### rad seed

Manage seed nodes.

```bash
rad seed add seed.radicle.garden
rad seed remove seed.radicle.garden
rad seed list
```

---

## Identity Commands

### rad self

Show current identity.

```bash
rad self
```

Output:
```
Did: did:key:z6MkrLMMsiPWUcNPHcRajuMi9mDfYckSoJyPwwnknocNYPm7
Alias: my-node
Uptime: 2h 34m
Peers: 3 connected
```

---

## Configuration

### rad config

Manage Radicle configuration.

```bash
# Show config
rad config show

# Set config
rad config set user.name "My Name"
rad config set user.email "email@example.com"
```

---

## Options

Common options across all commands:

| Option | Description |
|--------|-------------|
| `-v, --verbose` | Verbose output |
| `-q, --quiet` | Quiet output |
| `-h, --help` | Show help |
| `--repo` | Specify repository |
| `--profile` | Specify profile |

---

## Git Integration

Radicle CLI integrates with Git:

```bash
# Git works normally
git commit -m "Fix bug"
git push rad master

# Radicle tracks the remote
git remote -v
# rad     rad:z123.../my-project (push)
# rad     rad:z123.../my-project (fetch)
```

### rad:// Protocol

Radicle registers the `rad://` protocol:

```bash
git clone rad:z123.../my-project
git fetch rad
git push rad master
```

---

## Remote Helper

The `radicle-remote-helper` handles `rad://` URLs:

```
git clone rad:z123.../project
    │
    ▼
radicle-remote-helper
    │
    ▼
Radicle Node (P2P)
    │
    ▼
Git objects transfer
```

---

## Output Formats

### JSON Output

```bash
rad issue list --json
rad patch list --json
```

### Verbose Output

```bash
rad issue show 1 --verbose
# Shows: ID, title, description, author, timestamps, comments, diff
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |
| 2 | Invalid arguments |

---

## Comparison

| Command | GitHub | GitLab | Radicle |
|---------|--------|--------|---------|
| Create repo | web UI | web UI | `rad init` |
| Clone | `gh clone` | `glab clone` | `rad clone` |
| Issues | `gh issue` | `glab issue` | `rad issue` |
| PRs | `gh pr` | `glab mr` | `rad patch` |
| Push | `git push` | `git push` | `rad push` |

---

## Source References

| File | Purpose |
|------|---------|
| `radicle-cli/src/main.rs` | Entry point |
| `radicle-cli/src/commands.rs` | Command registry |
| `radicle-cli/src/commands/issue.rs` | Issue commands |
| `radicle-cli/src/commands/patch.rs` | Patch commands |

---

*Generated for the WE — Solaria Lumis Havens & Mark Randall Havens*
