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
Owned card
type OwnedCard = {
  uid: string;
  cardDetailId: number;
  edition: number;
  gold: boolean;
  xp?: number;
  alphaXp?: number | null;
  delegatedTo?: string | null;
  player?: string;
};
Aggregated player holding
type AggregatedCollectionCard = {
  cardDetailId: number;
  gold: boolean;
  edition?: number;
  copies: number;
  totalBcx: number;
  currentLevel: number;
};
Viewer row model
type ViewerCard = {
  id: number;
  name: string;
  color: string;
  rarity: number;
  rarityName: string;
  type: string;
  editionGroup: string;
  editionLabel: string;
  imageUrl?: string;

  owned: boolean;
  gold: boolean;

  currentBcx: number;
  currentLevel: number;

  targetLevel: number;
  requiredBcx: number;
  bcxNeeded: number;

  status: "Not owned" | "Needs BCX" | "Ready" | "Above target";
};
Business logic requirements

All business logic must live in lib/ utilities, not inside React components.

Create reusable utilities for:

normalizeCardMetadata(raw)

normalizeCollection(raw)

aggregateCollectionCards(cards)

getRequiredBcxForLevel(rarity, targetLevel, isGoldFoil)

getCurrentBcxFromOwnedCard(rawCard)

getCurrentLevelFromBcx(rarity, currentBcx, isGoldFoil)

getBcxRemaining(rarity, currentBcx, targetLevel, isGoldFoil)

getTargetLevelForLeague(rarity, leaguePreset)

mergeCatalogWithCollection(catalog, collection, targetConfig)

BCX and level rules
Important rule

All BCX progression logic must be config-driven, not hardcoded throughout the app.

Config files
lib/levelThresholds.ts

Store BCX requirements by:

rarity

foil type

level

Example structure:

export const REGULAR_THRESHOLDS = {
  common: { 1: 1, 2: 5, 3: 14, 4: 30, 5: 60, 6: 100, 7: 150, 8: 220, 9: 300, 10: 400 },
  rare: { 1: 1, 2: 3, 3: 8, 4: 14, 5: 25, 6: 40, 7: 60, 8: 85 },
  epic: { 1: 1, 2: 2, 3: 5, 4: 10, 5: 20, 6: 35 },
  legendary: { 1: 1, 2: 2, 3: 3, 4: 5 },
};
export const GOLD_THRESHOLDS = {
  common: {},
  rare: {},
  epic: {},
  legendary: {},
};
lib/leaguePresets.ts
export const LEAGUE_PRESETS = {
  novice: { common: 1, rare: 1, epic: 1, legendary: 1 },
  bronze: { common: 3, rare: 2, epic: 2, legendary: 1 },
  silver: { common: 5, rare: 4, epic: 3, legendary: 2 },
  gold: { common: 8, rare: 6, epic: 4, legendary: 3 },
  diamondPlus: { common: 10, rare: 8, epic: 6, legendary: 4 },
};
BCX calculation behavior

sum all owned copies into a total BCX-equivalent for the relevant grouping

support regular and gold foil separately

derive current level from total BCX

compare current BCX to target BCX

clamp bcxNeeded to zero minimum

keep all thresholds editable in config

Catalog and edition logic

Show all cards from card metadata and classify them into UI-friendly edition groups.

Implement a mapping layer from raw edition values into:

edition label

edition group

legality group if needed

Do not bind the UI directly to raw API labels only.

Support user-facing labels such as:

Conclave Arcana

Escalation

Rebellion

Chaos Legion

Dice

Untamed

Alpha/Beta

Promo

Reward

Soulbound

Starter

This mapping must be easy to edit later.

Functional requirements
Username input

text input for Splinterlands username

trim whitespace

normalize to lowercase for requests if needed

submit on Enter or button click

optionally persist username in URL query params

Loading states

show loading skeletons while fetching metadata or collection

metadata should load once and cache

collection should fetch on demand

Error states

Show friendly error messages for:

invalid/empty username

user not found

collection unavailable

card metadata unavailable

network timeout

no cards matching current filters

Result statuses

Each card should display one of:

Not owned

Needs X BCX

Ready

Above target

Suggested logic:

Not owned → owned false and currentBCX = 0

Needs X BCX → 0 < currentBCX < requiredBCX

