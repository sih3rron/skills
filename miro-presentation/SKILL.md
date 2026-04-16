---
name: miro-presentation
description: Guide for generating presentation decks in Miro boards using the Miro MCP tools and miroctl CLI. Use when the user wants to create slides, a deck, or a multi-slide presentation inside a Miro board. Covers slide layouts, dimensions, colors, typography, and step-by-step construction patterns reverse-engineered from a real Miro presentation deck.
---

# Miro Presentation Skill

Miro has no native "presentation" or "slides" API — presentations are built from **frames** laid out inside a parent container frame. This skill documents the exact patterns, dimensions, and item structures needed to replicate a professional Miro presentation programmatically using the Miro MCP tools and miroctl CLI.

**Always load the `miro-geometry` skill before placing items** — it defines the coordinate system and frame-relative vs board-absolute rules that govern all placement.

---

## Core Concept: How Miro Presentations Work

A Miro presentation deck is:
1. A **parent container frame** that holds all slides
2. **Child slide frames** nested inside the parent, each representing one slide
3. **Content items** (text, shapes, sticky notes, images, embeds) placed inside each slide frame

Miro's "presentation mode" sequences through all direct child frames of a parent frame in order, making each child a slide.

---

## Slide Dimensions (16:9 Format)

All slides in the reference deck use `format: "ratio_16x9"`. The actual pixel dimensions vary slightly per slide but are consistent within ~1% across the deck:

| Dimension | Value |
|-----------|-------|
| Width | ~476 units (range: 474–494) |
| Height | ~268 units (range: 267–278) |
| Aspect ratio | 16:9 |

**Standard slide size to use when creating new slides:**
```
width: 1920, height: 1080   ← use this for miroctl (full scale)
width: 476,  height: 268    ← approximate internal Miro scale (if you observe existing items)
```

When using `miroctl`, specify `format: ratio_16x9` and the board will handle the ratio. In practice, use `width: 1920, height: 1080` for all slide frames — this is the standard 16:9 HD resolution that Miro maps to `ratio_16x9`.

---

## Brand & Design System

### Colors

| Purpose | Color | Hex |
|---------|-------|-----|
| Yellow accent (title slides, section headers) | Miro yellow | `#ffdd33` |
| White (content slides) | White | `#ffffff` |
| Primary text | Near-black | `#1c1c1e` |
| Teal / highlight accent | Teal | `#47d1c4` |
| Purple card border | Purple | `#ce70fc` |
| Purple card fill | Light lavender | `#f8c8fc` |
| Impact highlight background | Yellow inline | `rgb(255,221,51)` |
| Sticky note background | Pale yellow | `#fff9c4` |

### Typography

| Use | Font | Size | Style |
|-----|------|------|-------|
| Slide title (large) | RoobertPRO | 87–95pt | bold italic |
| Slide title (medium) | RoobertPRO | 72–74pt | bold italic |
| Section label / header bar | RoobertPRO | 25–28pt | bold |
| Body copy | RoobertPRO | 23–33pt | regular |
| Quarter / date indicator | RoobertPRO | 40–41pt | regular |

> **Font fallback**: If RoobertPRO is not available via API, use `open_sans` or another Miro-supported font. Miro API accepts `fontFamily: "roobert_pro"` for brand-aligned output.

---

## Slide Layout Templates

The deck contains **8 distinct layout patterns**. Each is described below with exact item placement logic.

---

### Layout 1: Title Slide (Yellow)

**When to use:** Opening slide, section dividers, major transitions.

**Frame:** `fillColor: #ffdd33`, `format: ratio_16x9`

**Items (frame-relative coordinates):**

| Item | Type | Position (x, y) | Size | Style |
|------|------|-----------------|------|-------|
| Main title | text | x: 136, y: 63 | width: 691 | fontSize: 86, bold italic, color: #1c1c1e |
| Quarter / subtitle | text | x: 431, y: 248 | width: 262 | fontSize: 40, regular, color: #1c1c1e |
| Optional logo/image | image | x: 33, y: 250 | width: 124, height: 46 | Miro logo bottom-left |
| Optional hero image | image | x: 338, y: 134 | width: 538, height: 609 | right-side decorative |

