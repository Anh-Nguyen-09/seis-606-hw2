# Research: Walkable Academy Hub

**Date**: 2026-09-30 | **Plan**: [plan.md](./plan.md) | **Spec**: [spec.md](./spec.md)

The Technical Context had no open NEEDS CLARIFICATION items: the stack is fixed by the
constitution and the starting point is the existing prototype. Research was therefore an audit
of `academy-town.html` against the spec, followed by one decision per gap.

## Prototype audit

| Requirement | Prototype behaviour | Gap |
|-------------|---------------------|-----|
| FR-004 diagonal not faster | Adds 4 px on each axis | Diagonal is about 41% faster |
| FR-005 refresh-rate independent | 4 px per frame | Twice as fast on a 120 Hz display |
| FR-006 stay inside map | Clamped to 900×600, but `.frame` shrinks with `max-width`/`max-height` | On a small window the map is cropped and the character can walk out of view |
| FR-007 solid footprint, sliding | Only the lower 55% is solid; any overlap cancels the whole move | Can walk over roofs; freezes against walls; stops up to 4 px short of a wall |
| FR-008 discovery range | Zone is 20 px left/right, 10 px above, 30 px below | Not one character-width on every side |
| FR-010 nearest building | First match in DOM order | Not nearest |
| FR-012 no page scroll | No `preventDefault` | Arrow keys can scroll the page |
| FR-013 release on blur | No blur handling | Character keeps walking after Alt/Cmd-Tab |
| FR-011 themed UI | Hint is plain translucent white text; buildings show a pointer cursor | Hint is generic; cursor implies a click that does nothing |
| SC-001 instant response | `.player` has `transition: transform 0.1s` (unused today) | Must be removed before rendering via `transform` |

Already correct and kept: the five buildings with their names and one-line descriptions, the
parchment popup, the palette variables, the fonts with fallbacks, the start position
(420, 240), which is clear of every building.

## Decisions

### 1. Time-based movement

- **Decision**: Speed is `240 px/s`. Each frame computes `dt` from the `requestAnimationFrame`
  timestamp, capped at 50 ms, and moves `speed × dt`.
- **Rationale**: 240 px/s equals the prototype's feel at 60 Hz (4 px × 60). The cap limits a
  step to 12 px after a stall or tab switch, far smaller than any building (110 px minimum), so
  the character cannot jump through a wall.
- **Alternatives considered**: Fixed-timestep accumulator (more code, no benefit without
  physics); keeping per-frame pixels (fails FR-005).

### 2. Diagonal speed

- **Decision**: Build a direction vector from the held keys (each axis −1, 0, or 1) and
  normalise it when both axes are non-zero.
- **Rationale**: Satisfies FR-004, and opposite keys cancel to 0 on their axis for free (edge
  case in the spec).
- **Alternatives considered**: Hard-coded 0.707 multiplier (same result, less clear).

### 3. Key handling

- **Decision**: Track held keys in a `Set` keyed by `KeyboardEvent.code` (`ArrowUp`, `KeyW`,
  and so on). Call `preventDefault()` for the eight movement codes only. Ignore key presses
  made with Ctrl, Alt, or Meta held. Clear the set on `window` `blur` and on
  `visibilitychange` when the page is hidden.
- **Rationale**: `code` is unaffected by Shift or Caps Lock, which change `key` from `w` to
  `W` and could leave a key stuck in the prototype's scheme. Skipping modified presses keeps
  browser shortcuts working and avoids the macOS behaviour where `keyup` never fires for a key
  pressed with Cmd. Clearing on blur meets FR-013.
- **Alternatives considered**: `event.key` lower-cased (current approach; stuck-key risk);
  `keyCode` (deprecated).
- **Note**: `code` is positional, so on a non-QWERTY layout "WASD" means the keys in those
  physical positions. Arrow keys work identically everywhere.

### 4. Collision and wall sliding

- **Decision**: Each building's full box (`left`, `top`, `width`, `height`) is solid. Move on
  the X axis and resolve, then move on the Y axis and resolve. Resolving means: if the
  character's box overlaps a building, place it flush against the side it came from. Map
  bounds are applied as a clamp on each axis in the same step.
- **Rationale**: Per-axis resolution gives sliding automatically (a blocked axis does not
  affect the other), leaves no gap at the wall, and cannot end inside a building. The spec
  assumes the whole visible shape is solid.
