# Game Design Direction

## Vision

Create a customized AzerothCore PlayerBots distribution for solo and LAN play that gradually reshapes Azeroth into a dangerous, EverQuest-inspired group experience while reusing the existing World of Warcraft world and assets.

This is a game distribution and systems-integration project, not a new MMORPG engine.

## Player Experience Goals

- Make outdoor travel and combat meaningfully dangerous.
- Encourage party composition, preparation, positioning, and cooperation.
- Support solo and small-group play through AI-controlled party members and a living bot population.
- Slow progression enough for zones, equipment, and encounters to remain meaningful.
- Create longer fights with readable tactical decisions instead of rapid enemy deletion.
- Give zones strong identities through spawn ecology, camps, named enemies, mini-bosses, and loot.
- Preserve a playable Azeroth foundation before introducing broad customization.

## Encounter Direction

- Social aggro and assist chains should make enemy placement matter.
- Camps should create deliberate pulls, recovery windows, and positional choices.
- Named enemies and mini-bosses should provide memorable goals and localized loot progression.
- A meaningful portion of the outdoor world should be designed for small parties rather than effortless solo completion.
- Difficulty should reward group success without depending on a large population of human players.

## Progression Direction

- Favor slower, more deliberate character progression.
- Make loot upgrades recognizable and tied to zone or encounter identity.
- Rework spawn ecology and loot progression only after the base PlayerBots server is stable and reproducible.
- Validate one complete slice of customized content before scaling the approach across the world.

## Implementation Constraints

Prefer, in order:

1. Existing community modules
2. Configuration
3. SQL and database changes
4. Existing scripting systems
5. Small isolated custom modules
6. Core changes only when no maintainable alternative exists

PlayerBots compatibility and the ability to absorb official upstream updates take priority over invasive customization.

## Design Decision Method

Every meaningful mechanic, property, system, or content decision receives two passes:

1. **First principles:** Define the intended player experience, the problem being solved, relevant constraints, interactions, failure modes, and the simplest mechanism that can produce the desired behavior.
2. **Best practices:** Compare the result with established game-design practice and with AzerothCore and PlayerBots conventions, limitations, and maintainability requirements.

Neither pass automatically overrides the other. A convention without a reason may not fit this project's goals, while a novel first-principles solution may carry avoidable implementation or balance risks. The selected design should explain the tradeoff when the two lenses point in different directions.

For consequential mechanics, document:

- Intended player behavior and experience
- Inputs, outputs, and affected systems
- Alternatives considered
- Balance, exploit, AI, performance, and maintenance risks
- How the mechanic will be measured or playtested
- Reversal or tuning path if the result underperforms

## Open Design Work

- Define measurable targets for progression speed and combat duration.
- Select the first zone or content slice for end-to-end redesign.
- Define party-size expectations by content type.
- Establish rules for social aggro, assist radius, camps, respawns, named enemies, and mini-bosses.
- Establish the first loot progression model.
- Determine how RandomBots should populate and interact with the world.
