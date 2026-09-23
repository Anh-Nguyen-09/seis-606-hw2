<!--
Sync Impact Report
- Version change: 1.0.0 → 1.0.1 (PATCH: structural reorganization, no change in substance)
- Restructured into four required top-level headers: Objective, Behavior, Constraints,
  Verification (course format requirement).
- Principle moves (numbering and titles unchanged so existing references still resolve):
  - I. Static Web Only → Constraints
  - II. Bite-Sized Features → Behavior
  - III. Always Runnable → Verification
  - IV. Fantasy-First Presentation → Behavior
  - V. Stateless for Now → Behavior
- Former "Technical Constraints" split: gameplay/runtime items → Behavior; code organization
  and dependency items → Constraints.
- Former "Development Workflow & Quality Gates": workflow → Behavior; quality gate and
  Constitution Check → Verification.
- Former "Governance": supremacy, amendments, versioning → Constraints; compliance review →
  Verification.
- Removed sections: none (all content retained)
- Follow-up TODOs: none
-->

# Objective

Zodiac Academy Hub is a browser-based, top-down exploration game. The player moves a character
in real time around a fantasy academy town and discovers buildings by walking near them.

This constitution defines how the game must behave, the constraints it is built under, and how
every change is verified. It supersedes other project practices; where guidance conflicts, this
document wins. Its principles are numbered I–V and keep those numbers wherever they appear below.

# Behavior

## Gameplay

- Target: current evergreen desktop browsers (Chrome, Firefox, Safari, Edge) with keyboard
  controls; real-time movement MUST feel smooth (driven by `requestAnimationFrame`).
- Building discovery is proximity-based: walking the character near a building MUST be the
  trigger for discovering it.

## IV. Fantasy-First Presentation

- All visible elements — HUD, panels, prompts, buttons, text, and transitions — MUST read as
  part of the academy's fantasy setting (e.g., parchment, wood, crystal, gold, arcane motifs)
  rather than generic web or app UI.
- New visuals MUST reuse the shared palette and typography defined as CSS custom properties
  and fonts, extending them rather than introducing ad-hoc colors or styles.
- Placeholder art (emoji, CSS shapes, simple SVG) is explicitly acceptable until real art is
  ready, provided it is styled to fit the setting.
- Default browser controls and unstyled elements MUST NOT ship in player-facing UI.

Rationale: a consistent atmosphere is the core of the experience, even with placeholder art.

## V. Stateless for Now

- The game MUST NOT implement persistence (localStorage, IndexedDB, cookies, remote storage)
  or user accounts/authentication at this stage.
- Every page load MUST start from a clean, deterministic initial game state.
- Introducing persistence or accounts requires a constitution amendment first.

Rationale: deferring state keeps features simple and avoids premature data-model commitments.

## II. Bite-Sized Features

- Every feature MUST be scoped so it can be built and manually tested in a single sitting.
- A feature that cannot be finished in one sitting MUST be split into smaller, independently
  playable increments before implementation begins.
- Each spec MUST state how the feature is verified in the browser (what to do, what to see).

Rationale: small increments keep momentum high and make regressions easy to locate.

## Development Workflow

- Workflow: specify → plan → tasks → implement, one bite-sized feature at a time (Principle II).

# Constraints

## I. Static Web Only (No Backend, No Build)

- The game MUST be built with plain HTML, CSS, and JavaScript that run directly in a modern
  browser.
- There MUST be no server-side code, backend services, bundlers, transpilers, or package-install
  step required to play or develop the game.
- Opening the entry HTML file (or serving the folder with any static file server) MUST be
  sufficient to run the game.
- External resources are limited to static assets loaded by URL (e.g., web fonts); the game MUST
  remain playable if they fail to load, degrading only in appearance.

Rationale: zero tooling keeps the project approachable, fast to iterate on, and trivially
hostable.

## Code Organization and Dependencies

- Code SHOULD stay organized in a small number of readable files; splitting into additional
  `.js`/`.css` files is allowed only via plain `<script>`/`<link>` tags (ES modules via
  `<script type="module">` are permitted as they need no build step).
- No third-party runtime frameworks or libraries are added unless an amendment justifies them.

## Governance

- Amendments are made by editing this file with a Sync Impact Report, a version bump, and an
  updated Last Amended date, committed on their own or alongside the change that motivated them.
- Versioning follows semantic versioning:
  - MAJOR: removal or backward-incompatible redefinition of a principle.
  - MINOR: a new principle or section, or materially expanded guidance.
  - PATCH: clarifications, wording, or typo fixes.

# Verification

## III. Always Runnable (NON-NEGOTIABLE)

- Every commit MUST leave the game loadable and playable: the page opens without JavaScript
  errors in the console, the character can move, and existing buildings can still be
  discovered.
- Unfinished work MUST NOT be committed in a state that breaks the game; hide it behind
  inert code paths or keep it uncommitted.
- Before committing, the contributor MUST open the game in a browser and perform a quick
  smoke test of movement and discovery.

Rationale: a runnable main branch means any commit can be demoed, bisected, or shared.

## Quality Gate

Before every commit:

1. Game opens in a browser with no console errors.
2. Character movement works.
3. Existing building discoveries still trigger.
4. Any new visuals fit the fantasy setting (Principle IV).

## Compliance Review

- Plans MUST include a Constitution Check confirming compliance with all five principles;
  any deviation MUST be documented and justified in the plan's complexity tracking.
- Every spec, plan, and review MUST check conformance with principles I–V; unjustified
  violations block the change.

**Version**: 1.0.1 | **Ratified**: 2026-09-23 | **Last Amended**: 2026-09-23
