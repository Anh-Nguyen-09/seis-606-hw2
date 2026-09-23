<!--
Sync Impact Report
- Version change: (unratified template) → 1.0.0
- Modified principles (template placeholder → new title):
  - [PRINCIPLE_1_NAME] → I. Static Web Only (No Backend, No Build)
  - [PRINCIPLE_2_NAME] → II. Bite-Sized Features
  - [PRINCIPLE_3_NAME] → III. Always Runnable
  - [PRINCIPLE_4_NAME] → IV. Fantasy-First Presentation
  - [PRINCIPLE_5_NAME] → V. Stateless for Now
- Added sections: Technical Constraints; Development Workflow & Quality Gates
- Removed sections: none
- Templates: dependent templates/commands read this file at runtime; none modified.
- Follow-up TODOs: none
- Note: template resolved manually (pwsh unavailable); only the core layer exists
  and its sha256 matches .specify/memory/.constitution-template.json.
-->

# Zodiac Academy Hub Constitution

Zodiac Academy Hub is a browser-based, top-down exploration game. The player moves a character
in real time around a fantasy academy town and discovers buildings by walking near them.

## Core Principles

### I. Static Web Only (No Backend, No Build)

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

### II. Bite-Sized Features

- Every feature MUST be scoped so it can be built and manually tested in a single sitting.
- A feature that cannot be finished in one sitting MUST be split into smaller, independently
  playable increments before implementation begins.
- Each spec MUST state how the feature is verified in the browser (what to do, what to see).

Rationale: small increments keep momentum high and make regressions easy to locate.

### III. Always Runnable (NON-NEGOTIABLE)

- Every commit MUST leave the game loadable and playable: the page opens without JavaScript
  errors in the console, the character can move, and existing buildings can still be
  discovered.
- Unfinished work MUST NOT be committed in a state that breaks the game; hide it behind
  inert code paths or keep it uncommitted.
- Before committing, the contributor MUST open the game in a browser and perform a quick
  smoke test of movement and discovery.

Rationale: a runnable main branch means any commit can be demoed, bisected, or shared.

### IV. Fantasy-First Presentation

- All visible elements — HUD, panels, prompts, buttons, text, and transitions — MUST read as
  part of the academy's fantasy setting (e.g., parchment, wood, crystal, gold, arcane motifs)
  rather than generic web or app UI.
- New visuals MUST reuse the shared palette and typography defined as CSS custom properties
  and fonts, extending them rather than introducing ad-hoc colors or styles.
- Placeholder art (emoji, CSS shapes, simple SVG) is explicitly acceptable until real art is
  ready, provided it is styled to fit the setting.
- Default browser controls and unstyled elements MUST NOT ship in player-facing UI.

Rationale: a consistent atmosphere is the core of the experience, even with placeholder art.

### V. Stateless for Now

- The game MUST NOT implement persistence (localStorage, IndexedDB, cookies, remote storage)
  or user accounts/authentication at this stage.
- Every page load MUST start from a clean, deterministic initial game state.
- Introducing persistence or accounts requires a constitution amendment first.

Rationale: deferring state keeps features simple and avoids premature data-model commitments.

## Technical Constraints

- Target: current evergreen desktop browsers (Chrome, Firefox, Safari, Edge) with keyboard
  controls; real-time movement MUST feel smooth (driven by `requestAnimationFrame`).
- Building discovery is proximity-based: walking the character near a building MUST be the
  trigger for discovering it.
- Code SHOULD stay organized in a small number of readable files; splitting into additional
  `.js`/`.css` files is allowed only via plain `<script>`/`<link>` tags (ES modules via
  `<script type="module">` are permitted as they need no build step).
- No third-party runtime frameworks or libraries are added unless an amendment justifies them.

## Development Workflow & Quality Gates

- Workflow: specify → plan → tasks → implement, one bite-sized feature at a time (Principle II).
- Quality gate before every commit:
  1. Game opens in a browser with no console errors.
  2. Character movement works.
  3. Existing building discoveries still trigger.
  4. Any new visuals fit the fantasy setting (Principle IV).
- Plans MUST include a Constitution Check confirming compliance with all five principles;
  any deviation MUST be documented and justified in the plan's complexity tracking.

## Governance

- This constitution supersedes other project practices. Where guidance conflicts, this document
  wins.
- Amendments are made by editing this file with a Sync Impact Report, a version bump, and an
  updated Last Amended date, committed on their own or alongside the change that motivated them.
- Versioning follows semantic versioning:
  - MAJOR: removal or backward-incompatible redefinition of a principle.
  - MINOR: a new principle or section, or materially expanded guidance.
  - PATCH: clarifications, wording, or typo fixes.
- Compliance: every spec, plan, and review MUST check conformance with the Core Principles;
  unjustified violations block the change.

**Version**: 1.0.0 | **Ratified**: 2026-09-23 | **Last Amended**: 2026-09-23
