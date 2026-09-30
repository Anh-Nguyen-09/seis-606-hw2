---

description: "Task list for Walkable Academy Hub"
---

# Tasks: Walkable Academy Hub

**Input**: Design documents from `/specs/001-walkable-hub/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/ui-contract.md, quickstart.md

**Tests**: Not requested. Principle I forbids a package-install step, so there is no test framework. Every story ends with a manual browser verification task that follows [quickstart.md](./quickstart.md).

**Organization**: Tasks are grouped by user story. Every task edits the single file `academy-town.html` (repository root), so tasks are sequential by default and no task carries `[P]`. Each phase leaves the game loadable, movable, and discoverable (Principle III).

## Format: `[ID] [Story] Description`

- **[Story]**: US1, US2, US3 (user story phases only)
- All paths are relative to the repository root
- "Console clean" means the browser developer console shows no errors (Quality Gate)

## Path Conventions

- Single file: `academy-town.html` holds markup, inline `<style>`, and one inline classic `<script>` (no ES modules, so it runs from `file://`)
- Script section order (plan.md): constants and state, input, simulation (movement, collision, discovery), rendering

---

## Phase 1: Setup

**Purpose**: Establish the baseline and the target structure before changing behaviour

- [ ] T001 Open `academy-town.html` in a browser (`open academy-town.html`) and confirm the prototype loads with a clean console, the character moves, and popups show; this is the baseline that must stay runnable after every later task
- [X] T002 Add section comment banners inside the `<script>` of `academy-town.html` in this order: `// --- Constants and state ---`, `// --- Input ---`, `// --- Simulation ---`, `// --- Rendering ---`, and move the existing code under the matching banner without changing behaviour

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Time-based game loop and clean data reads that every story builds on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [X] T003 Replace the constants in the `// --- Constants and state ---` section of `academy-town.html` with: `MAP_W = 900`, `MAP_H = 600`, `PLAYER_SIZE = 40`, `SPEED = 240` (px per second), `RANGE = 40` (discovery range in px), and `MAX_DT = 0.05` (seconds); remove the per-frame `SPEED = 4` and `FRAME_W`/`FRAME_H`
- [X] T004 Keep the player state as float `pos = { x: 420, y: 240 }` (data-model.md Character: `0 ≤ x ≤ 900 − 40`, `0 ≤ y ≤ 600 − 40`) and keep the building array read once at startup from `.building` elements (`x`, `y`, `w`, `h` from inline `left/top/width/height`, `name` from `data-name`, `desc` from `data-desc`) in `academy-town.html`
- [X] T005 Rewrite the loop in `academy-town.html` as `function frame(ts)`: on the first call store `ts` and skip; otherwise `dt = Math.min((ts - last) / 1000, MAX_DT)`, call `step(dt)` (empty stub for now), call `render()`, then `requestAnimationFrame(frame)`; start it with `requestAnimationFrame(frame)` (data-model.md Per-frame order, research.md decision 1)
- [X] T006 In the `<style>` of `academy-town.html`, change `.player` to `left: 0; top: 0;` and remove its `transition: transform 0.1s ease` (research.md decision 7, SC-001); remove `left:420px; top:240px;` from the `#player` inline style
- [X] T007 Implement `render()` in the `// --- Rendering ---` section of `academy-town.html` so it sets `player.style.transform = translate(${Math.round(pos.x)}px, ${Math.round(pos.y)}px)`; call it once at startup so the character shows at (420, 240) before the first frame
- [ ] T008 Verify in a browser that the page loads with a clean console and the character appears at the centre start position; movement may be temporarily inert until US1 (Principle III: if movement is inert, complete T009 to T011 in the same sitting before committing)

**Checkpoint**: Loop, state, and rendering are in place; user stories can begin

---

## Phase 3: User Story 1 - Walk the academy town (Priority: P1) 🎯 MVP