**Construction pattern (miroctl):**
```bash
# 1. Create the slide frame
SLIDE_ID=$(miroctl board items create frame \
  --board-id $BOARD_ID \
  --title "Title Slide" \
  --format ratio_16x9 \
  --fill-color "#ffdd33" \
  --parent-id $PARENT_FRAME_ID \
  --format json | jq -r '.id')

# 2. Add main title text (frame-relative coords)
miroctl board items create text \
  --board-id $BOARD_ID \
  --content "<p><strong><em>Your Title Here</em></strong></p>" \
  --font-family roobert_pro \
  --font-size 86 \
  --color "#1c1c1e" \
  --x 480 --y 220 \
  --width 2400 \
  --parent-id $SLIDE_ID

# 3. Add quarter/date label
miroctl board items create text \
  --board-id $BOARD_ID \
  --content "<p>Q4 2025</p>" \
  --font-family roobert_pro \
  --font-size 40 \
  --color "#1c1c1e" \
  --x 1500 --y 870 \
  --width 900 \
  --parent-id $SLIDE_ID
```

**Construction pattern (MCP):**
```
Use mcp__miro__doc_create or shape tools — but for frames+text the CLI is preferred.
For MCP text placement: x/y are board-absolute. Calculate:
  frame_left = frame_x - frame_width/2
  frame_top  = frame_y - frame_height/2
  item_x = frame_left + item_offset_x + item_width/2
  item_y = frame_top  + item_offset_y + item_height/2
```

---

### Layout 2: Statement / Impact Slide (Yellow)

**When to use:** Single bold statement, section intro, key value prop.

**Frame:** `fillColor: #ffdd33`, `format: ratio_16x9`

**Items:**

| Item | Type | Position (x, y) | Size | Notes |
|------|------|-----------------|------|-------|
| Bold statement | text | centered vertically, left-aligned ~x:100 | width: full-width ~1700 | fontSize: 73–95pt, bold italic |

**Construction notes:**
- Single text item fills most of the slide
- Yellow background with large bold italic text — simple and high-impact
- No other decoration needed

---

### Layout 3: Discussion / Workshop Slide (White + Sticky Notes)

**When to use:** Facilitated discussion questions, workshop prompts, brainstorming prompts.

**Frame:** `fillColor: #ffffff`, `format: ratio_16x9`

**Items:**

| Item | Type | Position | Size | Notes |
|------|------|----------|------|-------|
| "Let's discuss:" header | text | top-left, y: ~80 | width: ~1500 | fontSize: 72pt, bold italic, #1c1c1e |
| Sticky note 1 | sticky_note | x: ~340, y: ~650 | 200×200 (default) | fillColor: #fff9c4, center-aligned text |
| Sticky note 2 | sticky_note | x: ~960, y: ~650 | 200×200 | fillColor: #fff9c4 |
| Sticky note 3 | sticky_note | x: ~1580, y: ~650 | 200×200 | fillColor: #fff9c4 |

**3-column sticky layout formula (frame-relative, miroctl):**
```
sticky_width  = 200
gap           = 80
total_stickies = 3
start_x = (frame_width - (3 * sticky_width + 2 * gap)) / 2 + sticky_width/2

col[0].x = start_x
col[1].x = start_x + sticky_width + gap
col[2].x = start_x + 2*(sticky_width + gap)
y = frame_height * 0.6   ← lower 40% of slide
```

---

### Layout 4: Challenges / Problem Slide (White + 3 Purple Cards)

**When to use:** Listing 3 customer pain points, negative consequences, challenges.

**Frame:** `fillColor: #ffffff`, `format: ratio_16x9`

**Items:**

| Item | Type | Position (frame-relative) | Size | Style |
|------|------|--------------------------|------|-------|
| Title text | text | x: 232, y: 50 | width: 1235 | fontSize: 72pt, bold italic |
| "Negative Consequences" label | text | x: 327, y: 114 | width: 425 | fontSize: 25pt, bold, color: #ffffff (on dark bar) |
| Card 1 (shape) | shape/round_rectangle | x: 90, y: 175 | 408×319 | fillColor: #f8c8fc, borderColor: #ce70fc, borderWidth: 2 |
| Card 2 (shape) | shape/round_rectangle | x: 239, y: 175 | 408×319 | same |
| Card 3 (shape) | shape/round_rectangle | x: 388, y: 175 | 408×319 | same |
| Card 1 text | text | x: 88, y: 172 | width: 321 | fontSize: 32pt, left-aligned, overlaid on card |
| Card 2 text | text | x: 239, y: 172 | width: 348 | fontSize: 32pt |
| Card 3 text | text | x: 388, y: 172 | width: 359 | fontSize: 32pt |

**Card layout formula (3-col equal width, frame-relative):**
```
card_width  = 408
card_height = 319
padding     = 90
gap         = 149   ← gap between card centers
card_y      = 175   ← center-y of cards

card[0].x = 90
card[1].x = 239
card[2].x = 388
```

