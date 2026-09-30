# Data Model: Walkable Academy Hub

**Date**: 2026-09-30 | **Plan**: [plan.md](./plan.md) | **Research**: [research.md](./research.md)

All data lives in memory for the life of the page; nothing is stored (Principle V).
Coordinates are pixels in map space, origin at the map's top-left, x to the right, y down.
Every box is `{ x, y, w, h }` with `x, y` as its top-left corner.

## Map

| Field | Value | Notes |
|-------|-------|-------|
| `w` | 900 | Fixed |
| `h` | 600 | Fixed |

The map is scaled down visually on small windows; its coordinates never change.

## Character

| Field | Type | Initial | Rules |
|-------|------|---------|-------|
| `x` | number (float) | 420 | `0 ≤ x ≤ 900 − 40` (FR-006) |
| `y` | number (float) | 240 | `0 ≤ y ≤ 600 − 40` (FR-006) |
| `size` | constant | 40 | Square box, width and height |
| `speed` | constant | 240 px/s | Same for straight and diagonal (FR-004, FR-005) |

Invariants, true at the end of every frame:

- The character's box is fully inside the map.
- The character's box does not overlap any building's box (touching edges is allowed) (FR-007).

## Building

Read once at startup from each `.building` element (see
[contracts/ui-contract.md](./contracts/ui-contract.md)).

| Field | Type | Source | Rules |
|-------|------|--------|-------|
| `name` | string | `data-name` | Non-empty; shown as popup title |
| `desc` | string | `data-desc` | One line; shown as popup body |
| `x`, `y`, `w`, `h` | number | inline `left`, `top`, `width`, `height` | Whole box is solid; fully inside the map; must not cover the character's start box |

Derived: discovery range = every point within 40 px of the box (FR-008).

Current instances:

| Name | x | y | w | h | Description |
|------|---|---|---|---|-------------|
| Academy Hall | 150 | 70 | 160 | 120 | The central hall where lectures and shared year-one training take place. |
| Spirit Beast Stables | 620 | 90 | 150 | 110 | From year two, students contract with their bonded spirit beast here. |
| Archives | 70 | 400 | 140 | 110 | Records of past students, lore of the twelve houses, and old contracts. |
| Market Row | 660 | 410 | 150 | 110 | Stalls selling potion reagents, gear, and the occasional rare relic. |
| Training Grounds | 340 | 420 | 150 | 110 | Where mages and swordsmen spar to prepare for their next trial. |

## Input state

| Field | Type | Rules |
|-------|------|-------|
| `held` | set of key codes | Contains only movement codes currently held; emptied when the window loses focus or the page is hidden (FR-013) |

Direction per frame: `dx = right − left`, `dy = down − up`, each −1, 0, or 1, where a
direction counts as pressed if either of its two keys is held. If both are non-zero the vector
is normalised.

## Discovery state

| Field | Type | Initial | Rules |
|-------|------|---------|-------|
| `active` | Building or none | none | The nearest building whose gap to the character is ≤ 40 px; none if no building qualifies (FR-008 to FR-010) |

State transitions:

```text
none ──(gap to B ≤ 40)──────────────▶ B      popup shows B
B ────(gap to B > 40, no other)────▶ none   popup hides
B ────(another building C is nearer)▶ C      popup text switches to C
```

On an exact tie the current `active` building is kept. The popup changes only on a
transition, never while `active` is unchanged.

## Per-frame order

1. Compute `dt` (capped at 50 ms) and the direction from `held`.
2. Move on X, clamp to the map, resolve against buildings.
3. Move on Y, clamp to the map, resolve against buildings.
4. Recompute `active`; update the popup if it changed.
5. Draw the character at its new position.