**Goal**: Smooth real-time movement with arrow keys or WASD, diagonal at equal speed, bounded to the map, identical speed at any refresh rate

**Independent Test**: Quickstart Scenario 1 and Scenario 4; hold each direction key and diagonals, push into all four map edges, confirm no page scroll

### Implementation for User Story 1

- [X] T009 [US1] In the `// --- Input ---` section of `academy-town.html`, replace the `keys` object and the `e.key.toLowerCase()` listeners with `const held = new Set()` and a `MOVE_KEYS` set of the eight codes `ArrowUp`, `ArrowDown`, `ArrowLeft`, `ArrowRight`, `KeyW`, `KeyA`, `KeyS`, `KeyD`; on `keydown` for a code in `MOVE_KEYS`, ignore the event if `e.ctrlKey || e.altKey || e.metaKey`, otherwise call `e.preventDefault()` and add the code to `held`; on `keyup` for a code in `MOVE_KEYS`, delete it from `held` (FR-002, FR-012, ui-contract.md section 1)
- [X] T010 [US1] Add `window.addEventListener('blur', () => held.clear())` and `document.addEventListener('visibilitychange', () => { if (document.hidden) held.clear(); })` in the `// --- Input ---` section of `academy-town.html` (FR-013)
- [X] T011 [US1] Implement `step(dt)` movement in the `// --- Simulation ---` section of `academy-town.html`: `dx = (right ? 1 : 0) - (left ? 1 : 0)` and `dy = (down ? 1 : 0) - (up ? 1 : 0)` where a direction counts if either of its two codes is in `held`; if both `dx` and `dy` are non-zero divide each by `Math.SQRT2`; move `pos.x += dx * SPEED * dt` and clamp to `0 … MAP_W - PLAYER_SIZE`, then `pos.y += dy * SPEED * dt` and clamp to `0 … MAP_H - PLAYER_SIZE` (X then Y, per data-model.md Per-frame order; FR-003, FR-004, FR-005, FR-006; opposite keys cancel to 0)
- [X] T012 [US1] In the `<style>` of `academy-town.html`, remove `max-width: 100vw` and `max-height: 100vh` from `.frame` and add `transform-origin: top left`; change `body` to `overflow: hidden` with `min-height: 100vh` kept so the page never scrolls (research.md decision 8, FR-012)
- [X] T013 [US1] Add `fitFrame()` to the `// --- Rendering ---` section of `academy-town.html` that computes `scale = Math.min(1, innerWidth / MAP_W, innerHeight / MAP_H)` and sets `frame.style.transform = scale(${scale})` plus a negative margin or wrapper-free centring that keeps the scaled map centred (e.g. set `frame.style.margin` from `(1 - scale) * -MAP_W / 2` horizontally and `(1 - scale) * -MAP_H / 2` vertically); call it at startup and on `window` `resize` (research.md decision 8)
- [ ] T014 [US1] Verify User Story 1 in a browser per quickstart.md Scenario 1 (all eight keys, diagonal, opposite keys, four edges, no page scroll with a short window, window smaller than 900×600, blur while holding D, reload returns to start) and Scenario 4 (left-to-right crossing takes about 3.6 s); console clean

**Checkpoint**: The town is fully walkable and bounded (MVP); buildings still use the prototype popup and are not yet solid on the whole footprint

---

## Phase 4: User Story 2 - Discover buildings by walking near them (Priority: P2)

**Goal**: A themed popup with the nearest building's name and description appears within 40 px of any side of a building and hides when the character leaves

**Independent Test**: Quickstart Scenario 2; visit all five buildings from every side, stand still in range for ten seconds, walk from one building to another

### Implementation for User Story 2

