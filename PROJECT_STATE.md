# Project State

## Purpose

This document is the durable operational handoff for the custom AzerothCore PlayerBots distribution. Update it when the project state, workflow, architecture, risks, or next implementation step changes.

## Current Phase

The reproducible native PlayerBots core, module, extraction tools, MySQL 8.0 runtime, private configuration, WoW 3.3.5a server-side client data, all four database schemas, the default PlayerBots population, initial post-provisioning logical backup, end-to-end LAN login path, manual native systemd operating workflow, and first approved gameplay module increment are installed and verified for a trusted two-to-three-player household deployment. The owner account `DITRAIN` can authenticate, create and enter a character, see PlayerBots, and invite a bot that accepts the group invitation. `DITRAIN` is now the permanent owner GM account at security level 2 (`SEC_GAMEMASTER`); it is not an administrator-level account. The accidental `YOACCOUNT` account was deleted through the supported worldserver command. Transmog is pinned, restore-backed, installed, conservatively configured, database-verified, and fully play-tested through apply, relog persistence, removal, and original-appearance restoration. Neither game server is currently running; boot startup remains disabled and the private updater temporary directory is empty.

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
- Extracted-data directory: `/mnt/data/wow-server/runtime/data`
- Runtime log directory: `/mnt/data/wow-server/runtime/logs`
- Private updater temporary directory: `/mnt/data/wow-server/runtime/tmp`
- Previous upstream build and accessible data directories were reviewed; the stopped Docker client-data volume remains unverified because its contents require elevated filesystem access.
- The PlayerBots-native build is isolated from previous upstream output and from the native default build location under `source/var/build`.
- `/mnt/data/wow-server/data/Data` contains the raw enUS client MPQs used for the verified extraction.
- `/mnt/data/wow-server/runtime/data` contains the pinned-tool output: `dbc`, `maps`, `Cameras`, `vmaps`, and `mmaps`.
- Do not reuse the previous upstream build output for this PlayerBots fork.
- Windows WoW 3.3.5a client realmlist: `192.168.0.154`

Verified native build configuration:

- GCC/G++ 13.3.0 with ccache and Unix Makefiles
- `Release` build with `APPS_BUILD=all`, `TOOLS_BUILD=none`, `SCRIPTS=static`, and `MODULES=static`
- Unit tests disabled; core and script precompiled headers enabled; compiler warnings enabled
- Git revision embedding, jemalloc, and VMAP checks enabled; dynamic linking, core debug, and gperftools disabled
- Build parallelism limited to three jobs on the four-core, 15 GiB RAM host
- Installed binaries: `env/dist/bin/authserver` and `env/dist/bin/worldserver`
- Installed configuration templates: `env/dist/etc/{authserver,worldserver}.conf.dist` and `env/dist/etc/modules/{playerbots,transmog}.conf.dist`

Verified native extraction-tool configuration:

- Isolated build directory: `/mnt/data/wow-server/build/playerbots-tools-release-gcc13`
- `Release` build with `APPS_BUILD=none`, `TOOLS_BUILD=maps-only`, `SCRIPTS=none`, and `MODULES=none`
- GCC/G++ 13.3.0 with ccache, Unix Makefiles, tests disabled, warnings enabled, and three-job build/extraction parallelism
- Installed tools: `env/dist/bin/{map_extractor,vmap4_extractor,vmap4_assembler,mmaps_generator}` and `env/dist/bin/mmaps-config.yaml`
- Extraction logs: `/mnt/data/wow-server/runtime/logs/client-data-*-777577671778.log`

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

Verified native database and runtime state:

- Provisioned databases: `acore_auth`, `acore_characters`, `acore_world`, and `acore_playerbots`
- Core database defaults: `utf8mb4` with `utf8mb4_unicode_ci`
- PlayerBots database default: `utf8mb4` with `utf8mb4_general_ci`
- Application identity: `acore@127.0.0.1`
- Grants are limited to the four databases, without global privileges or `GRANT OPTION`.
- The credential authenticates successfully over TCP loopback and is retained privately by the owner for runtime use.
- Runtime files: `env/dist/etc/authserver.conf`, `env/dist/etc/worldserver.conf`, and `env/dist/etc/modules/{playerbots,transmog}.conf`
- Runtime configuration files are ignored by Git, owned by `ditrain`, and mode `600`. The core and PlayerBots files contain secrets and must never be read or fingerprinted remotely; the Transmog file is non-secret. Installed `.conf.dist` templates remain unchanged.
- Runtime root, data, and log directories are mode `750`; the updater temporary directory is mode `700` and was empty at closeout.
- Explicit runtime paths use the native source directory, `/usr/bin/mysql`, and the runtime data, log, and private temporary directories above.
- The built-in updater initialized all four schemas and applied the pinned core, module, and PlayerBots updates without manual SQL imports.
- First world initialization created 100 default PlayerBots accounts and 1,000 bot characters, reached the exact `(worldserver-daemon) ready...` marker, and produced an empty `Errors.log`.
- Authserver and worldserver were stopped gracefully after verification; all four database pools closed and neither game server is currently running.

## Native Server Operations

A minimal native systemd workflow is defined under `ops/systemd/`, with its
household operator guide in `ops/README.md`.

