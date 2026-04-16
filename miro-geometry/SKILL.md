---
name: miro-geometry
description: Guide to Miro board coordinate system, item positioning, sizing, and layout. ALWAYS use this skill before placing any items on a Miro board using the Miro MCP tools or miroctl CLI — required to avoid overlap, frame containment errors, and coordinate mistakes.
---

# Miro Geometry & Positioning

## Coordinate System

Miro uses a **Cartesian coordinate system** with the board center at `(0, 0)`:

```
         -Y (up)
          |
-X -------+------- +X (right)
          |
         +Y (down)
```

- `x` increases to the **right**
- `y` increases **downward** (opposite to standard math convention)
- All units are in **pixels** at 100% zoom

## Item Origin Point

**All item coordinates refer to the item's center**, not its top-left corner.

To place an item at a specific top-left position:
```
center_x = desired_top_left_x + (width / 2)
center_y = desired_top_left_y + (height / 2)
```

Example — place a 400×200 item with its top-left at (100, 100):
```
x = 100 + (400 / 2) = 300
y = 100 + (200 / 2) = 200
```

## Typical Item Sizes

These are approximate defaults and useful planning dimensions:

| Item Type     | Typical Width | Typical Height | Notes                        |
|---------------|---------------|----------------|------------------------------|
| Sticky note   | 200           | 200            | Square by default            |
| Card          | 320           | 160            | Wider than tall              |
| Shape         | 200           | 200            | Varies by shape type         |
| Text          | 100–400       | auto           | Height grows with content    |
| Frame         | 1000–4000     | 600–3000       | Highly variable              |
| Diagram       | 800–2000      | 600–1500       | Depends on node count        |
| Document      | 400–800       | 400–1200       | Height grows with content    |
| Table         | 800–1600      | 200–800        | Grows with rows/columns      |

## Spacing Recommendations

Minimum safe gaps between items to avoid visual overlap:

| Content type  | Recommended gap |
|---------------|-----------------|
| Sticky notes  | 50–100 units    |
| Cards         | 100–200 units   |
| Diagrams      | 500–1000 units  |
| Documents     | 300–500 units   |
| Tables        | 500–1000 units  |
| Frames        | 200–500 units   |

## Common Layout Patterns

### Grid layout (e.g. sticky notes)

```
col_width = item_width + gap
row_height = item_height + gap

x = col_index * col_width
y = row_index * row_height
```

Example — 3-column grid of 200×200 stickies with 50px gap:
```
col_width = 250
row_height = 250

Item (0,0): x=0,    y=0
Item (1,0): x=250,  y=0
Item (2,0): x=500,  y=0
Item (0,1): x=0,    y=250
```

### Horizontal row of diagrams

```
Diagram 1: x=0,    y=0
Diagram 2: x=2500, y=0
Diagram 3: x=5000, y=0
```

### Vertical stack of documents

```
Doc 1: x=0, y=0
Doc 2: x=0, y=1000
Doc 3: x=0, y=2000
```

### Two-column layout (diagram + document)

```
Diagram:  x=0,    y=0
Document: x=2500, y=0
```

## Frames

### Frame coordinate behaviour

Frame `x`/`y` refers to the **frame's center** in board coordinates, same as other items.

### Children inside frames

**Coordinate behaviour differs between the Miro MCP and miroctl CLI:**

| Tool | Coordinate system when parent is set |
|------|--------------------------------------|
| Miro MCP (`mcp__miro__*`) | Board-absolute coordinates |
| miroctl CLI | Frame-relative coordinates (origin = frame top-left) |

#### miroctl CLI (frame-relative)

When using miroctl with a `parent.id`, `x`/`y` are measured from the **frame's top-left corner**:
```
item_x = padding + (item_width / 2)       # e.g. 100 + 100 = 200
item_y = padding + (item_height / 2)      # e.g. 100 + 100 = 200
```

Grid layout inside a frame with miroctl:
```
col_width  = item_width + gap             # e.g. 200 + 50 = 250
row_height = item_height + gap            # e.g. 200 + 50 = 250

item_x = padding + col_index * col_width + (item_width / 2)
item_y = padding + row_index * row_height + (item_height / 2)
```

**Do not pass `position.relativeTo`** — miroctl does not support this field and will return a 400 error.

#### Miro MCP (board-absolute)

When using MCP tools with a parent frame, `x`/`y` are in board coordinates. Calculate the frame's bounding box first:
```
frame_left = frame_x - (frame_width / 2)
frame_top  = frame_y - (frame_height / 2)

item_x = frame_left + padding + (item_width / 2)
item_y = frame_top  + padding + (item_height / 2)
```

The item must be within the frame's bounding box or it will visually escape the frame.

### Recommended frame padding

Leave at least **100 units** of padding between a frame edge and any child item center.

## Common Mistakes

### Overlap from ignoring item size

Setting all items to `x=0, y=0` stacks them all on top of each other. Always offset by at least `(item_width + gap)` per column and `(item_height + gap)` per row.

### Children escaping frames

Placing a child item with coordinates outside the frame's bounding box causes it to visually overflow the frame. Always calculate the frame bounds before placing children.

### Y-axis confusion

Miro's Y axis is **inverted** vs standard math. Moving "down" the board means **increasing** Y. A common mistake is using negative Y offsets expecting to move items down.

### Coordinate drift when creating many items

When programmatically creating many items in a loop, cumulative rounding or fixed offsets can cause drift. Use explicit calculated positions rather than incrementing from a previous item's assumed size.

## Quick Reference

| Scenario | Formula |
|----------|---------|
| Item center from top-left | `cx = left + w/2`, `cy = top + h/2` |
| Top-left from item center | `left = cx - w/2`, `top = cy - h/2` |
| Next column position | `x = prev_x + prev_width + gap` |
| Next row position | `y = prev_y + prev_height + gap` |
| Frame inner top-left | `x = frame_x - frame_w/2 + padding`, `y = frame_y - frame_h/2 + padding` |
| Board center | `x=0, y=0` |
