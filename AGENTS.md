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