- `azerothcore.target` groups authserver and worldserver for coordinated operation.
- Both services run as the existing `ditrain` owner and use the existing private runtime configuration in place.
- MySQL is a required dependency; authserver is ordered before worldserver, and shutdown reverses that order.
- The core's supported `SIGTERM` path provides graceful shutdown, with a five-minute stop timeout for PlayerBots database queues to drain.
- `Restart=on-failure` retries crashes after five seconds without restarting an intentional clean shutdown.
- The initial policy is manual on-demand startup. Boot startup remains disabled unless the owner explicitly enables `azerothcore.target` later.
- Status, journal logs, selective restarts, complete-pair restarts, and boot enable/disable commands are documented in the operator guide.
- An attachable tmux console is intentionally not included; it remains a separate option if a concrete operational need develops.

The tracked definitions and their installed copies pass `systemd-analyze verify`
and byte-match. The installed files are root-owned mode `644`; the grouping target
is disabled, and both component services are static. A controlled manual start/stop
test verified execution as `ditrain`, auth-before-world startup, listeners on 3724
and 8085, fresh world readiness, journal access, zero restart loops, reverse graceful
shutdown, closure of all five database pools, inactive final state, and empty private temp storage.

Verified initial logical backup:

- Bundle: `/mnt/data/wow-server/backups/initial-playerbots-20260716T142210Z` (94 MiB)
- MySQL data device: `/dev/mapper/ubuntu--vg-ubuntu--lv` on the system disk
- Backup device: `/dev/sdb1`, a separate physical disk mounted at `/mnt/data`
- Protected ownership and permissions: backup root mode `750`; bundle mode `700`; files mode `600`; owner `ditrain`
- Included compressed dumps: `acore_auth`, `acore_characters`, `acore_world`, and `acore_playerbots`
- Source/restored table counts matched: auth 22, characters 111, world 313, and PlayerBots 30
- Critical source/restored row counts matched: 100 auth accounts and 1,000 characters
- All four SHA-256 checksums and gzip streams verified; all restored tables passed `mysqlcheck`; `restore_test=passed`
- The bundle includes `BACKUP_METADATA.txt`, `SHA256SUMS`, and `RESTORE.txt`; the credential-free one-use creation script was removed after verification.

Verified household LAN smoke test:

- WoW 3.3.5a uses `set realmlist 192.168.0.154`; no PlayerBots-specific client patch is required.
- Fresh authserver evidence advertised `AzerothCore` at `192.168.0.154:8085`; authserver listened on `3724` and worldserver on `8085`.
- Worldserver reached the exact ready marker with all 1,000 bot characters available and 500 random bots prepared for login.
- Owner account `DITRAIN` authenticated from the LAN, created and entered a character, saw other PlayerBots, invited one, and the bot accepted.
- The mistakenly created `YOACCOUNT` account was deleted with the supported `account delete YOACCOUNT` console command; `DITRAIN` was retained.
- Both servers then stopped gracefully, all four database pools closed, and the updater temporary directory was returned to empty mode-`700` state.
- The fresh error log contained only a repeated missing waypoint-path warning for stock creature `Eye of Dar'Khan`; it did not affect readiness, login, bot visibility, or grouping and is not a foundation blocker.

## Selected Pre-Development Module Stack

The owner wants a period of ordinary household play before custom gameplay development. The following community components are selected for staged compatibility review and installation; none was installed during the selection session:

- Transmogrification: `azerothcore/mod-transmog`. This is the lowest-risk first candidate and should use its normal module build and built-in database-update path.
- Auction-house population: `NathanHandley/mod-ah-bot-plus`, canonical repository `https://github.com/NathanHandley/mod-ah-bot-plus.git`. It was selected over the older stock AHBot because its configuration and pricing controls are more suitable for a small private realm. At the 2026-07-18 closeout, upstream `master` pointed to `f685832994c825f90aa5a3dc0e1620aa568e875b`; treat that as reconnaissance evidence, not an installation pin until the next-session compatibility review is complete. The module explicitly requires ordinary non-bot characters when used alongside PlayerBots. Do not use `DITRAIN`, RandomBots, PlayerBots accounts, or characters intended for normal play as AH seller identities.
- Living-world dialogue: `Hokken/mod-llm-chatter`, its matching Chatter Companion client addon, and the module's separate Python bridge on Ubuntu. The bridge should call LM Studio on the owner's Windows desktop over the private LAN; do not expose the inference API publicly. Disable overlapping stock PlayerBots chatter if required by the module to avoid duplicate dialogue.
- Normal local dialogue model: `anthracite-org/magnum-v4-9b-gguf`, quantization `Q4_K_M`. It was selected as the roleplay-quality and GPU-headroom balance for the desktop RTX 4070 while WoW is running.
- Dependable secondary local model: `ggml-org/gemma-4-12B-it-GGUF`, quantization `Q4_K_M`. Only one model needs to be loaded for normal play; compare response validity, latency, character distinction, repetition, fantasy-roleplay behavior, and WoW frame-time impact before final tuning.

The initial local inference policy is one request at a time, Flash Attention enabled, conservative context and output limits, and restrained chatter frequency. The LLM supplies dialogue only: PlayerBots remains authoritative for combat, movement, questing, loot, and other gameplay decisions. The module's persistent identities, memories, backstories, event context, and selected NPC proximity conversations provide the two desired experiences--ambient living-world chatter and richer recurring party companions--without installing two competing LLM chat modules.

