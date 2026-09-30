# Implementation Plan: Walkable Academy Hub

**Branch**: `001-walkable-hub` | **Date**: 2026-09-30 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-walkable-hub/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Turn the existing prototype `academy-town.html` into a hub that fully meets the spec: real-time
keyboard movement at a refresh-rate-independent speed, the character always inside the map,
all five buildings solid across their whole footprint with wall sliding, and a themed popup
that shows the nearest building within one character-width and hides on leaving.

The prototype already has the scene, the five buildings with their descriptions, the popup, and
a `requestAnimationFrame` loop. The work is a rewrite of the inline script (input, movement,
collision, discovery) plus small CSS fixes, all inside the same single file. See
[research.md](./research.md) for each decision and the prototype gaps it closes.

## Technical Context

**Language/Version**: HTML5, CSS3, JavaScript (ES2020) as supported by current evergreen browsers

**Primary Dependencies**: None. Google Fonts (Cinzel, Quicksand) loaded by URL as an optional
static asset with `serif` / `sans-serif` fallbacks

**Storage**: N/A (Principle V: stateless; every load starts from the same state)

**Testing**: Manual browser verification following [quickstart.md](./quickstart.md); no test
framework (Principle I forbids a package-install step)

**Target Platform**: Desktop Chrome, Firefox, Safari, Edge (current versions), keyboard input;
runs from `file://` or any static file server

**Project Type**: Static single-page browser game (one HTML file with inline CSS and JS)

**Performance Goals**: Movement starts within 0.1 s of key press (SC-001); popup shows/hides
within 0.25 s (SC-003); smooth at the display's refresh rate with equal speed at 60 Hz and
120 Hz+ (SC-005)

**Constraints**: No backend, build step, or third-party library; no persistence; must work when
opened directly from disk (so no ES modules, which browsers block over `file://`); remains
playable if the web fonts fail to load

**Scale/Scope**: One screen, a fixed 900×600 map, 1 character, 5 buildings, 1 file of roughly
400 lines

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Checked against constitution v1.0.1.

| Principle | Gate | Status |
|-----------|------|--------|
| I. Static Web Only | Plain HTML/CSS/JS, no server, bundler, or install step; opening `academy-town.html` runs the game; only external resource is a web font with fallbacks | PASS |
| II. Bite-Sized Features | Three independently testable stories in one file; each is a single-sitting change; spec states browser verification per story | PASS |
| III. Always Runnable | Work is ordered so each story leaves the game loadable, movable, and discoverable; quickstart includes the pre-commit smoke test | PASS |
| IV. Fantasy-First Presentation | Popup and hint use the existing parchment/wood/gold palette variables and Cinzel/Quicksand fonts; no new ad-hoc colors; no default browser controls | PASS |
| V. Stateless for Now | No localStorage, cookies, or accounts; fixed start position on every load | PASS |
| Code Organization | Stays a single file; no third-party libraries | PASS |
| Gameplay | `requestAnimationFrame` loop; discovery is proximity-based | PASS |

**Initial gate**: PASS, no violations.

**Post-design re-check** (after Phase 1): PASS. The design adds no files to the runtime, no
dependencies, and no stored state. The one presentation change beyond the popup (restyling the
key hint as a themed plaque) exists to satisfy Principle IV.

## Project Structure

### Documentation (this feature)

```text
specs/001-walkable-hub/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/
│   └── ui-contract.md   # Phase 1 output (/speckit-plan command)
├── checklists/
│   └── requirements.md  # Spec quality checklist (from /speckit-specify)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
academy-town.html        # Entry file: markup, inline <style>, inline <script> (all changes land here)
ui-mockup.png            # Visual reference for the target look (unchanged)
```

**Structure Decision**: Keep the game as the single file `academy-town.html`. The feature is
about 400 lines, which is still readable in one file, and an inline classic script keeps the
game runnable by double-clicking the file. Within the script, code is grouped into four
sections in this order: constants and state, input, simulation (movement, collision,
discovery), and rendering.

## Complexity Tracking

No constitution violations; nothing to justify.