- [X] T015 [US2] In the `// --- Simulation ---` section of `academy-town.html`, add `gap(b)` returning the distance between the player box `{ x: pos.x, y: pos.y, w: PLAYER_SIZE, h: PLAYER_SIZE }` and building `b`: `gx = Math.max(b.x - (pos.x + PLAYER_SIZE), pos.x - (b.x + b.w), 0)`, `gy = Math.max(b.y - (pos.y + PLAYER_SIZE), pos.y - (b.y + b.h), 0)`, return `Math.hypot(gx, gy)` (0 when touching; research.md decision 5)
- [X] T016 [US2] Add `let active = null` to the `// --- Constants and state ---` section and a `findNearest()` function in `// --- Simulation ---` of `academy-town.html` that returns the building with the smallest `gap(b)` among those with `gap(b) <= RANGE` (`RANGE = 40`), returns `null` if none, and on an exact tie keeps the current `active` building (FR-008, FR-010)
- [X] T017 [US2] At the end of `step(dt)` in `academy-town.html`, call `findNearest()`; only when the result differs from `active`, set `active` and call `showDialog(active)` when non-null or `hideDialog()` when null; do not touch the DOM while `active` is unchanged (research.md decision 6, no flicker)
- [X] T018 [US2] Update `showDialog`/`hideDialog` in the `// --- Rendering ---` section of `academy-town.html`: `hideDialog` only removes the `show` class and leaves the text in place so it fades out with content; remove the old per-frame proximity loop and the `overlaps`-based `zone` code from the prototype `update()`
- [X] T019 [US2] In the `<style>` of `academy-town.html`, change the `.dialog` transition from `opacity 0.25s ease, transform 0.25s ease` to `opacity 0.2s ease, transform 0.2s ease` (SC-003), and add `role="status"` and `aria-live="polite"` to the `<div class="dialog" id="dialog">` element
- [X] T020 [US2] Restyle the hint per research.md decision 10 in `academy-town.html`: change `.hint` to a themed plaque positioned top-centre (`top: 14px; left: 50%; transform: translateX(-50%)`, `z-index: 20`), with `background: var(--parchment)`, `border: 2px solid var(--wood-dark)`, `color: var(--ink)`, `border-radius: 8px`, `padding: 4px 12px`, `font-weight: 700`, `font-size: 12px`, using existing variables only; change its text to `Move: Arrow keys or W A S D`; remove `cursor: pointer` from `.building` (FR-011, FR-014, Principle IV)
- [ ] T021 [US2] Verify User Story 2 in a browser per quickstart.md Scenario 2 (each of the five buildings shows its own name and description from data-model.md, appears at about 40 px on every side, hides in about 0.25 s, stays steady for ten seconds, switches when walking to another building); confirm the hint no longer overlaps the popup and looks like part of the academy; console clean

**Checkpoint**: Movement and discovery both work independently of full-footprint collision

---

## Phase 5: User Story 3 - Buildings are solid (Priority: P3)

**Goal**: The whole building box is solid, the character stops flush against walls and slides along them, and can never end up inside a building

**Independent Test**: Quickstart Scenario 3; walk into every side of every building including corners and diagonals, then two minutes of deliberate pushing

### Implementation for User Story 3

- [X] T022 [US3] In the `// --- Simulation ---` section of `academy-town.html`, add `resolveX(prevX, dx)` logic used inside `step(dt)`: after moving and clamping on X, for every building whose full box `{ x, y, w, h }` overlaps the player box (`pos.x < b.x + b.w && pos.x + PLAYER_SIZE > b.x && pos.y < b.y + b.h && pos.y + PLAYER_SIZE > b.y`), if `dx > 0` set `pos.x = b.x - PLAYER_SIZE`, if `dx < 0` set `pos.x = b.x + b.w`; touching edges is allowed (FR-007, research.md decision 4)
- [X] T023 [US3] In `step(dt)` of `academy-town.html`, add the same resolution on Y after the Y move and clamp: on overlap with a building's full box, if `dy > 0` set `pos.y = b.y - PLAYER_SIZE`, if `dy < 0` set `pos.y = b.y + b.h`; a blocked axis must not change the other axis, which gives wall sliding
- [X] T024 [US3] Delete the prototype's partial collision in `academy-town.html` (the `solid` box at 45% height with the 4 px inset, the `blocked` flag, and the whole-move rejection) so only the full-box per-axis resolution from T022 and T023 remains; leave the `.tag` element outside the solid box (it is not part of the building box)
- [ ] T025 [US3] Verify User Story 3 in a browser per quickstart.md Scenario 3 (each side of each building including over the roof, diagonal into a wall slides, corners do not stick, no gap at walls, two-minute stress test with zero overlaps or escapes from the map); also re-run Scenario 2 to confirm all five buildings are still discoverable with solid walls; console clean