The owner later identified additional roleplay-model candidates because Magnum has not been updated in LM Studio for almost two years: Mistral-Nemo-12B-Celeste v1.9, Fimbulvetr-11B-v2, MythoMax L2 13B, roleplay-tuned or uncensored Llama 3.x 8B-to-13B variants, Mistral NeMo/Small 12B variants, and Chronos Hermes 13B-class models. Treat Magnum as a prior baseline, not a final decision. During the later LLM increment, verify exact current model repositories and LM Studio compatibility, then benchmark one loaded model at a time on the RTX 4070 for valid-response rate, latency, persona separation, repetition, lore/fantasy behavior, and WoW frame-time impact before selecting the normal-play model and quantization.

## Transmog Increment - Installed and Verified

- Approval: the owner explicitly approved the one-module Transmog increment.
- Canonical repository: `https://github.com/azerothcore/mod-transmog.git`
- Reviewed branch: `master`
- Pinned candidate and current nested checkout: `33ac64b2c305eb1b6fbc97310a7ecbc30c2ba4ef`
- Nested checkout: `/mnt/data/wow-server/source/modules/mod-transmog`, detached at the pinned commit, clean, with canonical `origin`.
- Parent tracking behavior: `/modules/*` is intentionally ignored, so the parent repository does not track the nested module. Preserve and verify the exact revision explicitly.
- License: AGPL-3.0.
- Compatibility/build evidence: the pinned module meets the inspected core requirements; CMake discovered both `mod-playerbots` and `mod-transmog` as static modules. A warning-enabled clean Release build completed successfully with three jobs on 2026-07-16. The build-directory worldserver is a valid x86-64 ELF executable containing a Transmogrification symbol; its SHA-256 is `bd6d2f2d1d21667601ec230ddbdfa215e7166fa0ae4527555bbe2fe2900ec854`.
- Database scope reviewed before approval: auth table `acore_cms_subscriptions`; character tables `custom_transmogrification`, `custom_transmogrification_sets`, and `custom_unlocked_appearances`; world content including creatures `190010`/`190011`, spell `200100`, item entries `57575`/`57576`, text entries `601083`/`601084`, and related commands, strings, and locales.
- Verified pre-module backup: `/mnt/data/wow-server/backups/pre-transmog-20260718T155801Z`. All four dumps exist with owner-only permissions; all checksum and gzip checks pass; isolated MySQL restore, table checks, and source/restored counts pass; metadata reports `restore_test=passed`; and all disposable data was removed. Source/restored counts were 22 auth tables, 111 character tables, 313 world tables, 30 PlayerBots tables, 101 accounts, and 1,001 characters. The protected initial baseline backup remains unchanged.
- The first two corrected restore attempts failed safely and cleaned up because the disposable `mysqld` first lacked parent traversal and then was blocked by AppArmor. The successful helper used the profile-approved `/var/lib/mysql-files/**` path without disabling AppArmor and was removed after verification.
- Installed artifacts: `env/dist/bin/{authserver,worldserver}` and `env/dist/etc/modules/transmog.conf.dist`. Installed SHA-256 values are authserver `38a0c75b0036fd0b9b07b8c60f142c61ab26e0dc5e231345a791fc3890c4e542` and worldserver `c240237ed1916bfb0c1989211ada40c49bee069b8b8769adaf09d2fe1b1317eb`; the installed worldserver contains the Transmogrification symbol.
- Conservative runtime configuration: ignored owner-only `env/dist/etc/modules/transmog.conf`, derived from the pinned template with only `Transmogrification.UseCollectionSystem = 0` and `Transmogrification.RetroActiveAppearances = 0` changed; `Transmogrification.EnablePlus = 0` remains the explicit pinned default. No permanent Transmog NPC is placed.
- Built-in updater verification: auth table `acore_cms_subscriptions`; character tables `custom_transmogrification`, `custom_transmogrification_sets`, and `custom_unlocked_appearances`; two world creature templates with zero permanent spawns; spell `200100`; item entries `57575`/`57576`; text entries `601083`/`601084`; 45 module strings; eight commands; and exact auth, character, and world module update records. New Transmog player-data tables were empty at verification.
- Runtime smoke test: both supported systemd services started without restart, ports 3724/8085 listened, worldserver reached its exact readiness marker, `DITRAIN` logged in, PlayerBots were visible and accepted a group invitation, and `.transmog 0` / `.transmog 1` returned the expected hide/show responses. Shutdown reversed service order, drained queued PlayerBots work, closed all five database pools, and exited both services with status `0`.
- Live usability validation: the level-1 owner character `Test` received two unrestricted green cloth chest items through GM commands: Grunt Vest (`3752`) as the equipped target and Banshee Armor (`5420`) as the bag source. The collection system remained disabled. Banshee Armor was applied to Grunt Vest, survived unequip/re-equip and a fresh login, was removed through the normal gossip interface, and the original Grunt Vest appearance returned and survived another fresh login.
- Owner access decision: the owner explicitly selected `DITRAIN` as the permanent GM account. The account was changed from security level 0 to level 2 (`SEC_GAMEMASTER`) for all realms through the supported worldserver command. Three temporary direct grants used during diagnosis (`npc add temp`, `additem`, and `modify money`) were revoked after the GM role was applied; access now comes from the standard GM role graph. No PlayerBots account or character received GM or Transmog-specific access.
- Known command-routing limitations: in this pinned module, named Transmog subcommands registered after the empty root handler are routed to its boolean parser; `.transmog interface off` reports that `Interface` is not a valid boolean. The root hide/show command works. Separately, `.npc add temp 190010` returned the generic `.npc` usage list even after a fresh `DITRAIN` session showed effective `SEC_GAMEMASTER` access and the GM role statically linked permissions 571-577, including `npc add temp`. Do not treat that command as a dependable temporary access path in this pin.
- Verified temporary access path: after targeting self, `.cast self 200100` invoked the module-provided spell and summoned the player-owned Ethereal Warpweaver template `190011`. Its standard `npc_transmogrifier` gossip applied and removed appearances successfully. This spell-created NPC made no permanent world placement and did not require enabling the collection system, Transmog Plus, or PlayerBots behavior.
- Permanent-placement decision: no permanent Warpweaver was placed. The owner deferred placement and may later stand at the desired location and run `.npc add 190010`; selecting that spawn and using `.npc delete` is the supported removal path.
- Final runtime state: all players were saved, the direct test worldserver gracefully drained 2,057 queued character queries, all four worldserver database pools closed, and authserver then closed its auth pool. MySQL is healthy and loopback-only; all AzerothCore units are inactive; boot startup is disabled; ports 3724/8085 are closed; `/mnt/data/wow-server/runtime/tmp` is empty and mode `700`; and no authserver or worldserver process or temporary tmux session remains. The fresh `Errors.log` retained only the accepted stock `Eye of Dar'Khan` missing-waypoint warning; `Server.log` also recorded one non-fatal movement-spline velocity validation warning for stock creature entry `7439` during play.