**Note:** The "Negative Consequences" label uses white text and appears to sit on a dark background bar — create a dark shape behind it or use an inline colored shape.

---

### Layout 5: Solution / Value Prop Slide (White + Header + Body)

**When to use:** Introducing a solution, stating the product value proposition.

**Frame:** `fillColor: #ffffff`, `format: ratio_16x9`

**Items:**

| Item | Type | Position | Notes |
|------|------|----------|-------|
| Main headline | text | top area, large | fontSize: 72pt, bold italic |
| Supporting text / sticky note | sticky_note or text | below headline | fillColor: #fff9c4 for sticky, or plain text |

---

### Layout 6: Case Study Slide (White + Problem/Solution Two-Column)

**When to use:** Customer examples, before/after comparisons, ROI stories.

**Frame:** `fillColor: #ffffff`, `format: ratio_16x9`

**Items:**

| Item | Type | Position (frame-relative) | Size | Notes |
|------|------|--------------------------|------|-------|
| Title | text | x: 221, y: 58 | width: 1142 | fontSize: 74pt, regular |
| Left column header shape | shape | x: 127, y: 119 | 575×110 | "Problem:" header bar |
| Right column header shape | shape | x: 343, y: 119 | 575×110 | "How Miro helps:" header bar |
| Left column body shape | shape | x: 127, y: 150 | 575×288 | main content area left |
| Right column body shape | shape | x: 343, y: 150 | 575×288 | main content area right |
| "Problem:" label | text | x: 149, y: 119 | width: 672 | fontSize: 46pt, bold |
| "How Miro helps:" label | text | x: 313, y: 128 | width: 339 | fontSize: 46pt, bold |
| Left body text | text | x: 120, y: 169 | width: 503 | fontSize: 23pt |
| Right body text | text | x: 343, y: 169 | width: 535 | fontSize: 23pt |
| Impact statement | text | x: 199, y: 238 | width: 989 | fontSize: 38pt, italic, bg: rgb(255,221,51) |

**Two-column layout formula:**
```
frame_width   = 1920 (at full scale)
col_width     = frame_width / 2 - padding
left_col_x    = padding + col_width/2
right_col_x   = frame_width/2 + col_width/2
header_y      = 420   ← ~39% down
body_y        = 560   ← ~52% down
impact_y      = 860   ← ~80% down (full-width, highlighted)
```

**Impact row styling:**
```html
<p><em style="background-color:rgb(255,221,51)">IMPACT: metric here — </em><strong style="background-color:rgb(255,221,51)"><em>key result</em></strong><em style="background-color:rgb(255,221,51)"> across the team.</em></p>
```

---

### Layout 7: SDLC / Feature Overview Slide (Teal Multi-Column)

**When to use:** Product capability overview, workflow stages, feature matrix.

**Frame:** `fillColor: #ffffff`, `format: ratio_16x9`

**Items:**
- Main headline (top, bold italic, ~72pt)
- 4 vertical columns with:
  - Teal header bar (`fillColor: #47d1c4`) with stage name
  - White body area with descriptive text
  - Optional product name bar (teal)
- Logos/images at base of columns
- Connector label "Via Model Context Protocol (MCP)" spanning columns

**4-column formula:**
```
col_width = (frame_width - 2*padding) / 4
col_gap   = 20

col[0].x = padding + col_width/2
col[1].x = padding + col_width + gap + col_width/2
col[2].x = padding + 2*(col_width + gap) + col_width/2
col[3].x = padding + 3*(col_width + gap) + col_width/2
```

---

### Layout 8: Two Use Case Cards (White + 2-Column Cards)

**When to use:** Comparing two equal features, two value propositions, two options.

**Frame:** `fillColor: #ffffff`, `format: ratio_16x9`

**Items:**
- Headline text (top, full width)
- Optional images row (center, 4 images evenly spaced)
- Left card: background shape + header bar shape + title text + body text
- Right card: same structure, right half of slide

**Two-card formula:**
```
card_width  = (frame_width - 3*padding) / 2
card_height = frame_height * 0.35

left_card.x  = padding + card_width/2
right_card.x = padding*2 + card_width + card_width/2
cards_y      = frame_height * 0.75
```

---

## Step-by-Step: Building a Full Presentation

### Step 1: Create the Parent Container Frame

```bash
PARENT_ID=$(miroctl board items create frame \
  --board-id $BOARD_ID \
  --title "My Presentation" \
  --width 8000 --height 3000 \
  --x 0 --y 0 \
  --format json | jq -r '.id')
```