**Checkpoint**: All three user stories work together

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final constitution and quality checks across all stories

- [X] T026 Review `academy-town.html` against the constitution: no `localStorage`, `sessionStorage`, cookies, or IndexedDB (Principle V); no new libraries or files (Principles I and Code Organization); only existing `:root` variables used for new styling (Principle IV); the Google Fonts `<link>` remains the only external resource and `serif`/`sans-serif` fallbacks still apply
- [ ] T027 Simulate a failed font load (browser offline mode or block `fonts.googleapis.com`) and confirm the hub is still playable with fallback fonts (Principle I)
- [ ] T028 Run the full quickstart.md "Smoke test" and Scenarios 1 to 4 in one browser (and Scenario 4 on a high-refresh display if available), confirm the console is clean, then commit (Quality Gate, Principle III)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies
- **Foundational (Phase 2)**: Depends on Setup; blocks all user stories
- **US1 (Phase 3)**: Depends on Foundational; delivers the MVP
- **US2 (Phase 4)**: Depends on Foundational; needs `step(dt)` from T011 to hook into, but is testable on its own once the character can move
- **US3 (Phase 5)**: Depends on Foundational and on the axis-by-axis move in T011 (US3 extends it); its check in T025 also reuses US2 discovery
- **Polish (Phase 6)**: Depends on all stories

### User Story Dependencies

- **US1 (P1)**: Independent after Foundational
- **US2 (P2)**: Uses the movement from US1 to be exercised; no code dependency beyond `step`/`render`
- **US3 (P3)**: Modifies the move code written in US1 (T011); do it after US1

### Within Each Story

- Input, then simulation, then rendering, then verification
- The verification task closes each story; do not start the next story before it passes (Principle III)

### Parallel Opportunities

- None of the tasks are marked `[P]`: they all edit `academy-town.html`
- If two people work at once, split by region and merge carefully: T019 and T020 (CSS/markup) can run alongside T015 to T018 (script), and T012 (CSS) alongside T009 to T011 (script)

---

## Parallel Example: User Story 2

```bash
# Different regions of the same file, safe to do in two editors then merge:
Task: "T015-T018: gap(), findNearest(), active-change dialog updates in the <script> of academy-town.html"
Task: "T019-T020: dialog transition, aria attributes, themed hint plaque in the <style>/markup of academy-town.html"
```

---

## Implementation Strategy

### MVP First (User Story 1 only)

1. Phase 1 Setup, Phase 2 Foundational
2. Phase 3 (US1) and verify with T014
3. Commit; the town is walkable, bounded, and runs at the same speed on any display

### Incremental Delivery

1. Add US2 (discovery), verify with T021, commit
2. Add US3 (solid buildings), verify with T025, commit
3. Polish and the final smoke test (T026 to T028)

Each commit must pass the Quality Gate: no console errors, movement works, discovery works, visuals fit the fantasy setting.

---

## Notes

- Every task names its file: `academy-town.html` at the repository root
- Fixed values to use verbatim: map 900×600, character 40×40 starting at (420, 240), speed 240 px/s, discovery range 40 px, `dt` cap 50 ms
- Building data (name, box, description) is already in the markup and must not change (data-model.md table)
- Commit after each verification task
