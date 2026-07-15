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

## PlayerBots Module

- Module repository: `/mnt/data/wow-server/source/modules/mod-playerbots`
- Official remote: `mod-playerbots/mod-playerbots`
- Required branch for the core `Playerbot` branch: `master`
- Installed module commit: `93aaea3de19243c09ce9ecb25627dc9671715eed`
- The module is an independent nested Git repository and is intentionally ignored by the parent repository's `/modules/*` rule.
- A functional PlayerBots build requires both the `mod-playerbots/azerothcore-wotlk` `Playerbot` core branch and this module.
- With `MODULES=static`, the core discovers the `mod-playerbots` directory and enables the `MOD_PLAYERBOTS` integration automatically.

Current module data and configuration paths:

- Dedicated PlayerBots database creation: `modules/mod-playerbots/data/sql/playerbots/create/create_mysql.sql`
- Dedicated PlayerBots base schema: `modules/mod-playerbots/data/sql/playerbots/base/`
- Dedicated PlayerBots updates: `modules/mod-playerbots/data/sql/playerbots/updates/`
- Character database additions: `modules/mod-playerbots/data/sql/characters/{base,updates}/`
- World database additions: `modules/mod-playerbots/data/sql/world/{base,updates}/`
- Installed module configuration source: `modules/mod-playerbots/conf/playerbots.conf.dist`
- Installed runtime configuration target: `env/dist/etc/modules/playerbots.conf`

The active core PlayerBots database updater uses `data/sql/playerbots/base/`. The module's legacy `conf/conf.sh.dist` still references obsolete `sql/...` paths that do not exist in the current module checkout; do not use those legacy paths for manual imports without reconciling them against the current `data/sql/...` layout.

## Environment

- Server: Ubuntu 24.04 LTS at `192.168.0.154`
- Server source: `/mnt/data/wow-server/source`
- Previous upstream build and accessible data directories were reviewed; the stopped Docker client-data volume remains unverified because its contents require elevated filesystem access.
- `/mnt/data/wow-server/build` is empty, and the native default build location under `source/var/build` contains no prior compiled output.
- `/mnt/data/wow-server/data/Data` contains raw enUS client MPQ data, not extracted server-side `dbc`, `maps`, `vmaps`, or `mmaps` directories.
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
- Completed read-only native Ubuntu build and installation reconnaissance.
- Cloned the official `mod-playerbots` `master` branch at commit `93aaea3de19243c09ce9ecb25627dc9671715eed`.
- Verified the current module build detection, database updater, SQL layout, and configuration paths.

## Lessons Learned

- PlayerBots is a modified core foundation, not a drop-in module for standard upstream AzerothCore.
- Build artifacts from a different AzerothCore source tree must not be reused.
- A personal fork does not forfeit upstream updates; separate `origin` and `upstream` remotes preserve both customization and synchronization.
- SSH access to the server and SSH authentication from the server to GitHub are separate trust relationships and require separate keys.
- Project-specific work should remain isolated on `custom` so the official `Playerbot` branch can serve as a clean synchronization reference.
- The PlayerBots core fork supplies required core integration, while the separately cloned module supplies the bot implementation, configuration, and module SQL.
- Module revisions must be recorded explicitly because the nested module repository is ignored by the parent Git repository.
- Current PlayerBots SQL is stored under `modules/mod-playerbots/data/sql/`; some legacy module tooling still refers to obsolete `modules/mod-playerbots/sql/` paths.

## Immediate Next Step

Install the required native Ubuntu build dependencies in one controlled step. Do not configure databases, extract client data, or start a build during that step.

## Session Closeout Record

### 2026-07-15 — Repository foundation

- Confirmed official branch, commit, remote, and working-tree state.
- Reviewed and committed `AGENTS.md`.
- Established Git identities and SSH authentication.
- Created the thin-fork remote and branch model.
- Added durable project-state, lessons-learned, game-design, and closeout documentation practices.
- Documented the project's program-management, engineering, and two-lens decision framework.

### 2026-07-15 — PlayerBots native source pairing

- Completed the read-only native Ubuntu workflow reconnaissance.
- Confirmed that PlayerBots requires the `Playerbot` core fork plus the separate `mod-playerbots` module.
- Cloned and verified `mod-playerbots` `master` at commit `93aaea3de19243c09ce9ecb25627dc9671715eed`.
- Confirmed both the parent `custom` checkout and nested module checkout were clean after cloning.
- Recorded the active module SQL/configuration paths and the obsolete legacy path conflict.
- Selected native Ubuntu dependency installation as the next controlled implementation step.