Any Race/Any Class remains desired but is deliberately outside the initial module stack. The known community candidate, `heyitsbench/mod-arac`, requires server DBC changes and a client `Patch-A.MPQ` on every household client and has greater PlayerBots, trainer, spell, quest, form, pet, and resource compatibility risk. Audit it separately after the initial modules are stable and the owner has spent time playing.

Installation policy:

- Pin and record every module repository, branch, commit, upstream remote, license, and update path before integration.
- Add, build, provision, configure, and smoke-test one module at a time, with a cohesive commit and rollback point for each.
- Preserve the verified pre-gameplay baseline backup. Do not overwrite it; create a new separately named backup before the first module changes a database.
- Never commit runtime secrets, LM Studio credentials, or secret-bearing `.conf` files. Keep inference traffic on the trusted private LAN.

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
- Built and installed the four pinned client-data extraction tools in an isolated tool-only build without changing the verified server binaries or installed configuration templates.
- Extracted and verified enUS client build 12340 data into `/mnt/data/wow-server/runtime/data`: 246 DBC files, 5,744 map files, 14 camera files, 101 vmap trees plus 2,693 vmap tiles, and 98 mmap headers plus 3,682 mmap tiles.
- Provisioned the auth database through a controlled authserver first start, verified its database pool, and stopped it gracefully.
- Provisioned the characters, world, and dedicated PlayerBots databases through a controlled worldserver first start using only the built-in updaters.
- Created and verified the official default population of 100 PlayerBots accounts and 1,000 bot characters, reached full worldserver readiness, and stopped worldserver gracefully after all queued database writes drained.
- Created and independently verified the initial four-database logical backup on a separate physical disk, including a successful disposable-database restore test and matching source/restored table and critical row counts.
- Completed an end-to-end LAN client smoke test through account login, character entry, visible PlayerBots activity, and successful bot grouping; removed the accidental test account and retained only the intended owner account.
- Installed and verified the pinned Transmog module through a separate restore-tested pre-module backup, conservative configuration, built-in database updates, database inspection, LAN/PlayerBots/Transmog command smoke tests, and graceful supported shutdown.

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
- The updater can leave `mysql_ac.conf` behind after a clean server shutdown and creates it with broader file permissions than desired. Never read it; after confirming both servers are stopped, restrict and remove only that exact temporary file, then verify the mode-`700` temporary directory is empty.
- Selecting the MariaDB driver in DBeaver identifies the client driver, not the database server installed on the target host; confirm the saved connection host and port before inferring server state.
- `GRANT USAGE ON *.*` in `SHOW GRANTS` represents an account with no global privileges; the effective application rights are the four explicit database-scoped grants.
- Insert operational secrets through an owner-controlled hidden prompt. Verify permissions and placeholder removal locally, and avoid displaying or fingerprinting credential-bearing configuration during remote review.
- The current map extractor also produces `Cameras`; preserve that directory under `DataDir` because the pinned worldserver loads cinematic camera assets from the extracted data path.
- A zero exit from `map_extractor` is not sufficient evidence by itself because a missing locale can also return zero; require the detected locale, client build, key files, counts, and compatible headers.
- The pinned vmap extractor's successful raw output contains `Buildings/dir_bin` without a separate `Buildings/dir`; use its explicit completion message and the assembler's successful output rather than assuming both index names exist.
- A backup is not verified merely because `mysqldump` exited successfully. Require protected storage on a separate device, per-file checksums, compression tests, an actual restore into uniquely named disposable databases, table checks, critical row-count comparisons, restore instructions, and cleanup verification.
- Retained logs can make a current configuration appear stale. Preserve old logs under explicit archive names and use fresh logs when validating a changed or uncertain runtime endpoint.
- Ubuntu AppArmor confines `/usr/sbin/mysqld` even when ordinary ownership and traversal permissions are correct. Disposable restore tests should use an explicitly permitted path such as `/var/lib/mysql-files/**`; do not disable the profile to accommodate an arbitrary path.
- In the pinned Transmog command table, the empty root handler precedes several named player subcommands. Those later names can be consumed as boolean arguments; verify command routing against the live core instead of assuming every database command row is reachable.
- A module can be integrated and smoke-tested without permanent world placement: preserve zero creature spawns until the owner makes a separate placement decision. Account RBAC still controls whether an in-game temporary-spawn command is available.
- Effective GM RBAC does not guarantee every nested command route works in this pin. `DITRAIN` had `SEC_GAMEMASTER`, and role 197 linked the relevant NPC permissions, but `.npc add temp 190010` still returned generic usage. The module's spell `200100` provided a clean temporary Ethereal Warpweaver path without a database spawn.
- In a collection-disabled Transmog configuration, a reliable level-1 compatibility pair is Grunt Vest (`3752`) as the equipped target and Banshee Armor (`5420`) as the bag source. Both are unrestricted uncommon cloth chest items with different displays and no required level.
- `NathanHandley/mod-ah-bot-plus` explicitly requires ordinary non-bot AH identities when used with PlayerBots. Keep AH seller characters separate from `DITRAIN`, RandomBots, PlayerBots accounts, and ordinary playable characters.

