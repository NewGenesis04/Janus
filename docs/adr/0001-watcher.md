# ADR-0001: The watcher — change detection, journal, and reconciliation

**Status:** Pending
**Date:** 2026-08-05

## Context

The client on the local machine must stream the live workspace to a remote replica — including agent transcripts as plain files. The laptop is unreliable by design: it's shut mid-sync, put in a bag, crashes. Nothing may be lost, and reconnect must be cheap.

## Decision

Four cooperating pieces, no fsnotify trust:

1. **Change detection** — fsnotify is a wake-up hint only. A periodic full scan (mtime+size, then sha256 of the differing files) is the source of truth. Survives dropped events, atomic renames, event bursts.
2. **Change tracking** — append-only **journal** on disk. Every detected change is journaled with a sequence number *before* send; replica acks; then it's trimmed. Unsent changes survive a crash; the journal doubles as the resume point.
3. **Reconciliation** — the replica holds a **manifest (`path → sha256`)**, the client mirrors it. Sync = exchange manifests, diff, transfer what differs. One mechanism handles bootstrap, reconnect, missed events, and sync-back-home. No special cases.
4. **Transport** — WebSocket, JSON frames, gzip per message. Works with the Python hub, survives NAT.

**Scope choices:** v1 sends full files on change (compression suffices; rsync-style deltas are a later optimization). `.gitignore`-aware filtering both directions. Workspace ID + token, TLS, no auth framework.

## Consequences

- Crash-safe by construction; reconnect is a manifest diff.
- Cost: periodic full scans; large-file sync is full-file v1.

## Open

- **Conflict rule** (one-writer v1): last-arrived wins, deterministic by hash+timestamp. Needs explicit sign-off.
- Changeset wire schema: defined in ADR-0002, or inline here?
