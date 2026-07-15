# Project State

## Purpose

This document is the durable operational handoff for the custom AzerothCore PlayerBots distribution. Update it when the project state, workflow, architecture, risks, or next implementation step changes.

## Current Phase

The reproducible native PlayerBots core and module foundation has been configured, compiled, verified, and installed. Runtime configuration, database provisioning and migration, client-data extraction, server startup, and gameplay customization have not begun.

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
- Native build directory: `/mnt/data/wow-server/build/playerbots-release-gcc13`
- Native install prefix: `/mnt/data/wow-server/source/env/dist`
- Previous upstream build and accessible data directories were reviewed; the stopped Docker client-data volume remains unverified because its contents require elevated filesystem access.
- The PlayerBots-native build is isolated from previous upstream output and from the native default build location under `source/var/build`.
- `/mnt/data/wow-server/data/Data` contains raw enUS client MPQ data, not extracted server-side `dbc`, `maps`, `vmaps`, or `mmaps` directories.
- Do not reuse the previous upstream build output for this PlayerBots fork.
- Windows WoW 3.3.5a client realmlist: `192.168.0.154`

Verified native build configuration:

- GCC/G++ 13.3.0 with ccache and Unix Makefiles
- `Release` build with `APPS_BUILD=all`, `TOOLS_BUILD=none`, `SCRIPTS=static`, and `MODULES=static`
- Unit tests disabled; core and script precompiled headers enabled; compiler warnings enabled
- Git revision embedding, jemalloc, and VMAP checks enabled; dynamic linking, core debug, and gperftools disabled
- Build parallelism limited to three jobs on the four-core, 15 GiB RAM host
- Installed binaries: `env/dist/bin/authserver` and `env/dist/bin/worldserver`
- Installed configuration templates: `env/dist/etc/{authserver,worldserver}.conf.dist` and `env/dist/etc/modules/playerbots.conf.dist`

Verified native Ubuntu build toolchain:

- Git 2.43.0
- CMake 3.28.3
- GNU Make 4.3
- GCC/G++ 13.3.0
- Clang/Clang++ 18.1.3
- ccache 4.9.1
- MySQL client development library 8.0.46
- OpenSSL development library 3.0.13
- Boost development libraries 1.83.0
- bzip2 development library 1.0.8
- Readline development library 8.2
- ncurses development library 6.4

All verified versions meet the documented AzerothCore requirements. MySQL Server is not installed as a native service; only the client development library required for compilation is installed.

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
- Installed and verified the native Ubuntu compiler, build tools, and development libraries without installing MySQL Server, configuring databases, extracting client data, or starting a build.
- Defined and reviewed a reproducible native GCC Release configuration with explicit source, build, install, CMake, and parallelism settings.
- Configured and compiled the paired PlayerBots core and module successfully, including both `authserver` and `worldserver`.
- Installed and verified the native binaries and configuration templates without configuring databases, extracting client data, or starting either server.

## Lessons Learned

- PlayerBots is a modified core foundation, not a drop-in module for standard upstream AzerothCore.
- Build artifacts from a different AzerothCore source tree must not be reused.
- A personal fork does not forfeit upstream updates; separate `origin` and `upstream` remotes preserve both customization and synchronization.
- SSH access to the server and SSH authentication from the server to GitHub are separate trust relationships and require separate keys.
- Project-specific work should remain isolated on `custom` so the official `Playerbot` branch can serve as a clean synchronization reference.
- The PlayerBots core fork supplies required core integration, while the separately cloned module supplies the bot implementation, configuration, and module SQL.
- Module revisions must be recorded explicitly because the nested module repository is ignored by the parent Git repository.
- Current PlayerBots SQL is stored under `modules/mod-playerbots/data/sql/`; some legacy module tooling still refers to obsolete `modules/mod-playerbots/sql/` paths.
- Build-tool installation and database-server installation are separate operational steps; the core can be compiled against the MySQL 8 client development library before any database service is configured.
- Enabling `WITH_WARNINGS` exposes repeated non-fatal unused-variable warnings from PlayerBots headers; these warnings did not prevent a successful build and should be reviewed upstream rather than patched locally during foundation work.
- A valid PlayerBots build can be verified before runtime startup through CMake's static module graph, the generated module loader, the `MOD_PLAYERBOTS` compile definition, linked PlayerBots symbols, installed configuration, and shared-library checks.

## Immediate Next Step

Define and review the native runtime and database-provisioning plan: establish configuration-file handling, database service and credential boundaries, database names and ownership, the supported core and PlayerBots schema/update sequence, backup and rollback points, and acceptance checks. Do not configure or import databases, extract client data, or start either server until the plan is reviewed and explicitly approved.

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

### 2026-07-15 — Native Ubuntu build dependencies

- Installed and verified the supported native compiler toolchains, CMake, Make, ccache, and required development libraries.
- Confirmed MySQL client development version 8.0.46, OpenSSL 3.0.13, and Boost 1.83.0 satisfy the documented minimum versions.
- Confirmed MySQL Server remains uninstalled and inactive as a native service.
- Confirmed no database configuration, client-data extraction, CMake configuration, or compilation occurred.
- Confirmed the parent `custom` checkout and nested `mod-playerbots` checkout remained clean and synchronized.
- Selected native build configuration review as the next controlled implementation step.

### 2026-07-15 — Reproducible native PlayerBots build and installation

- Reconfirmed the live parent and nested module branches, commits, remotes, tracking relationships, and clean working-tree state before configuration.
- Selected GCC/G++ 13.3.0, ccache, Unix Makefiles, a `Release` build, static scripts and modules, and a three-job parallel limit.
- Configured in `/mnt/data/wow-server/build/playerbots-release-gcc13` and installed to `/mnt/data/wow-server/source/env/dist`.
- Confirmed CMake discovered `mod-playerbots`, generated the static module-loader call, and emitted the `MOD_PLAYERBOTS` compile definition.
- Built and linked `authserver`, `libmodules.a`, and `worldserver` successfully despite non-fatal PlayerBots unused-variable warnings.
- Confirmed both installed executables are valid x86-64 ELF binaries with no unresolved shared libraries and that PlayerBots symbols are linked into `worldserver`.
- Confirmed the installed `playerbots.conf.dist` exactly matches the module source template.
- Recorded binary SHA-256 hashes: `authserver` `24a5cdf52c7542b985cc4685f92537293772063cf7ae689376e8725f1fc029dc`; `worldserver` `114971bce2601e4b42aa4f83c1feb58168854d2b981a9c7de8291677fbf27a39`.
- Confirmed neither server was started and no database configuration, SQL import, or client-data extraction occurred.
- Confirmed the parent `custom` checkout and nested `mod-playerbots` checkout remained clean after generated build and ignored install artifacts were created.
- Selected native runtime and database-provisioning review as the next controlled implementation step.