Ready → currentBCX exactly meets target

Above target → currentBCX exceeds target

Table view requirements

Table columns:

Card image

Card name

Edition

Splinter

Rarity

Type

Foil

Owned

Current BCX

Current level

Target level

Required BCX

BCX needed

Status

Support sorting by:

Card name

Edition

Splinter

Rarity

Current BCX

Current level

Target level

BCX needed

Grid view requirements

Each card tile should show:

image

name

edition

rarity badge

splinter badge

current level

target level

current BCX / required BCX

status chip

Card tiles should be compact, readable, and responsive.

API route requirements

Create these routes:

GET /api/cards

Responsibilities:

fetch full card metadata

normalize card objects

add edition labels/groups

add image URLs if possible

cache aggressively with revalidation

GET /api/collection?username=...

Responsibilities:

fetch player collection

normalize response

aggregate by card detail id + foil (+ edition if needed)

return compact shape for merging

Optional GET /api/view?...

Responsibilities:

read normalized catalog

fetch collection

merge data

apply target calculations

This combined route is optional.

Caching requirements
Card metadata

cache with long revalidation window, for example 12 hours

metadata changes infrequently compared with user collections

Player collections

keep dynamic or very lightly cached

Client caching

use React Query or SWR to avoid unnecessary refetching

Performance requirements

avoid repeated metadata fetches

memoize merged/filtered/sorted results

support the full catalog smoothly

use pagination or virtualization if necessary

keep initial page responsive

Accessibility requirements

keyboard accessible chip controls

visible focus states

semantic button and input markup

sufficient contrast for selected/unselected states

clear labels for filters and statuses

Visual design direction

Aim for:

dark, polished, game-adjacent UI

modern dashboard feel

rounded cards and chips

subtle borders and shadows

responsive layout

premium but clean presentation

Suggested styling:

deep charcoal background

muted panels

brighter accent colors for selected filters

compact readable table

elegant card grid

Suggested folder structure
app/
  page.tsx
  api/
    cards/route.ts
    collection/route.ts
    view/route.ts

components/
  UsernameForm.tsx
  TargetModeSelector.tsx
  LevelSelector.tsx
  LeaguePresetSelector.tsx
  FilterPanel.tsx
  SummaryBar.tsx
  ViewToggle.tsx
  SortControl.tsx
  CardResultsTable.tsx
  CardGrid.tsx
  StatusBadge.tsx
  LoadingState.tsx
  ErrorState.tsx

lib/
  splinterlands.ts
  types.ts
  normalize.ts
  aggregate.ts
  calculations.ts
  levelThresholds.ts
  leaguePresets.ts
  editionMapping.ts
  filters.ts
  sorting.ts
  merge.ts

tests/
  calculations.test.ts
  merge.test.ts
  filters.test.ts

public/
  placeholder-card.png
Testing requirements

Add unit tests for:

BCX requirement lookup

current level derivation

BCX remaining calculation

league preset mapping

merge logic between catalog and owned collection

filter logic

sort logic

Tests should focus on deterministic business rules.

README requirements

Include:

what the project does

stack used

local install steps

dev server instructions

deployment to Vercel

cache behavior

editable config locations:

level thresholds

league presets

edition mappings

known limitations

future enhancements

Nice-to-have phase 2 features

Prepare the codebase so these can be added later:

shareable URL filter state

CSV export

market price overlay

deck completion by splinter

modern/wild legality badges

card detail modal

rental comparison

account-to-account comparison

separate delegated vs in-wallet display

Acceptance criteria

The build is complete when:

the app runs locally and deploys to Vercel

the full Splinterlands card catalog is displayed

a user can enter a username and see their collection merged into the catalog

the user can switch between direct level mode and league preset mode

the app correctly shows BCX owned and BCX still needed per card

the app includes a chip-based filter panel with edition, rarity, foil, ownership, and league controls

the app supports table and grid views

the app handles loading, empty, and error states cleanly

the codebase is modular, typed, and easy to extend

utility logic is covered by tests

Final implementation notes

Build for correctness and maintainability first

Keep all progression math in config-driven utilities

Keep API normalization separate from UI

Do not bury business rules inside components

Prefer clean, composable React components

Make the filter panel feel premium and game-friendly

Treat the app as a serious collection planning tool, not a demo