- **Alternatives considered**: Rejecting the whole move (current; freezes on walls, fails
  FR-007); rejecting per axis without snapping (leaves a gap of up to one step); swept
  collision (unnecessary at a 12 px maximum step).
- **Note**: The name tag above each building sits outside the box and is not solid.

### 5. Discovery range and nearest building

- **Decision**: Measure the gap between the character's box and each building's box
  (`hypot` of the horizontal and vertical gaps, 0 when touching). A building is in range when
  the gap is at most `40 px` (one character-width). The building with the smallest gap is the
  active one; on an exact tie the currently active building stays.
- **Rationale**: Matches FR-008 on every side including corners, and FR-010. With the current
  layout the ranges of the five buildings never overlap (closest pair is Archives and Training
  Grounds, 130 px apart), so nearest-wins is a safeguard for future layouts. All five ranges
  are reachable: the tightest space is 70 px (left of Archives, below Training Grounds, above
  Academy Hall), enough for the 40 px character.
- **Alternatives considered**: Expanded-rectangle overlap test (equivalent except at corners,
  gives no distance for nearest); centre-to-centre distance (unfair to wide buildings);
  hysteresis on leaving (not needed, a stationary character produces a constant result, so the
  popup cannot flicker).

### 6. Popup updates

- **Decision**: Touch the DOM only when the active building changes. When none is active,
  remove the `show` class and leave the text in place so it fades out with content. Shorten
  the transition from 0.25 s to 0.2 s. Add `role="status"` and `aria-live="polite"`.
- **Rationale**: Avoids rewriting text every frame, and keeps the full fade inside the 0.25 s
  limit of SC-003.
- **Alternatives considered**: Updating every frame (current; wasteful); no transition (meets
  timing but loses the feel).

### 7. Rendering

- **Decision**: Keep simulation positions as floats; draw the character with
  `transform: translate(x, y)` using rounded values, with `left: 0; top: 0` in CSS. Remove the
  `transition` on `.player`. Building rectangles are read once at startup.
- **Rationale**: Transforms avoid layout work each frame; rounding keeps the emoji crisp; a
  transition on the character would add visible lag against SC-001.
- **Alternatives considered**: `left`/`top` each frame (works, triggers layout); `<canvas>`
  (would discard the working DOM scene).

### 8. Map size on small windows

- **Decision**: The map is always 900×600 in its own coordinates. Remove `max-width` and
  `max-height` from `.frame`; a small `fitFrame()` function sets a CSS scale of
  `min(1, innerWidth / 900, innerHeight / 600)` on load and on `resize`.
- **Rationale**: Game logic and the visible map always agree, so FR-006 holds in any window
  size, with no change at normal desktop sizes.
- **Alternatives considered**: Cropping (current; character can leave view); pure CSS scaling
  (needs unit division in `calc()`, not yet supported in all target browsers); reading live
  size into the logic (building positions are fixed pixels, so it would not help).

### 9. Building data source

- **Decision**: Keep the buildings in the HTML with `data-name`, `data-desc`, and inline
  position and size; the script reads them at startup.
- **Rationale**: Already in place, one source of truth, adding a building is one block of
  markup.
- **Alternatives considered**: A JS array that generates the markup (more change for no gain
  at five buildings).

### 10. Themed hint and cursor

- **Decision**: Restyle the hint as a small plaque (parchment background, wood-dark border,
  ink text, existing variables) showing `Move: Arrow keys or W A S D`, placed top-centre
  between the profile and the currencies. Remove `cursor: pointer` from buildings.
- **Rationale**: Principle IV and FR-011/FR-014. The current bottom-right position is partly
  covered by the popup; top-centre is clear of the popup, tags, and paths.
- **Alternatives considered**: Leaving it bottom-right (overlapped when the popup shows);
  hiding the hint after first movement (extra state, not requested).

### 11. File layout

- **Decision**: Everything stays in `academy-town.html` with a classic inline `<script>`.
- **Rationale**: Principle I requires that opening the file is enough. ES modules do not load
  over `file://`, and external `.js`/`.css` files add nothing at this size.
- **Alternatives considered**: `<script type="module">` (needs a server); separate files
  (premature).

## Known limitation (accepted)

The popup is anchored at the bottom centre (as in `ui-mockup.png`) and covers roughly
y 510–582 between x 170 and 730. A character standing directly below the Training Grounds is
hidden behind the popup while it shows. The spec does not ask for the popup to move, so this
is left as is; moving the popup to the top when the character is in the bottom strip is a
candidate for a later feature.
