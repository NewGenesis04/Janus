# Janus — sync the live, uncommitted workspace

Janus is the two-faced god of doorways. This project is a doorway between machines: it carries your working directory — and your agent session — from one computer to another in real time.

## The problem

Git preserves committed history. Everything between commits, your live working state, remains tied to a single machine. The file you're mid-edit on, the terminal scrollback, the agent conversation that's half-way through a feature — all of it lives and dies with that machine. Close the laptop, lose the thread.

The scene: you're coding at your desk and have to leave. You shut the laptop, throw it in a bag, get into a cab. Halfway home your boss calls — a small feature needs to finish today. Today that means waiting until you're back at the laptop, or hoping remote access was set up beforehand.

With Janus: open your phone, resume the exact agent session that was working on your repository, ask it to finish the feature, run the tests against the synchronized workspace, review the diff, commit and push. The original laptop doesn't need to remain online while work continues. When you're home, the workspace syncs back and your project is exactly where you left it — or roll back to an earlier state if you want.

The goal isn't to replace Git, cloud IDEs, or local development. The goal is to make the live, uncommitted workspace portable, so development can continue regardless of where the original machine is.

## The premise

Almost everything required to resume development already exists as files: source code, Git metadata, configuration, and even agent transcripts. Janus starts by making those portable.

## Model

- A **watcher** on the local machine streams changes to a remote **Workspace Replica** (self-hosted or hosted, OSS core).
- The replica holds the materialized workspace — source tree + session transcript.
- A dev can drop in on the VPS, boot an agent pointed at the transcript, finish the work, commit/push. Laptop stays off.
- Changes sync back home on return. Git stays the source of truth for versioning; Janus only moves the *uncommitted present*.

## Core primitive

A **live changeset stream**: `{path, op: add/modify/delete, delta, timestamp}` delivered in real time and applied to the replica copy. That's the whole engine. Session transcripts are just `paths` in the same stream.

## Architecture

```
laptop (watcher + client) ──delta stream──► workspace replica
                                                │
                              agent session resumes here, commits & pushes
```

- **Client:** fs watcher, debounce/coalesce, delta encode, transport, apply inbound.
- **Workspace Replica:** receive, apply, persist workspace + transcript; optional hosted flavor (auth, relay, storage) as the paid offering.
- **Session-as-file:** transcript lives under the watched dir, so continuity is free — remote agent reads it as context, no replay.

## Roadmap

1. **Files** — watcher → delta stream → replica apply → sync back. Real-time under sustained edits is the hard part (coalescing, ordering, large files).
2. **Sessions as files** — watch the transcript dir; remote resume uses the file as context.
3. **OSS + hosted split** — core repo, self-hostable; hosted replica as a fee service.

## Deliberately not in scope

- No conflict resolution v1 — one writer at a time per workspace.
- No replay/streaming/remote desktop.
- No git replacement.

**Why not Git?** Git captures milestones. Janus captures everything between milestones.

**Why not Codespaces?** Codespaces moves the development environment into the cloud. Janus keeps the development environment local and continuously mirrors the working directory.

**Why not Replit?** Replit is cloud-first. Janus is local-first.
