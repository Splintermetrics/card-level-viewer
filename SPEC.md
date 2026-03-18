# Splinterlands Card Level Viewer

## Overview

Build a production-ready website called **Splinterlands Card Level Viewer** using **Next.js + TypeScript + Tailwind CSS**, designed for deployment on **Vercel**.

The app should display **all Splinterlands cards in the game** and allow a user to:

- enter a Splinterlands username
- choose a target card level or league preset
- view every card in the game with their current collection progress
- see how much BCX they own for each card
- see how much BCX is still needed to reach the selected target

The site should behave as a **collection progression viewer**, not just an owned-card list.

---

## Goals

### Primary goal
Help Splinterlands players understand, at a glance, how far their collection is from a chosen target level or league-ready state.

### Secondary goals
- Show the full card catalog, not just owned cards
- Make progression gaps obvious and easy to filter
- Provide a polished, game-adjacent UI suitable for regular use
- Keep progression logic modular and config-driven

---

## Core product behavior

### Full catalog first
- Always load and display the **full Splinterlands card catalog**
- Do **not** limit the results to only cards the entered player owns
- When no username is entered, still show all cards with neutral/default progress data

### Username overlay
When a username is entered:
- fetch that player’s collection
- merge the collection into the full catalog
- show per-card progress against the selected target

For cards the user does **not** own:
- `owned = false`
- `currentBCX = 0`
- `currentLevel = 0` or `1` depending on chosen display convention
- `requiredBCX = full amount needed for selected target`
- `bcxNeeded = requiredBCX`
- `status = "Not owned"`

---

## User story

> As a Splinterlands player, I want to enter my username, choose a target level or league preset, and instantly see for every card in the game whether I already meet that target or how much BCX I still need.

---

## Tech stack

Use the following stack:

- **Next.js App Router**
- **TypeScript**
- **Tailwind CSS**
- **Vercel**
- **React Query** or **SWR**
- **Vitest** or **Jest** for tests

Use **server-side API proxy routes** for Splinterlands API access where possible.

---

## API/data requirements

Use Splinterlands public API data for:

### Card metadata
Load the full catalog including:
- card id
- card name
- splinter/color
- rarity
- type
- edition/set info
- image reference if available
- any level/progression-related metadata that is useful

### Player collection
Load all cards owned by a given username including:
- card detail id
- edition
- foil/gold status
- xp or BCX-related values
- delegated/ownership state if returned

### API architecture
Do not rely on direct browser-to-Splinterlands requests if avoidable. Instead:

- create Next.js API routes that proxy requests
- normalize data server-side
- cache metadata server-side
- keep player collection requests dynamic or lightly cached
- centralize error handling

---

## Main modes

Support **two target modes**.

### 1. Direct level mode
The user manually selects a target level.

Example:
- target level = 4

The app compares each card’s current BCX against the BCX required for that level, based on rarity and foil.

### 2. League preset mode
The user selects a preset league target instead of a raw level.

Presets:
- Novice
- Bronze
- Silver
- Gold
- Diamond+

This mode should map each rarity to a target level using a config-driven preset table.

Example default presets:

- Novice → Common 1, Rare 1, Epic 1, Legendary 1
- Bronze → Common 3, Rare 2, Epic 2, Legendary 1
- Silver → Common 5, Rare 4, Epic 3, Legendary 2
- Gold → Common 8, Rare 6, Epic 4, Legendary 3
- Diamond+ → Common 10, Rare 8, Epic 6, Legendary 4

These values must live in a config file so they can be easily adjusted later.

---

## UI requirements

## Layout

### Header
Show:
- page title: `Splinterlands Card Level Viewer`
- one-sentence helper text
- username input
- target mode selector

### Main controls area
Show a **card-style filter panel** using grouped chip/toggle controls inspired by a segmented modern dashboard layout.

### Summary area
Show summary cards with:
- total cards in catalog
- owned cards
- unowned cards
- cards at/above target
- cards below target
- percentage of catalog meeting target

### Results area
Provide:
- table view
- grid view
- sort control
- result count
- pagination or virtualization if needed

---

## Filter panel requirements

Use a polished grouped chip-based filter panel.

### Visual rules
- grouped rounded chips/buttons
- selected chips clearly highlighted
- unselected chips muted
- headings per group
- support both multi-select and single-select groups
- mobile-friendly layout

### Filter groups

#### Editions
Split visually into:

##### Modern
- Conclave Arcana
- Escalation
- Rebellion

##### Wild
- Alpha/Beta
- Chaos Legion
- Dice
- Untamed

Also provide:
- `All`
- `None`

Edition chips must be **multi-select**.

#### Rarity
Options:
- Common
- Rare
- Epic
- Legendary

Multi-select.

#### League / Target bracket
Options:
- Novice
- Bronze
- Silver
- Gold
- Diamond+

Single-select.

This group should be active in league preset mode and may be hidden or secondary in direct level mode.

#### Foil
Options:
- All
- Regular
- Gold

Single-select.

#### Ownership
Options:
- All
- Owned
- Unowned
- Below target
- At/above target

Single-select.

#### Type
Options:
- All
- Monster
- Summoner

Single-select or multi-select with `All`.

### Optional future exclusions group
Prepare for future support of:
- Exclude Promo
- Exclude Reward
- Exclude Soulbound
- Exclude Starter

### Filter panel actions
Include:
- `Reset filters`
- optional `Collapse filters` on mobile

---

## Data model

Create normalized internal data models.

### Card metadata
```ts
type CardMeta = {
  id: number;
  name: string;
  color: string;
  rarity: number;
  rarityName: "Common" | "Rare" | "Epic" | "Legendary";
  type: "Monster" | "Summoner" | string;
  editionsRaw?: string;
  editionGroup: string;
  editionLabel: string;
  isStarter?: boolean;
  imageUrl?: string;
};