Or via MCP (board-absolute coords):
```
Use mcp__miro tools to create a large outer frame at board center (0,0)
```

### Step 2: Create Slide Frames Inside the Parent

Slides stack horizontally inside the parent with spacing. Use these offsets from the parent's top-left:

```
slide_width  = 1920
slide_height = 1080
gap          = 100   ← between slides

slide[0].x_from_parent_topleft = 0
slide[1].x_from_parent_topleft = slide_width + gap
slide[2].x_from_parent_topleft = 2*(slide_width + gap)
...

# frame-relative center coords (miroctl):
slide[i].x = i * (slide_width + gap) + slide_width/2
slide[i].y = slide_height / 2
```

```bash
SLIDE_ID=$(miroctl board items create frame \
  --board-id $BOARD_ID \
  --title "Slide 1 - Title" \
  --format ratio_16x9 \
  --fill-color "#ffdd33" \
  --x 960 --y 540 \
  --parent-id $PARENT_ID \
  --format json | jq -r '.id')
```

### Step 3: Add Content to Each Slide

Use frame-relative coordinates (miroctl) or board-absolute (MCP). See the layout templates above for exact item positions.

**Text HTML format for Miro:**
```html
<!-- Bold italic title -->
<p><strong><em>Your Title Here</em></strong></p>

<!-- Bold label -->
<p><strong>Label</strong></p>

<!-- Body copy -->
<p>Regular body text here.</p>

<!-- Highlighted impact -->
<p><em style="background-color:rgb(255,221,51)">IMPACT: result</em></p>
```

### Step 4: Arrange Slides in Presentation Order

Miro's presentation mode follows child frame order (creation order). Create slides in sequence to control order, or reorder via the Miro UI.

---

## Quick Reference: Item Creation Commands

### Frame (slide)
```bash
miroctl board items create frame \
  --board-id $BOARD_ID \
  --title "Slide Title" \
  --format ratio_16x9 \
  --fill-color "#ffdd33" \
  --parent-id $PARENT_ID \
  --x $CX --y $CY
```

### Text
```bash
miroctl board items create text \
  --board-id $BOARD_ID \
  --content "<p><strong><em>Title</em></strong></p>" \
  --font-family roobert_pro \
  --font-size 86 \
  --color "#1c1c1e" \
  --x $X --y $Y \
  --width $W \
  --parent-id $SLIDE_ID
```

### Shape (card, background, header bar)
```bash
miroctl board items create shape \
  --board-id $BOARD_ID \
  --shape round_rectangle \
  --fill-color "#f8c8fc" \
  --border-color "#ce70fc" \
  --border-width 2 \
  --x $X --y $Y \
  --width $W --height $H \
  --parent-id $SLIDE_ID
```

### Sticky Note
```bash
miroctl board items create sticky_note \
  --board-id $BOARD_ID \
  --content "<p>Discussion question here</p>" \
  --fill-color "#fff9c4" \
  --text-align center \
  --x $X --y $Y \
  --parent-id $SLIDE_ID
```

---

## Coordinate Scaling

The board data shows two scales in use:

| Scale | Width | Height | When used |
|-------|-------|--------|-----------|
| Internal (Miro board scale) | ~476 | ~268 | Observed in API for this deck's nested frames |
| Full HD | 1920 | 1080 | Recommended for new slide creation |

**Recommendation:** Always create new slides at **1920 × 1080** using `format: ratio_16x9`. Position item coordinates proportionally within that space. This matches Miro's standard presentation format and will display correctly in presentation mode.

When placing items inside a 1920×1080 slide frame (miroctl, frame-relative):
```
Title text:     x: 550,  y: 220,  width: 2700
Subtitle/date:  x: 1700, y: 870,  width: 900
3-col card[0]:  x: 360,  y: 700,  width: 1600, height: 1200
3-col card[1]:  x: 960,  y: 700,  ...
3-col card[2]:  x: 1560, y: 700,  ...
Left half:      x: 480,  y: 600,  width: 880
Right half:     x: 1440, y: 600,  width: 880
Impact row:     x: 960,  y: 940,  width: 3700, (full-width, bottom)
```

---

## Common Mistakes to Avoid

- **Do not** use board-absolute coordinates when using miroctl with `--parent-id` — coordinates are frame-relative (origin = frame top-left)
- **Do not** add `position.relativeTo` when using miroctl — it causes a 400 error
- **Do not** place items outside the frame bounding box — they will visually escape
- **Always** create the parent frame before creating child slides
- **Always** create slide frames before creating their content items
- **Text in Miro must be HTML** — plain strings are not accepted; wrap in `<p>` tags
