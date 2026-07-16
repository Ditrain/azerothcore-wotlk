# Project State

## Purpose

This document is the durable operational handoff for the custom AzerothCore PlayerBots distribution. Update it when the project state, workflow, architecture, risks, or next implementation step changes.

## Current Phase

The reproducible native PlayerBots core and module foundation has been configured, compiled, verified, and installed. Native MySQL 8.0, the four empty AzerothCore databases, the scoped application account, and private runtime configuration are installed and verified for a trusted two-to-three-player household deployment. Repository SQL has not been applied, client data has not been extracted, neither game server has been started, and gameplay customization has not begun.

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
- Runtime root: `/mnt/data/wow-server/runtime`
- Planned extracted-data directory: `/mnt/data/wow-server/runtime/data`
- Runtime log directory: `/mnt/data/wow-server/runtime/logs`
- Private updater temporary directory: `/mnt/data/wow-server/runtime/tmp`
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

All verified versions meet the documented AzerothCore requirements. Ubuntu MySQL Server and Client 8.0.46 are installed. The `mysql` service is enabled, active, and healthy; classic protocol port 3306 and X Protocol port 33060 listen only on `127.0.0.1`. A LAN connection probe to port 3306 fails as intended.

Verified native database and runtime baseline:

- Empty databases: `acore_auth`, `acore_characters`, `acore_world`, and `acore_playerbots`
- Core database defaults: `utf8mb4` with `utf8mb4_unicode_ci`
- PlayerBots database default: `utf8mb4` with `utf8mb4_general_ci`
- Application identity: `acore@127.0.0.1`
- Grants are limited to the four databases, without global privileges or `GRANT OPTION`.
- The credential authenticates successfully over TCP loopback and is retained privately by the owner for runtime use.
- Runtime files: `env/dist/etc/authserver.conf`, `env/dist/etc/worldserver.conf`, and `env/dist/etc/modules/playerbots.conf`
- Runtime configuration files are ignored by Git, owned by `ditrain`, and mode `600`; installed `.conf.dist` templates remain unchanged.
- Runtime root, data, and log directories are mode `750`; the updater temporary directory is mode `700` and was empty at closeout.
- Explicit runtime paths use the native source directory, `/usr/bin/mysql`, and the runtime data, log, and private temporary directories above.

## Reviewed Native Runtime and Database Plan

The deployment is a trusted household LAN server for two or three family members. Operational choices should protect against credible failures--configuration mistakes, failed schema updates, accidental secret commits, unintended network exposure, and disk loss--without enterprise-style identity or secret-management complexity.

Selected baseline:

- Install Ubuntu's native MySQL 8.0 server and matching client. The current binaries are linked to MySQL client 8.0.46, the core requires MySQL 8.0 or newer, and the external `mysql` executable is required by the built-in updater.
- Bind MySQL to loopback only. Do not expose port 3306 to the LAN.
- Use the four conventional databases: `acore_auth`, `acore_characters`, `acore_world`, and `acore_playerbots`.
- Use one strong local `acore@127.0.0.1` credential shared by authserver, worldserver, and PlayerBots, with privileges limited to those four databases and without global privileges or `GRANT OPTION`.
- Do not execute `modules/mod-playerbots/data/sql/playerbots/create/create_mysql.sql`; it grants global privileges on `*.*` with `GRANT OPTION` and is unnecessarily broad.
- Create runtime files from the installed `.conf.dist` templates while leaving the templates unchanged: `env/dist/etc/authserver.conf`, `env/dist/etc/worldserver.conf`, and `env/dist/etc/modules/playerbots.conf`. Keep runtime files untracked and restrict their permissions.
- Set explicit `SourceDirectory`, `MySQLExecutable`, `DataDir`, `LogsDir`, and a private `TempDir`. The updater writes the password to a fixed `mysql_ac.conf` under `TempDir`; use a non-shared, access-restricted location and do not expose or commit it.
- Keep the built-in updater enabled. Authserver initializes auth first. Worldserver then initializes characters and world, discovers PlayerBots character/world SQL through the compiled `AC_MODULES_LIST`, and finally invokes the PlayerBots database hook to initialize and update `acore_playerbots`.
- Do not use the module's legacy `conf/conf.sh.dist` paths. Current SQL is under `modules/mod-playerbots/data/sql/`.
- Extract and verify `dbc`, `maps`, `vmaps`, and `mmaps` from the raw client data before the first worldserver provisioning run so database mutation and full worldserver readiness can be validated in one controlled step.
- Create a complete logical backup after successful initial provisioning and before future schema/module updates. Keep restorable copies on a disk separate from MySQL's data directory and test recovery before treating the server as durable.

Current updater evidence:

