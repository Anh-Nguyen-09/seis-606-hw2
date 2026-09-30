# Quickstart: Walkable Academy Hub

**Date**: 2026-09-30 | **Plan**: [plan.md](./plan.md)

How to run the hub and confirm the feature works. Controls and popup behaviour are defined in
[contracts/ui-contract.md](./contracts/ui-contract.md); positions and sizes are in
[data-model.md](./data-model.md).

## Prerequisites

- A current desktop browser (Chrome, Firefox, Safari, or Edge) and a keyboard.
- Nothing to install or build.

## Run

Open `academy-town.html` from the repository root in the browser (double-click it, or on
macOS run `open academy-town.html`).

Optional, to serve it instead: `python3 -m http.server 8000`, then visit
`http://localhost:8000/academy-town.html`.

Open the browser's developer console before testing; it must stay free of errors throughout.

## Smoke test (before every commit)

1. The page loads with no console errors.
2. Hold any arrow key: the character moves.
3. Walk to any building: its popup appears; walk away: it hides.
4. Everything on screen looks like part of the academy setting.

## Scenario 1: Movement (User Story 1)

| Step | Expected |
|------|----------|
| Hold Right Arrow, then D | Character moves right at the same speed for both; stops the instant the key is released |
| Hold each of the other six movement keys in turn | Character moves in the matching direction |
| Hold W and D together, compare with D alone | Moves diagonally up-right; not visibly faster than straight |
| Hold Left Arrow and Right Arrow together | Character does not move horizontally |
| Walk into each of the four map edges and hold the key | Character stays fully visible inside the map |
| Make the window shorter than the page, hold Up/Down Arrow | Page does not scroll |
| Make the window smaller than 900×600 | Whole map shrinks to fit; character still cannot leave it |
| Hold D, switch to another app, release D there, switch back | Character is standing still |
| Press Cmd/Ctrl+R | Page reloads; character is back at the centre start position |

## Scenario 2: Discovery (User Story 2)

| Step | Expected |
|------|----------|
| Walk toward the Archives and stop about one character-width away | Popup shows "Archives" and its description |
| Walk away | Popup hides in about a quarter of a second |
| Visit all five buildings | Each shows its own name and description (see the table in data-model.md); doable in under a minute |
| Approach one building from the left, right, above, and below | Popup appears at about the same distance on every side |
| Stand still in range for ten seconds | Popup stays, no flicker |
| Walk from one building straight to another | Popup hides in the gap, then shows the new building |

## Scenario 3: Solid buildings (User Story 3)

| Step | Expected |
|------|----------|
| Walk into each side of each building and keep holding the key | Character stops touching the wall, no overlap, no gap, including over the roof |
| While pressed against a wall, hold a diagonal that points into it | Character slides along the wall |
| Slide past a building's corner | Character rounds the corner without sticking |
| Spend two minutes pushing into every building and edge from all directions | Character is never inside a building or outside the map |

## Scenario 4: Speed on different displays (SC-005)

Time a walk from the left edge to the right edge along the horizontal path. It should take
about 3.6 seconds. If a high-refresh-rate display is available, repeat there; the two times
should be within 10% of each other.

## Done when

All four scenarios pass in at least one browser, the smoke test passes, and the console shows
no errors.
