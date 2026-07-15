# Project State

## Purpose

This document is the durable operational handoff for the custom AzerothCore PlayerBots distribution. Update it when the project state, workflow, architecture, risks, or next implementation step changes.

## Current Phase

The project is establishing a reproducible PlayerBots-native foundation. No PlayerBots build, database migration, or gameplay customization has begun in this phase.

## Repository Model

- Working repository: `/mnt/data/wow-server/source`
- Active development branch: `custom`
- Writable remote: `origin` (`Ditrain/azerothcore-wotlk`)
- Official remote: `upstream` (`mod-playerbots/azerothcore-wotlk`)
- Official base branch: `upstream/Playerbot`
- The local `Playerbot` branch should remain aligned with `upstream/Playerbot`.
- Project-specific commits belong on `custom`.

This thin-fork model preserves official PlayerBots updates while providing a durable, writable home for project documentation, configuration, SQL, modules, and other custom work.

## Environment

- Server: Ubuntu 24.04 LTS at `192.168.0.154`
- Server source: `/mnt/data/wow-server/source`
- Previous upstream build and data directories still require review.
- Do not reuse the previous upstream build output for this PlayerBots fork.
- Windows WoW 3.3.5a client realmlist: `192.168.0.154`

## Completed

- Replaced the previous upstream source tree with the mod-playerbots AzerothCore fork.
- Confirmed the official `Playerbot` branch and repository remote.
- Added repository-level operating instructions in `AGENTS.md`.
- Configured SSH access from the Windows development environment to the Ubuntu server.
- Configured GitHub SSH authentication from the Ubuntu server.
- Created the writable GitHub fork and the `custom` development branch.
- Separated the writable `origin` remote from the official `upstream` remote.

## Lessons Learned

- PlayerBots is a modified core foundation, not a drop-in module for standard upstream AzerothCore.
- Build artifacts from a different AzerothCore source tree must not be reused.
- A personal fork does not forfeit upstream updates; separate `origin` and `upstream` remotes preserve both customization and synchronization.
- SSH access to the server and SSH authentication from the server to GitHub are separate trust relationships and require separate keys.
- Project-specific work should remain isolated on `custom` so the official `Playerbot` branch can serve as a clean synchronization reference.

## Immediate Next Step

Perform read-only repository reconnaissance to determine the officially supported native Ubuntu build and installation workflow for this PlayerBots branch. Do not install packages, configure databases, delete files, or start a build during that inspection.

## Session Closeout Record

### 2026-07-15 — Repository foundation

- Confirmed official branch, commit, remote, and working-tree state.
- Reviewed and committed `AGENTS.md`.
- Established Git identities and SSH authentication.
- Created the thin-fork remote and branch model.
- Added durable project-state, lessons-learned, game-design, and closeout documentation practices.
- Documented the project's program-management, engineering, and two-lens decision framework.