- Core base schemas are populated lexically from `data/sql/base/db_{auth,characters,world}/`.
- Released core updates run before module SQL.
- PlayerBots character and world SQL is recursively discovered from the module and recorded as `MODULE` updates.
- The dedicated PlayerBots database is populated from `modules/mod-playerbots/data/sql/playerbots/base/`, then updated from its configured include directories.
- The current module's early-sorting character and world update files operate on core tables that already exist, so the inspected ordering is viable at the pinned revisions. Recheck this after module upgrades.

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
- Defined and reviewed the native runtime and database-provisioning plan against the current core and PlayerBots updater implementation.
- Simplified the plan to match the trusted household deployment: one loopback-only MySQL service, one scoped application credential, four databases, built-in updates, private runtime configuration, and practical backups.
- Installed and verified matching Ubuntu MySQL Server and Client 8.0.46 packages and confirmed that MySQL is healthy and exposed only on loopback.
- Created the four empty AzerothCore databases with the repository-compatible character sets and collations.
- Created and authenticated the single `acore@127.0.0.1` application account with privileges limited to the four databases and without global privileges or `GRANT OPTION`.
- Created the untracked runtime configuration files from the installed templates, applied explicit native runtime paths, inserted the database credential through an owner-only hidden prompt, and restricted the secret-bearing files to mode `600`.
- Created the runtime data, log, and private updater temporary directories with proportional owner-controlled permissions while leaving all installed `.conf.dist` templates unchanged.

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
- Security and operational architecture must be proportional to the real deployment. This household server does not justify separate database identities, separate service identities, or enterprise secret infrastructure unless a concrete future requirement appears.
- Efficient reconnaissance starts with a bounded decision. Consolidate read-only checks, surface important findings early, and stop when further inspection is unlikely to change the selected mechanism.
- The PlayerBots database-creation script is not appropriate for this deployment because it grants global privileges with `GRANT OPTION`; create the four databases and scoped grants explicitly instead.
- The updater writes a fixed `mysql_ac.conf` containing the database password under `TempDir`; protect that directory and keep it out of Git.
- Selecting the MariaDB driver in DBeaver identifies the client driver, not the database server installed on the target host; confirm the saved connection host and port before inferring server state.
- `GRANT USAGE ON *.*` in `SHOW GRANTS` represents an account with no global privileges; the effective application rights are the four explicit database-scoped grants.
- Insert operational secrets through an owner-controlled hidden prompt. Verify permissions and placeholder removal locally, and avoid displaying or fingerprinting credential-bearing configuration during remote review.

## Immediate Next Step

Obtain explicit approval for one controlled implementation step: build and install the AzerothCore client-data extraction tools for the pinned PlayerBots source and module revisions. Then extract `dbc`, `maps`, `vmaps`, and `mmaps` from `/mnt/data/wow-server/data/Data` into `/mnt/data/wow-server/runtime/data`, verify completeness and compatibility, confirm neither game server was started, and stop before applying repository SQL or provisioning databases through the server updaters.

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

### 2026-07-15 — Native runtime and database-provisioning review

- Re-read `AGENTS.md`, `PROJECT_PRINCIPLES.md`, `PROJECT_STATE.md`, and `GAME_DESIGN.md` before inspection.
- Reconfirmed the clean parent `custom` branch at `28cecb85b1d69c1f2fca5b213c63728a5b871d53` tracking `origin/custom` and the clean nested module `master` branch at `93aaea3de19243c09ce9ecb25627dc9671715eed` tracking `origin/master`.
- Reconfirmed installed binary hashes and exact template/source matches; confirmed neither game server nor a native MySQL/MariaDB service was running.
- Inspected the current `DatabaseLoader`, `DBUpdater`, `UpdateFetcher`, world/auth startup loaders, PlayerBots database hook, configuration templates, and live SQL paths rather than relying on generic instructions.
- Selected native Ubuntu MySQL 8.0 plus its matching client, four conventional databases, one loopback-only scoped application credential, built-in schema/update handling, and a practical logical-backup procedure.
- Rejected the module's global-grant database creation script and recorded the updater's private `TempDir` requirement.
- Confirmed raw client MPQs are present but extracted server data and extraction tools are not; client-data extraction must precede the first worldserver provisioning run.
- Corrected the initial over-engineered credential/service separation proposal after applying the real household threat and failure model.
- Made no database, runtime configuration, client-data, package, or server-process changes during the review.
- Selected MySQL server/client installation and verification as the next single approval-gated step.

### 2026-07-15 — Native MySQL and private runtime baseline

- Reconfirmed parent `custom` at `b84de197e9b3774dbdb95b0a5be9def2181f3b64` and nested module `master` at `93aaea3de19243c09ce9ecb25627dc9671715eed`, both clean and synchronized before implementation.
- Installed matching Ubuntu MySQL Server and Client 8.0.46 packages with administrator authentication supplied directly by the owner.
- Verified the MySQL service is enabled, active, and healthy; ports 3306 and 33060 listen only on `127.0.0.1`, and a LAN probe to port 3306 fails.
- Created empty `acore_auth`, `acore_characters`, `acore_world`, and `acore_playerbots` databases with their repository-compatible character sets and collations.
- Created `acore@127.0.0.1`, verified TCP authentication, and confirmed its only effective privileges are scoped to the four databases without `GRANT OPTION`.
- Did not execute either repository database-creation script and did not manually import base or update SQL.
- Created `/mnt/data/wow-server/runtime/{data,logs,tmp}`; restricted `tmp` to mode `700` and the other runtime directories to mode `750`.
- Created ignored runtime configuration files from the installed `.conf.dist` templates, set explicit source, MySQL executable, data, log, and private temporary paths, and restricted the files to mode `600`.
- The owner inserted the saved application password through a hidden local prompt. Remote verification intentionally did not read, display, or fingerprint the credential.
- Reconfirmed MySQL remained healthy, authserver and worldserver remained stopped, the private updater directory remained empty, installed templates remained intact, and both Git worktrees remained clean and synchronized.
- Selected client-data extractor build, extraction, and verification as the next approval-gated step before any server-driven database provisioning.