## Immediate Next Step

Begin the separately bounded Auction Bot Plus increment with read-only compatibility and scope review of `https://github.com/NathanHandley/mod-ah-bot-plus.git`. Reconfirm upstream `master`, inspect the exact candidate commit, license, AzerothCore minimum-core requirement, SQL/update behavior, build integration, runtime configuration, auction mutation and rollback behavior, PlayerBots compatibility, and ordinary-character GUID requirements. Do not clone, build, install, create AH identities, mutate auctions, or change databases during the review. If the evidence supports installation, present the exact pin, database/backup scope, character plan, initial conservative seller/buyer settings, validation criteria, and rollback plan for owner approval before implementation.

## Next Session Start Checklist

Before acting in a new session:

- Read `AGENTS.md`, `PROJECT_PRINCIPLES.md`, `PROJECT_STATE.md`, and `GAME_DESIGN.md` completely.
- Work on `custom`; verify it is clean and synchronized with `origin/custom`. Preserve the tested local `Playerbot` pin `52f58186a53399e603c46c24977fe60fcaad7f9d` and PlayerBots module pin `93aaea3de19243c09ce9ecb25627dc9671715eed` until a separate upgrade increment. At 2026-07-18 closeout, refreshed upstream refs had advanced to core `bf25eae704f5fa2cb6c9fc2458f96ec099c03a8e` and module `3fa1c1e49f8f1324b72461e576bce7c89b0a6521`; do not fold that drift into unrelated work.
- Reconfirm MySQL is healthy and bound only to loopback, neither game server is running, `/mnt/data/wow-server/runtime/tmp` is empty and mode `700`, and all four runtime configuration files remain owned by `ditrain` and mode `600`. Treat only `transmog.conf` as non-secret.
- Do not read, display, hash, fingerprint, replace, or commit the secret-bearing `.conf` files. The saved application credential is already present; do not request the database password.
- Reconfirm `/mnt/data/wow-server/runtime/data` contains the verified `dbc`, `maps`, `Cameras`, `vmaps`, and `mmaps` outputs with the recorded counts and owner-controlled permissions. Do not re-extract unless verification identifies a concrete incompatibility.
- Preserve `/mnt/data/wow-server/backups/initial-playerbots-20260716T142210Z` as the verified pre-gameplay baseline and `/mnt/data/wow-server/backups/pre-transmog-20260718T155801Z` as the verified pre-module rollback point. Before relying on or moving either bundle, rerun `sha256sum -c SHA256SUMS` and the gzip integrity checks; do not overwrite them.
- Preserve the owner account `DITRAIN` and never request, display, log, or document its password. The accidental `YOACCOUNT` account has been deleted.
- Preserve the verified native systemd workflow: manual on-demand startup, boot startup disabled, services running as `ditrain`, and crash-only restart behavior.
- Do not rerun database creation scripts or manually import SQL; future schema changes continue through the pinned built-in updaters after a verified backup.
- Preserve the verified day-one bot population: 100 bot accounts and 1,000 bot characters. Do not remove or regenerate it unless a concrete gameplay decision requires that change.
- Treat Transmog as installed and verified at pinned commit `33ac64b2c305eb1b6fbc97310a7ecbc30c2ba4ef`; preserve its conservative runtime options and zero permanent NPC spawns. `mod-ah-bot-plus` and `mod-llm-chatter` remain uninstalled future increments.
- Preserve `DITRAIN` as the owner GM at security level 2. Do not raise it to administrator level 3 or grant GM access to PlayerBots without a separate explicit decision.
- Treat the Transmog usability objective as complete: apply, persistence, removal, and original-appearance restoration passed. Use `.cast self 200100` as the verified temporary Warpweaver path; `.npc add temp 190010` is not dependable in the current pin. The owner may independently place one permanent NPC later with `.npc add 190010`, but none exists at closeout.
- The next candidate is `NathanHandley/mod-ah-bot-plus`; upstream `master` was `f685832994c825f90aa5a3dc0e1620aa568e875b` at closeout. Re-fetch and review before pinning. AH identities must be new ordinary non-bot characters, not `DITRAIN` or PlayerBots. Obtain the owner's preferred visible seller name or names and whether seller-only or seller-plus-buyer behavior is desired before configuration.
- Install and verify only one module per separately approved increment, with a new restore-tested backup before each database-changing installation.
- Keep Any Race/Any Class deferred until its server-DBC, client-patch, PlayerBots, trainer, spell, quest, form, pet, and resource compatibility has been reviewed separately.
- For the LLM increment, obtain the Windows desktop LAN address, exact LM Studio version, exact installed model identifiers, and the owner's preferred API-listening procedure. Do not request or record unrelated credentials.

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

