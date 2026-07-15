# Project Principles

## Purpose

This document records the owner's durable program-management, engineering, and design preferences. Apply it to implementation plans, architecture choices, mechanics, content systems, reviews, and recommendations.

## Operating Standard

The project favors work that is:

- Clean: readable, cohesive, explicit, and easy to reason about
- DRY: shared knowledge and behavior have an appropriate single source of truth
- Modular: responsibilities are isolated behind narrow interfaces
- Efficient: the result uses project time, runtime resources, operational effort, and maintenance capacity well
- Maintainable: changes are understandable, testable, reversible, and compatible with upstream evolution

These are decision criteria rather than slogans. Apply them in balance:

- Do not introduce an abstraction until it clarifies a real responsibility or removes meaningful duplication.
- Do not merge superficially similar behavior when the concepts may evolve independently.
- Do not optimize runtime behavior without evidence when the optimization harms clarity or flexibility.
- Do not pursue architectural purity at the cost of delivering and testing a playable system.

## Two-Lens Decision Framework

Use both lenses for every meaningful mechanism, property, mechanic, architecture choice, or project decision.

### Lens 1: First Principles

Establish the reasoning from the ground up:

1. What problem are we solving?
2. What player, operator, or developer outcome do we want?
3. What constraints and invariants are real?
4. What assumptions are we making, and how can we test them?
5. What is the simplest mechanism that could produce the outcome?
6. What interactions, edge cases, failure modes, and unintended incentives might result?
7. What evidence would show that the decision works?

Do not begin with a preferred implementation and work backward to justify it.

### Lens 2: Best Practices

Validate the first-principles result against relevant proven practice:

- General software engineering and operational practice
- Established game-design principles
- AzerothCore architecture, module, database, configuration, scripting, build, and contribution conventions
- PlayerBots and RandomBots architecture, configuration, AI behavior, compatibility requirements, and supported workflows
- Existing community modules and solutions

Inspect current repository code and documentation before claiming that something is an AzerothCore or PlayerBots best practice. When authoritative guidance is absent, label conclusions as inference and explain the evidence.

### Resolving Tension Between the Lenses

Best practices are defaults informed by accumulated experience, not substitutes for reasoning. First-principles designs are hypotheses, not permission to ignore known constraints.

When the lenses disagree:

1. State the disagreement explicitly.
2. Identify the project-specific benefit of departing from convention.
3. Identify compatibility, maintenance, balance, operational, and upstream-sync costs.
4. Prefer the smallest reversible experiment that can resolve uncertainty.
5. Record the decision and its validation criteria.

## Program Management Preferences

- Build and preserve a playable, stable foundation before broad customization.
- Work in small, cohesive, testable increments.
- Define the outcome and acceptance criteria before implementation.
- Prefer reversible decisions while uncertainty is high.
- Surface dependencies, risks, assumptions, and opportunity costs early.
- Avoid scope expansion without explaining its value and downstream cost.
- Reuse proven community work before creating custom systems.
- Keep project state, design intent, implementation, configuration, and operational documentation synchronized.
- Capture durable lessons and decisions; exclude routine transcripts and noise.
- Commit coherent checkpoints and push verified work regularly.

## Engineering Preferences

- Prefer composition and isolated modules over invasive core modification.
- Keep interfaces narrow and dependencies explicit.
- Use configuration and data-driven behavior where they remain clear and safe.
- Give each rule or piece of domain knowledge an appropriate source of truth.
- Include error handling, observability, and operational recovery in the design rather than treating them as cleanup.
- Test at the lowest practical level and verify important behavior in the integrated server.
- Optimize based on measured bottlenecks and player-facing impact.
- Preserve compatibility with official PlayerBots updates unless a documented project benefit justifies divergence.

## Decision Record Standard

For a consequential decision, record enough information to reconstruct why it was made:

- Context and problem
- Desired outcome and acceptance criteria
- Constraints and assumptions
- Options considered
- First-principles assessment
- Relevant best practices and repository evidence
- Selected option and tradeoffs
- Validation plan
- Reversal or migration path

Use the most relevant living document rather than creating paperwork for its own sake. Gameplay decisions belong in `GAME_DESIGN.md`; operational state and lessons belong in `PROJECT_STATE.md`; stable project-wide preferences belong here.
