# UI Contract: Walkable Academy Hub

**Date**: 2026-09-30 | **Plan**: [../plan.md](../plan.md)

The hub has no API. Its interfaces are the keyboard controls offered to the player, the popup
the player sees, and the markup a contributor writes to define a building.

## 1. Controls (player → game)

| Action | Keys (either works) | Key codes |
|--------|---------------------|-----------|
| Move up | Up Arrow, W | `ArrowUp`, `KeyW` |
| Move down | Down Arrow, S | `ArrowDown`, `KeyS` |
| Move left | Left Arrow, A | `ArrowLeft`, `KeyA` |
| Move right | Right Arrow, D | `ArrowRight`, `KeyD` |

Guarantees:

- Movement lasts exactly as long as a key is held; release stops it with no drift.
- Two perpendicular directions move diagonally at the same overall speed as straight movement.
- Opposite directions held together cancel on that axis.
- The two key sets are interchangeable and can be mixed.
- Movement keys never scroll the page.
- Presses combined with Ctrl, Alt, or Cmd are left to the browser and do not move the character.
- Losing window focus releases all held keys.
- No other key, and no mouse or touch input, does anything.

## 2. Popup (game → player)

Element: `#dialog`, containing `#dialogTitle` and `#dialogDesc`.

| State | Condition | Presentation |
|-------|-----------|--------------|
| Hidden | No building within 40 px of the character | No `show` class; not visible, not interactive |
| Shown | A building is within 40 px | `show` class; title = building `name`, body = building `desc` |

Guarantees:

- Shows the nearest building when more than one is in range.
- Appears and disappears within 0.25 s of entering or leaving range.
- Does not flicker or re-animate while the character stays in the same building's range.
- Never receives pointer events and needs no dismissal.
- Styled with the shared palette variables and fonts only.

## 3. Building markup (contributor → game)

A building is one element inside `#frame`:

```html
<div class="building"
     data-name="Archives"
     data-desc="Records of past students, lore of the twelve houses, and old contracts."
     style="left:70px; top:400px; width:140px; height:110px;">
  <div class="tag">ARCHIVES</div>
  <div class="roof"></div>
  <div class="base"></div>
</div>
```

| Part | Required | Meaning |
|------|----------|---------|
| `class="building"` | yes | Registers the element as solid and discoverable |
| `data-name` | yes | Popup title |
| `data-desc` | yes | Popup one-line description |
| inline `left`, `top`, `width`, `height` in `px` | yes | Solid box in map coordinates |
| `.tag`, `.roof`, `.base` children | visual only | Placeholder art; the tag is not solid |

Rules for a valid building:

- The box lies fully inside the 900×600 map.
- The box does not cover the character's start box (420, 240, 40×40).
- At least 40 px of walkable space touches its discovery range, so it can be discovered.

Buildings are read once when the page loads; changing them afterwards has no effect.

## 4. Hint (game → player)

A themed plaque is always visible and names both key sets (FR-014). It has no behaviour.