### 2026-07-16 — Native client-data extraction

- Reconfirmed parent `custom` at `77757767177837939f33268f7556e2dd01c1a14a`, reference `Playerbot` at `52f58186a53399e603c46c24977fe60fcaad7f9d`, and nested module `master` at `93aaea3de19243c09ce9ecb25627dc9671715eed`, all clean and synchronized before implementation.
- Configured the isolated `/mnt/data/wow-server/build/playerbots-tools-release-gcc13` tool-only build and compiled and installed `map_extractor`, `vmap4_extractor`, `vmap4_assembler`, and `mmaps_generator` with the three-job limit.
- Confirmed the existing authserver and worldserver hashes and all installed `.conf.dist` templates remained unchanged after tool installation.
- Extracted enUS client build 12340 into an isolated staging directory, requiring successful tool completion and validating `WDBC`, `MAPS` v9, `VMAP_4.8`, and `MMAP` v19 headers before promotion.
- Promoted 246 DBC files (87 MiB), 5,744 map files (295 MiB), 14 camera files (60 KiB), 101 vmap trees plus 2,693 vmap tiles (657 MiB), and 98 mmap headers plus 3,682 mmap tiles (2.1 GiB) into `/mnt/data/wow-server/runtime/data`.
- Restricted extracted directories to mode `750` and files to mode `640`, confirmed no symlinks remained, and removed only the approved extraction staging remnants.
- Reconfirmed MySQL remained healthy and loopback-only, the LAN database probe remained refused, both game servers remained stopped, private runtime configuration permissions remained `600`, and no SQL, updater, database, server-start, or backup action occurred.
- Selected controlled built-in database provisioning and first-start readiness verification as the next approval-gated step.

### 2026-07-16 - Initial database and PlayerBots provisioning

- Reconfirmed the pinned parent, reference, and nested module revisions; verified clean tracking state, unchanged server binary hashes, byte-identical installed templates, complete client data, private configuration permissions, loopback-only MySQL listeners, and stopped game servers.
- Started authserver first with its saved private configuration, allowed the built-in updater to initialize `acore_auth`, verified `Started auth database connection pool.`, and stopped authserver gracefully.
- Started worldserver second and allowed only the built-in core and module updaters to initialize and update `acore_characters`, `acore_world`, and `acore_playerbots`; no repository creation script or manual SQL import was used.
- Included the intended official default bot population from day one: 100 bot accounts and 1,000 bot characters were created and reported available.
- Verified the exact `(worldserver-daemon) ready...` marker, an empty `Errors.log`, and no fatal provisioning marker.
- Sent a graceful interrupt and allowed 351 queued character queries and 541 queued PlayerBots queries to drain; verified all four database pools closed and both game servers stopped.
- Without reading it, restricted and removed the updater-created `mysql_ac.conf` remnant after each stopped-server phase; verified `/mnt/data/wow-server/runtime/tmp` is empty and mode `700` at closeout.
- Reconfirmed MySQL is enabled, active, and loopback-only; private runtime files remain mode `600`; installed templates and server binaries remain unchanged; and the parent and nested module worktrees remain clean and synchronized.
- Selected a complete, verified post-provisioning logical backup of all four databases as the next separate approval-gated step before gameplay customization.

### 2026-07-16 - Verified initial logical backup

- Reconfirmed the clean pinned repositories, stopped game servers, active loopback-only MySQL service, empty mode-`700` updater directory, matching MySQL 8.0.46 dump/restore tools, and available backup capacity.
- Confirmed MySQL data resides on the system disk while `/mnt/data` resides on separate physical disk `/dev/sdb1`.
- Created four compressed single-transaction logical dumps under `/mnt/data/wow-server/backups/initial-playerbots-20260716T142210Z` without reading or copying the saved application credential.
- Verified all SHA-256 checksums and gzip streams and retained metadata plus concise restore instructions in the owner-only bundle.
- Restored every dump into a uniquely named disposable database, ran table checks, matched source/restored table counts of 22 auth, 111 characters, 313 world, and 30 PlayerBots tables, and matched 100 auth accounts plus 1,000 characters.
- Removed the disposable restore databases through the script's exit cleanup and removed the credential-free one-use backup script after successful verification.
- Reconfirmed no partial backup remained, both game servers remained stopped, MySQL remained healthy and loopback-only, and the private updater directory remained empty.
- Selected a controlled paired-server LAN login and visible-PlayerBots smoke test as the next approval-gated milestone before gameplay customization.

### 2026-07-16 - End-to-end LAN login and PlayerBots smoke test

- Reconfirmed the pinned repositories, verified backup checksums, stopped servers, loopback-only MySQL service, and empty private updater directory before startup.
- Determined from fresh logs that realm ID 1 already advertised `192.168.0.154:8085`; a guarded one-row correction script correctly refused to change the already-correct row and was removed unused.
- Started authserver and worldserver together, verified listeners on ports 3724 and 8085, reached the exact worldserver ready marker, and observed the configured bot population come online.
- Used the unchanged WoW 3.3.5a client with `set realmlist 192.168.0.154`; no PlayerBots-specific client files were needed.
- Verified the owner could log in with `DITRAIN`, create and enter a character, see PlayerBots, invite a bot, and receive the bot's group acceptance.
- Identified account ID 101 `YOACCOUNT` as an accidental account and deleted exactly that account through the supported worldserver `account delete` command; retained `DITRAIN`.
- Recorded the non-blocking stock `Eye of Dar'Khan` missing-waypoint warning for future triage rather than expanding the foundation scope.
- Stopped worldserver and authserver gracefully, verified all four database pools closed, removed only the updater-created temporary credential remnant without reading it, and returned the private temporary directory to empty mode-`700` state.
- Selected review of a proportional native server start/stop/status/log workflow as the next approval-gated operational step before gameplay customization.

