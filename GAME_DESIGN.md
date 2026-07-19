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
- Spend meaningful time playing the stable foundation with carefully selected quality-of-life and living-world modules before beginning custom system development.
- Make AI companions and selected world inhabitants feel distinct and socially present through lore-grounded dialogue, persistent identity, remembered shared events, and restrained ambient conversation.

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

## Pre-Development Play Stack

The first play period should add convenience and social texture without yet changing the project's combat, progression, encounter, or loot foundations:

- Transmogrification provides appearance choice with low expected gameplay-system risk.
- Auction Bot Plus supplies a useful private-realm economy without relying on a large human population. Four dedicated ordinary characters share seller and buyer duties so market names remain believable without consuming PlayerBots or normal playable identities.
- LLM chatter provides two complementary experiences through one system: occasional ambient conversations among PlayerBots and eligible nearby NPCs, and deeper recurring companion dialogue shaped by persistent personalities, backstories, and memories.
- Dialogue generation does not control combat, navigation, quests, loot, economy actions, or database mutation. Existing AzerothCore and PlayerBots systems remain authoritative for gameplay.
- Chatter should favor memorable, divergent exchanges over message volume. Tune triggers, cooldowns, scene length, and concurrency to protect immersion, desktop gaming performance, and readable chat.
- The normal local model is Magnum v4 9B `Q4_K_M`; Gemma 4 12B IT `Q4_K_M` is the dependable secondary option. Model selection remains reversible and should be based on integrated play evidence rather than prose demonstrations alone.

## Household Auction Economy

The auction house should primarily support playing, gathering, and crafting rather than function as an unlimited equipment vendor.

- Maintain four dedicated ordinary market identities in one shared seller/buyer pool. Their purpose is visible economic activity; they are not playable characters, PlayerBots, RandomBots, or GM identities.
- Keep 800 baseline listings in each Alliance, Horde, and neutral auction house, with a 1,000-listing ceiling and slow 25-item maintenance cycles.
- Weight the seller catalog heavily toward profession inputs and outputs: cloth, leather, ore and stone, herbs, enchanting materials, gems, recipes, glyphs, and related goods.
- Do not list poor-quality items. White listings are acceptable only in explicitly profession-oriented categories. Ordinary white weapons, armor, consumables, containers, ammunition, quest items, keys, and miscellaneous goods remain excluded.
- Equipment should be mostly uncommon with a smaller rare share. Epic weapons and armor, legendary items, artifacts, and heirlooms are excluded. Epic gems, trade goods, and recipes may appear at very low weight because they directly serve crafting.
- Profession supplies matter more than equipment coverage at levels 1-20. Green and blue equipment should remain available across the broader leveling curve without requiring exact equal counts in every band.
- Buyer behavior should reward sensibly priced player gathering and crafting while avoiding inflation: one candidate per five-to-ten-minute cycle, a conservative price threshold, vendor-price protection, and no bidding wars against players.
- Seller filters and buyer policy are separate. The bot may buy ordinary player-listed goods that it would not generate itself.
- Validate the economy from live rows after population, not configuration alone: house counts, identity distribution, class/quality mix, forbidden-item counts, profession diversity, level-band coverage, and a real bid or purchase from a normal player.
- Tuning remains reversible through the ignored runtime configuration. Use `.ahbot empty` for bot-owned auction cleanup before module disablement; it does not reverse completed trades or AHBot bids already placed on player auctions.

Any Race/Any Class is a desired later option, not part of the initial play stack. Its client patch and broad class-system interactions require a separate compatibility and player-experience decision before adoption.

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
