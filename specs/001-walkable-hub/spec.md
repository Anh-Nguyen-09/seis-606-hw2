# Feature Specification: Walkable Academy Hub

**Feature Branch**: `001-walkable-hub`

**Created**: 2026-09-23

**Status**: Draft

**Input**: User description: "Build a walkable hub screen where the player moves a character with arrow keys or WASD in real time, bounded to the map. Walking near a building (Academy Hall, Spirit Beast Stables, Archives, Market Row, Training Grounds) shows a popup with its name and a one-line description, and hides it when walking away. The character cannot walk through buildings."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Walk the academy town (Priority: P1)

The player opens the hub and steers their character around the academy town in real time using
either the arrow keys or WASD. The character moves smoothly while a key is held, can move
diagonally, and stops at the edges of the map instead of leaving the visible town.

**Why this priority**: Movement is the foundation of the hub; without it nothing else can be
explored or discovered.

**Independent Test**: Open the hub, hold each direction key (both arrow and WASD sets) and
diagonal combinations, and push the character into every edge of the map. Delivers a playable,
explorable town even before buildings react.

**Acceptance Scenarios**:

1. **Given** the hub has just loaded, **When** the player holds the Right Arrow or D key,
   **Then** the character moves right continuously until the key is released.
2. **Given** the character is standing still, **When** the player holds W and D together,
   **Then** the character moves diagonally up-right at the same overall speed as straight
   movement.
3. **Given** the character is at the left edge of the map, **When** the player holds Left
   Arrow, **Then** the character stays fully inside the map and does not move off-screen.
4. **Given** the player is holding a movement key, **When** the key is released, **Then** the
   character stops immediately with no drift.
5. **Given** the player is using arrow keys, **When** they move the character, **Then** the
   page itself does not scroll.

---

### User Story 2 - Discover buildings by walking near them (Priority: P2)

As the player approaches any of the five buildings — Academy Hall, Spirit Beast Stables,
Archives, Market Row, Training Grounds — a themed popup appears showing that building's name
and a one-line description. When the player walks away, the popup disappears.

**Why this priority**: Discovery is the purpose of exploring the hub, but it depends on
movement (Story 1) existing first.

**Independent Test**: Walk the character up to each building in turn and then away again;
confirm the popup shows the correct name and description each time and hides once the
character has left.

**Acceptance Scenarios**:

1. **Given** the character is away from all buildings, **When** it walks to within the
   discovery range of the Archives, **Then** a popup appears showing "Archives" and its
   one-line description.
2. **Given** the Archives popup is showing, **When** the character walks out of discovery
   range, **Then** the popup hides.
3. **Given** the Academy Hall popup is showing, **When** the character walks directly into the
   range of a different building, **Then** the popup updates to that building's name and
   description.
4. **Given** the character stands still inside a building's range, **When** time passes,
   **Then** the popup stays visible and does not flicker.

---

### User Story 3 - Buildings are solid (Priority: P3)

The character cannot pass through any building. Walking into a building stops the character at
its edge, while movement along the building's side (sliding) remains possible so the player
does not get stuck.

**Why this priority**: Solidity makes the town feel physical, but the hub is already usable
and discoverable without it.

**Independent Test**: Walk into each side of each building, including at diagonals; confirm
the character never overlaps the building and can slide along its walls.

**Acceptance Scenarios**:

1. **Given** the character is walking toward the front of the Market Row, **When** it
   reaches the building, **Then** it stops at the edge and does not overlap the building.
2. **Given** the character is pressed against a building wall, **When** the player holds a
   diagonal that includes a direction parallel to that wall, **Then** the character slides
   along the wall instead of freezing.
3. **Given** the character is at any building, **When** the player tries every direction,
   **Then** there is no position from which the character can end up inside the building.

---

### Edge Cases

- Opposite keys held at once (e.g., Left and Right): movement on that axis cancels out.
- The browser window loses focus while a key is held: the character stops and does not keep
  walking when focus returns.
- The character is inside the discovery range of two buildings at once: the popup shows the
  nearest building only.
- A building sits near the map edge: the character can still reach its discovery range.
- The character's starting position is clear of all buildings and inside the map.
- Displays with different refresh rates: the character moves at the same speed on all of them.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The hub MUST show a top-down view of the academy town with five buildings:
  Academy Hall, Spirit Beast Stables, Archives, Market Row, and Training Grounds.
- **FR-002**: Players MUST be able to move the character up, down, left, and right using
  either the arrow keys or W/A/S/D, with the two key sets behaving identically.
- **FR-003**: The character MUST move continuously in real time while a direction key is held
  and stop as soon as all direction keys are released.
- **FR-004**: Diagonal movement MUST be supported and MUST NOT be faster than straight
  movement.
- **FR-005**: Movement speed MUST be consistent regardless of the display's refresh rate.
- **FR-006**: The character MUST remain fully within the map boundaries at all times.
- **FR-007**: The character MUST NOT overlap any building's footprint; blocked movement on one
  axis MUST NOT prevent movement on the other (wall sliding).
- **FR-008**: When the character comes within discovery range of a building (roughly one
  character-width from its edge on any side), a popup MUST appear showing that building's
  name and one-line description.
- **FR-009**: When the character leaves all buildings' discovery ranges, the popup MUST hide.
- **FR-010**: If the character is within range of more than one building, the popup MUST show
  the nearest one.
- **FR-011**: The popup and all on-screen elements MUST be styled to fit the academy's fantasy
  setting (per the project constitution); placeholder emoji/shapes are acceptable for the
  character and buildings.
- **FR-012**: Arrow keys used for movement MUST NOT scroll the page.
- **FR-013**: Held keys MUST be released automatically when the game window loses focus.
- **FR-014**: An on-screen hint MUST tell the player which keys move the character.

### Key Entities

- **Building**: A named location in the town with a fixed position and size on the map, a
  display name, a one-line description, a solid footprint, and a discovery range around it.
- **Character**: The player's avatar, with a position on the map, a fixed size, and a movement
  speed.
- **Map**: The bounded area of the town that contains the character and all buildings.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The character starts moving as soon as a key is pressed, with no noticeable
  delay (under 0.1 seconds).
- **SC-002**: All 5 buildings can be discovered, with the correct name and description, in a
  single play session of under 1 minute.
- **SC-003**: The popup appears or hides within 0.25 seconds of the character entering or
  leaving a building's discovery range.
- **SC-004**: In a 2-minute test of deliberately walking into every building and map edge, the
  character never overlaps a building or leaves the map (0 occurrences).
- **SC-005**: Crossing the map from one side to the other takes the same time (within 10%)
  on a standard and a high-refresh-rate display.
- **SC-006**: A first-time player can figure out how to move and discover a building without
  instructions beyond the on-screen hint.

## Assumptions

- Keyboard-only desktop play; touch and gamepad controls are out of scope.
- The map is a single fixed-size screen with no scrolling camera.
- The building descriptions are the existing one-line texts from the current prototype.
- The top bar (profile, currencies) is decorative and has no behaviour in this feature.
- Popups only inform; entering buildings or clicking them does nothing in this feature.
- Nothing is saved between visits; every load starts from the same position (per the
  constitution's Stateless principle).
- The whole visible building shape is solid; the player cannot walk "behind" roofs.
- This builds on the existing prototype `academy-town.html`, which already has basic movement,
  proximity popups, and partial collision.