### 2026-07-16 - Manual native systemd workflow prepared

- Reconfirmed the pinned parent, reference, and nested module repositories are clean and synchronized; MySQL remains healthy and loopback-only; both game servers remain stopped; the private updater directory remains empty; and the verified baseline backup still passes all checksum and gzip integrity checks.
- Inspected the pinned core's startup scripts and direct `SIGINT`/`SIGTERM` handling, the installed runtime layout, host systemd state, owner permissions, journal access, and current boot-user behavior.
- Selected two small native system services plus a grouping target over plain background scripts, user services, and the repository's broad service-manager framework.
- Defined manual-on-demand startup, auth-before-world ordering, reverse graceful shutdown, execution as `ditrain`, five-minute stop timeouts, and crash-only automatic restart.
- Documented start, stop, status, logs, full and selective restarts, failure behavior, and later boot enable/disable commands in `ops/README.md`.
- Verified the tracked and installed unit definitions byte-match and pass `systemd-analyze verify` without reading private configuration or starting either server.
- The owner installed the three public definitions under `/etc/systemd/system`; their ownership is `root:root` with mode `644`, the target remains disabled, and all three units remain inactive.

### 2026-07-16 - Manual systemd operational smoke test

- Reconfirmed the clean synchronized repository, active loopback-only MySQL service, disabled and inactive AzerothCore target, stopped game processes, and empty mode-`700` private updater directory before startup.
- Started the paired server workflow through `azerothcore.target` with owner-authenticated systemd control while leaving boot startup disabled.
- Verified both services ran as `ditrain`, authserver launched before worldserver, ports 3724 and 8085 listened, and worldserver reached its fresh `(worldserver-daemon) ready...` marker.
- Verified systemd status and journal visibility, no error-severity journal entries, no restart loop, and only the accepted stock `Eye of Dar'Khan` missing-waypoint warning in the fresh error log.
- Stopped `azerothcore.target` and verified systemd reversed the dependency order: worldserver stopped before authserver.
- Worldserver drained 1,844 queued character queries and closed the characters, world, auth, and PlayerBots database pools; authserver then closed its auth database pool.
- Both services exited successfully with status `0`, no restart occurred, ports 3724 and 8085 closed, and no authserver or worldserver process remained.
- Reconfirmed MySQL remained active and bound only to loopback, the private updater directory remained empty and mode `700`, all three AzerothCore units were inactive, and boot startup remained disabled.
- Completed the stable native PlayerBots foundation and selected bounded gameplay-design planning as the next approval-gated phase.

### 2026-07-16 - Pre-development module and local-LLM plan

- Reconfirmed the parent `custom` branch at `a34678c1679f216c892af39e0737bcc3f9ae4a33`, reference `Playerbot` at `52f58186a53399e603c46c24977fe60fcaad7f9d`, and nested PlayerBots module at `93aaea3de19243c09ce9ecb25627dc9671715eed`, all clean and synchronized before documentation changes.
- Reconfirmed MySQL is active with ports 3306 and 33060 bound only to loopback; authserver, worldserver, and `azerothcore.target` are inactive; boot startup remains disabled; and the private updater directory is empty and mode `700`.
- Reverified all four baseline-backup checksums and gzip streams and reconfirmed its recorded `restore_test=passed` result without modifying the bundle.
- Selected `azerothcore/mod-transmog`, `NathanHandley/mod-ah-bot-plus`, and `Hokken/mod-llm-chatter` plus its Chatter Companion addon and Ubuntu bridge as the initial play-before-development stack. No module was cloned, installed, built, configured, or provisioned.
- Selected LM Studio on the Windows RTX 4070 over the private LAN, with Magnum v4 9B `Q4_K_M` as the normal gameplay model and Gemma 4 12B IT `Q4_K_M` as the dependable secondary model. The owner will install both models in LM Studio.
- Chose one serialized inference request, conservative context/output limits, restrained chatter frequency, and measurement of response validity, latency, roleplay quality, repetition, and WoW frame times before increasing load.
- Deferred Any Race/Any Class until a separate compatibility review because the known module requires server DBC changes and a client patch and may interact broadly with PlayerBots and class-specific game systems.
- Selected read-only upstream compatibility review and exact revision pinning as the next step, followed by an approval-gated, one-module-at-a-time implementation beginning with transmogrification if the review supports it.

### 2026-07-16 - Transmog increment prepared through clean build

