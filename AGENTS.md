# AzerothCore Playerbots Project

## Foundation

- This repository is the permanent base for the project.
- It uses the mod-playerbots AzerothCore fork and the Playerbot branch.
- Do not replace it with upstream AzerothCore.
- Do not attempt to install mod-playerbots into upstream AzerothCore.
- This project is separate from Segovia.

## Architecture

Prefer solutions in this order:

1. Existing community module
2. Configuration
3. SQL or database changes
4. Existing scripting systems
5. Small isolated custom C++ modules
6. Core modifications only as an absolute last resort

## Program Management and Design Principles

- Use `PROJECT_PRINCIPLES.md` as the durable decision framework for implementation, architecture, mechanics, and project management.
- Evaluate meaningful decisions through both lenses: first-principles analysis and applicable best practices.
- Start with the player or operational problem, constraints, desired outcome, and simplest mechanism that could satisfy them.
- Then validate the proposal against established game-design practice and the documented conventions of AzerothCore and PlayerBots.
- Prefer clean, readable, cohesive code with explicit responsibilities and minimal hidden behavior.
- Apply DRY where duplicated knowledge or behavior would create maintenance risk; do not force abstractions merely to eliminate superficial repetition.
- Favor modular, isolated, replaceable changes with narrow interfaces and minimal core coupling.
- Treat efficiency as total project efficiency: player experience, runtime cost, development time, operational reliability, maintainability, and ability to absorb upstream updates.
- Record consequential decisions, rejected alternatives, tradeoffs, and validation evidence in the appropriate project document.
- Prefer small reversible increments with clear acceptance criteria and verification before expansion.

## Development Rules

- Avoid changes to AzerothCore core files whenever possible.
- Keep custom C++ isolated in focused modules.
- Prefer integration and data changes over new engine systems.
- Inspect existing repository code and documentation before proposing changes.
- Do not assume standard upstream AzerothCore instructions apply to this fork.
- Preserve compatibility with PlayerBots and RandomBots.
- Do not reuse build output created from the previous upstream installation.
- Use Git branches or commits before meaningful changes.
- Show and explain planned changes before performing destructive operations.
- Never import SQL, alter databases, delete files, install packages, or start long builds without explicit approval.
- Do not expose credentials or commit secrets.

## Workflow

- Work one implementation step at a time.
- Stop after each step and report results.
- Do not continue automatically into the next step.
- Prefer commands that are repeatable and scriptable.
- When uncertain, inspect rather than guess.
- Challenge unnecessary complexity.
- Recommend the simplest maintainable architecture.
- Make project changes on `custom`; keep `Playerbot` aligned with `upstream/Playerbot`.
- Treat `origin` as the project's writable fork and `upstream` as the official mod-playerbots repository.

## Documentation and Session Closeout

- Keep code, configuration, operational instructions, and documentation consistent whenever behavior changes.
- Update `PROJECT_STATE.md` when milestones, environment state, architecture, workflow, risks, or next steps change.
- Update `GAME_DESIGN.md` whenever gameplay goals, design decisions, progression, encounter philosophy, content direction, or player-experience assumptions change.
- Update `PROJECT_PRINCIPLES.md` when the decision framework or program-management preferences change.
- Record durable lessons learned in `PROJECT_STATE.md`; omit routine command transcripts and temporary troubleshooting noise.
- Before ending a work session or handing off the project, review all touched files, relevant diffs, and verification results.
- Commit cohesive checkpoints regularly on `custom` rather than accumulating a large mixed change.
- Push completed checkpoints to `origin/custom` after verification unless the user explicitly asks to keep them local.
- At closeout, report the active branch, commit, push status, tests or checks performed, unresolved issues, and the next recommended step.
- Never claim a clean or synchronized repository without checking Git status and the local/remote commit relationship.

## Gameplay Direction

The long-term goal is an EverQuest-inspired Azeroth experience:

- Dangerous outdoor zones
- Slower progression
- Longer fights
- Social aggro and assist chains
- Camps and named enemies
- Group-oriented content
- Strong zone identity
- Meaningful loot progression
- AI party members and a living world