- Re-read the four governing project documents and reconfirmed parent `custom` at `98106dabe76bdde4df4e8ce85b9cd7f2eb957f5a`, reference `Playerbot` at `52f58186a53399e603c46c24977fe60fcaad7f9d`, and PlayerBots module at `93aaea3de19243c09ce9ecb25627dc9671715eed` before acting.
- Completed the bounded upstream Transmog review and received explicit owner approval for the one-module implementation increment.
- Verified reserved Transmog database entries were absent before backup work. The first corrected dump attempt produced and gzip-verified all four dumps, but its disposable MySQL restore initialization failed safely because the helper pre-created the data directory. Cleanup left no partial backup or temporary restore data.
- Corrected and redeployed `/tmp/codex-transmog-backup-sudo.sh`; syntax validation passes. Unattended sudo was unavailable, so the required restore-tested backup remains incomplete and is the next-session gate.
- Cloned canonical `azerothcore/mod-transmog` into the intentionally ignored modules directory and pinned its clean detached checkout at `33ac64b2c305eb1b6fbc97310a7ecbc30c2ba4ef`.
- Reconfigured the existing reproducible Release build, confirmed CMake discovered `mod-playerbots` and `mod-transmog`, and completed a clean three-job build successfully. Verified Transmog objects and a linked Transmogrification symbol in the new build-directory worldserver.
- Did not install the new binary or template, create or alter runtime configuration, run database updates, start authserver/worldserver, or perform a gameplay smoke test.
- Reconfirmed the installed server hashes still match the pre-Transmog baseline, all AzerothCore units are inactive, the target remains manual, and the private updater directory is empty.
- Recorded the owner's newer roleplay-model candidates for later controlled benchmarking; made no LM Studio, addon, bridge, Chatter, or network changes.

### 2026-07-18 - Transmog increment installed and verified

- Re-read all four governing documents and reconfirmed parent `custom` at `ab3d42afdb579c5557268cfcff3b4756b62ddde2`, pinned local `Playerbot` at `52f58186a53399e603c46c24977fe60fcaad7f9d`, PlayerBots module at `93aaea3de19243c09ce9ecb25627dc9671715eed`, and Transmog at `33ac64b2c305eb1b6fbc97310a7ecbc30c2ba4ef`. Refreshed official PlayerBots refs had advanced, so the already tested pins were preserved rather than mixing an upgrade into this increment.
- Created and independently verified `/mnt/data/wow-server/backups/pre-transmog-20260718T155801Z`: four protected compressed dumps, passing checksums and gzip tests, successful isolated restore and table checks, matching source/restored table and critical row counts, `restore_test=passed`, and complete disposable-data cleanup. AppArmor remained enabled; the successful disposable server used `/var/lib/mysql-files/**`.
- Installed the clean Transmog-enabled Release artifacts and public template through CMake. Created ignored owner-only `transmog.conf` with the collection system, retroactive appearances, and Plus disabled; placed no permanent Transmog NPC.
- Started through `azerothcore.target`, let the built-in updater apply the pinned module SQL, and verified exact auth, character, and world objects plus updater records. Confirmed zero permanent Transmog creature spawns and empty new player-data tables.
- Reached the exact worldserver ready marker with no restart or fatal error. Verified `DITRAIN` login, visible PlayerBots, accepted bot grouping, and working `.transmog 0` / `.transmog 1` hide/show responses. Recorded the pinned module's named-subcommand routing limitation and the owner's lack of temporary-NPC RBAC permission without expanding scope or changing permissions.
- Stopped through the supported target; worldserver stopped before authserver, all five database pools closed, both services exited with status `0`, game listeners closed, and boot startup remained disabled. Removed the updater-created credential remnant without reading it, emptied the mode-`700` runtime temporary directory, and removed the credential-free one-use helpers.

### 2026-07-18 - Transmog live usability validation and session closeout

- Re-read the four governing project documents, reconfirmed parent `custom` and both installed nested-module pins, and preserved the existing Transmog configuration: collection and retroactive collection disabled, Plus disabled, and zero permanent Transmog creature spawns.
- Started authserver and worldserver in isolated temporary tmux consoles using the same installed binaries, private configuration paths, working directory, owner account, and file mask as the verified services. This provided the otherwise unavailable live worldserver console for bounded RBAC grant/revoke and closeout; it did not install, enable, or alter the systemd workflow.
- Made `DITRAIN` the permanent owner GM at security level 2 for all realms through the supported worldserver command. Revoked the three diagnostic direct grants after the standard GM role was active. Confirmed a fresh live session reported `SEC_GAMEMASTER`; made no PlayerBots-specific RBAC change.
- Confirmed `.npc add temp 190010` still returned generic NPC usage despite effective GM access and the expected role links. Used the installed module spell path instead: target self and run `.cast self 200100` to summon the player-owned Ethereal Warpweaver `190011` with the normal `npc_transmogrifier` gossip interface.
- Added Grunt Vest (`3752`) and Banshee Armor (`5420`) to the level-1 owner character with GM commands. Applied Banshee Armor's appearance to the equipped Grunt Vest from the bag source, verified it through unequip/re-equip and a fresh login, removed the Transmog through gossip, and verified the original Grunt Vest appearance through another unequip/re-equip and fresh login. The owner reported both stages as clean tests.
- Placed no permanent NPC and made no PlayerBots behavior or configuration change. The owner deferred permanent placement and was given `.npc add 190010` for later placement plus `.npc delete` for removal.
- Saved all players, gracefully stopped worldserver, drained 2,057 queued character queries, closed the characters/world/auth/PlayerBots pools, then gracefully stopped authserver and closed its auth pool. Confirmed only loopback MySQL remained listening, ports 3724/8085 were closed, all AzerothCore units were inactive, boot startup remained disabled, runtime temporary storage was empty and mode `700`, and no test tmux session remained.
- Reconfirmed parent `custom` was clean and synchronized with `origin/custom` at `77cfbeb7bccae71c49b9c2fd3e4643d1ae2c6b95` before this documentation update. Selected `NathanHandley/mod-ah-bot-plus` as the next bounded module review; upstream `master` resolved to `f685832994c825f90aa5a3dc0e1620aa568e875b` at closeout. No AH module clone, build, database mutation, character creation, or configuration occurred.
