# Brand Kit — Improvements & Missing Features

This document describes what already exists in the Brand Kit tab, what is missing,
and exactly what needs to be built. Claude Code should read this alongside CLAUDE.md
and treat it as an addendum to Task 3 (Branding Storage Per Project).

---

## Current State (What Already Exists)

The Brand Kit tab is implemented and functional. Per-project it stores:

- `name` — brand name (text input)
- `tagline` — tagline (text input)
- `colors[]` — array of `{ id, name, hex }` with a color picker and hex copy
- `typography[]` — array of `{ id, role, family, weight, size }` with a live "Aa" preview
- `logos[]` — array of `{ id, name, dataUrl }` — base64 image uploads (max 512KB)
- `notes` — free-form textarea

All data is encrypted inside the vault and auto-saves on blur.

---

## What Is Missing

### 1. Color `role` and `usage` fields (HIGH)

**Problem:** Colors only store `name` and `hex`. There is no way to designate a color
as Primary, Accent, Dark Background, Light Background, or Custom. There is no usage
description explaining what the color is for.

**Why it matters:** Without role context, the Claude-readable export is ambiguous —
Claude cannot tell which color to use for CTAs vs. hover states vs. backgrounds.

**Fix:** Add two fields to each color object:

```json
{
  "id": "...",
  "name": "Deep Violet",
  "hex": "#2E1065",
  "role": "primary",
  "usage": "CTAs, logo mark, key UI elements"
}
```

`role` should be a dropdown with these options:
- `primary` — Primary brand color
- `accent` — Accent / highlight color
- `dark-bg` — Dark mode background
- `light-bg` — Light mode background / near-white
- `surface-dark` — Dark mode card/surface
- `custom` — Custom / other

Display the role as a small badge on each color card (e.g. a pill label "Primary").
The `usage` field is a short text input below the color name on the card.

---

### 2. Voice & Personality section (HIGH)

**Problem:** Completely absent from the data model and UI. No fields exist for brand
keywords, words to avoid, or tone guidelines.

**Why it matters:** This is the most important section for Claude. Without voice
guidelines, Claude writes in a generic tone rather than the brand's voice.

**Fix:** Add a new `voice` object to the branding data model:

```json
{
  "voice": {
    "keywords": ["Clean", "Frictionless", "Energetic"],
    "avoid": ["Jargon", "Passive voice", "Bureaucratic language"],
    "toneNotes": "Speaks like a confident product team. Direct, friendly, never corporate.",
    "positioning": "Customer-driven software. You request it, we build it."
  }
}
```

**UI — new "Voice & Personality" section in the Brand Kit tab:**

- **Brand keywords** — tag/chip input. User types a word and presses Enter or comma
  to add it as a pill chip with an × remove button. Stored as `voice.keywords[]`.
- **Words/phrases to avoid** — same tag/chip input pattern. Stored as `voice.avoid[]`.
- **Tone notes** — textarea, 3 rows. Stored as `voice.toneNotes`.
- **Positioning statement** — single-line text input. Stored as `voice.positioning`.

Style chips as small pills using `--tag-bg` and `--tag-border` CSS variables (already
defined in the app). Each chip: `font-size: 11px`, `padding: 2px 8px`,
`border-radius: 100px`, with an × button that removes it.

---

### 3. Sub-brands section (HIGH)

**Problem:** The data model has no sub-brand structure at all. For a parent brand like
Realmdrop Software, each product (CaretCart, AuthRealm, GateKeep, Whitcomb) needs
its own color tokens stored as a child of the parent project.

**Fix:** Add a `subBrands[]` array to the branding data model:

```json
{
  "subBrands": [
    {
      "id": "...",
      "name": "CaretCart",
      "accent": "#FF9B21",
      "surface": "#FFF4E8",
      "text": "#7A4200",
      "border": "#FFD9A3",
      "notes": "E-commerce suite"
    }
  ]
}
```

**UI — new "Sub-brands" section in the Brand Kit tab:**

Show sub-brands as a list of rows. Each row shows:
- Sub-brand name (text input)
- Four color swatches inline: Accent, Surface, Text, Border — each with a color picker
  and hex value displayed below
- A notes field (single line)
- A remove button (×)

Add an "+ Add sub-brand" button at the bottom of the section that appends a new empty
row. Save on blur/change same as other brand fields.

---

### 4. Typography — missing fields (MEDIUM)

**Problem:** Typography entries only store `role`, `family`, `weight`, `size`. Missing:
`letterSpacing`, `lineHeight`, and `usage` context.

**Fix:** Expand the typography object:

```json
{
  "id": "...",
  "role": "Display",
  "family": "Syne",
  "weight": "500",
  "size": "28px",
  "letterSpacing": "-0.03em",
  "lineHeight": "1.1",
  "usage": "Hero headlines and section titles"
}
```

**UX fix — replace `prompt()` dialogs with a proper inline modal:**

The current "Add" button for typography fires four sequential `prompt()` browser
dialogs. This is inconsistent with the rest of the app's polished modal system and
feels broken.

Replace with a proper modal (matching the existing modal pattern in vault.html) that
has labelled inputs for all six fields. The modal should have:
- Role (text input, e.g. "Headings", "Body", "Code")
- Font family (text input)
- Font weight (text input, e.g. "400", "500", "700")
- Font size (text input, e.g. "16px", "1.5rem")
- Letter spacing (text input, e.g. "-0.02em", "0.1em") — optional
- Line height (text input, e.g. "1.5", "1.7") — optional
- Usage notes (text input, e.g. "Used for all display headlines") — optional

On the typography card, display the additional fields as secondary detail text if present.

---

### 5. Logo variant tagging (MEDIUM)

**Problem:** Logos are stored as `{ id, name, dataUrl }` with no context about which
variant it is (default full logo, dark mode version, light mode version, icon only, etc.)

**Fix:** Add a `variant` field to each logo:

```json
{
  "id": "...",
  "name": "Realmdrop Logo",
  "variant": "default",
  "dataUrl": "data:image/png;base64,..."
}
```

`variant` is a dropdown with: `default`, `dark`, `light`, `icon`, `wordmark`, `other`.

Display the variant as a small badge on each logo card in the UI. This lets the
Claude-readable export specify which logo file is for which context.

---

### 6. Wordmark / identity notes field (LOW)

**Problem:** There is no field for wordmark-specific guidelines — how the wordmark
is spelled, casing rules, color split rules, minimum sizes, clear space, etc.

**Fix:** Add a `wordmarkNotes` text field to the Brand Identity section:

```json
{
  "name": "Realmdrop Software",
  "tagline": "Software that works the way you do",
  "wordmarkNotes": "Lowercase 'd' in Realmdrop. Single weight. 'drop' uses primary color split. Minimum size 120px wide.",
  ...
}
```

Add as a textarea below the tagline field in the Brand Identity section. Label:
"Wordmark & Logo Usage Rules". Placeholder: "Casing, color split rules, minimum
sizes, clear space guidelines..."

---

### 7. "Export for Claude" — markdown export (HIGH)

**Problem:** There is no way to export the Brand Kit as a readable file. This is the
single most important missing feature — without it, Claude cannot reference the brand
in other conversations, in Claude Code sessions, or in Notion.

**Fix:** Add an "Export for Claude" button in the Brand Kit tab header (top right,
next to any existing controls). On click, generate and download a `.md` file with
the following format:

**Filename:** `{project-name-kebab-case}-brand.md`
**Example:** `realmdrop-software-brand.md`

**File format:**

```markdown
# {Brand Name} — Brand Reference
Generated: {ISO date}
Project: {project name} | Source: KeyRing by Realmdrop

---

## Identity
- **Brand name:** {value}
- **Tagline:** {value}
- **Wordmark rules:** {value}

## Colors
| Role | Name | Hex | Usage |
|------|------|-----|-------|
| Primary | Deep Violet | #2E1065 | CTAs, logo mark, key UI elements |
| Accent | Mint | #00FFB2 | Highlights, active states, badges |
| Dark bg | Void | #12062E | Dark mode base background |
| Light bg | Violet Mist | #F9F7FF | Light mode base background |

## Typography
| Role | Family | Weight | Size | Letter Spacing | Line Height | Usage |
|------|--------|--------|------|----------------|-------------|-------|
| Display | Syne | 500 | 28px | -0.03em | 1.1 | Hero headlines |
| Body | Inter | 400 | 15px | 0 | 1.6 | Primary body copy |

## Voice & Personality
- **Keywords:** Clean, Frictionless, Energetic, Fast-moving
- **Avoid:** Jargon, passive voice, bureaucratic language
- **Tone:** {toneNotes}
- **Positioning:** {positioning}

## Sub-brands
| Name | Accent | Surface | Text | Border |
|------|--------|---------|------|--------|
| CaretCart | #FF9B21 | #FFF4E8 | #7A4200 | #FFD9A3 |
| AuthRealm | #7C3AED | #F0E8FF | #4A1E8A | #D0AAFF |
| GateKeep | #10B759 | #E8FFF4 | #0A5C32 | #A0EFC4 |
| Whitcomb | #7B7B99 | #F4F4F8 | #3C3C50 | #D0D0DC |

## Logos & Assets
| Name | Variant |
|------|---------|
| Realmdrop Logo | default |
| Realmdrop Logo Dark | dark |

## Notes
{brand notes free text}
```

Omit any section that has no data (e.g. if no sub-brands are set, omit that section
entirely). Use `—` for empty fields within a table row.

This file is the portable brand reference. It can be dropped into:
- A Claude Project (as project instructions or an attached file)
- A Notion page
- The repo's `/Branding` folder alongside `tokens.json`
- A `CLAUDE.md` file in any sub-project

---

### 8. Tokens JSON export/import (LOW)

**Problem:** The vault Brand Kit and the `tokens.json` in `/Realmdrop/Branding/` are
disconnected. There is no way to import from or export to the W3C design tokens format.

**Fix:** Add two buttons in the Brand Kit tab:

**"Import tokens.json"** — file input (hidden, triggered by button click) that accepts
a `.json` file in the format used by `/Realmdrop/Branding/tokens.json`. On import,
map the token values into the branding data model:
- `global.color.primary.*` → colors with role `primary`
- `global.color.dark.*` → colors with role `dark-bg` or `surface-dark`
- `global.color.light.*` → colors with role `light-bg`
- `global.sub-brand.*` → sub-brands array
- `global.typography.*` → typography entries

Show a preview before importing: "Found X colors, Y typography entries, Z sub-brands.
Import?" with Cancel / Import buttons.

**"Export tokens.json"** — generates a `tokens.json` file in the same W3C format as
`/Realmdrop/Branding/tokens.json`, derived from the current Brand Kit data. This keeps
the vault and the design system in sync.

---

## Updated Data Model

The complete branding object after all improvements:

```json
{
  "branding": {
    "name": "",
    "tagline": "",
    "wordmarkNotes": "",
    "colors": [
      {
        "id": "uuid",
        "name": "Deep Violet",
        "hex": "#2E1065",
        "role": "primary",
        "usage": "CTAs, logo mark, key UI elements"
      }
    ],
    "typography": [
      {
        "id": "uuid",
        "role": "Display",
        "family": "Syne",
        "weight": "500",
        "size": "28px",
        "letterSpacing": "-0.03em",
        "lineHeight": "1.1",
        "usage": "Hero headlines and section titles"
      }
    ],
    "logos": [
      {
        "id": "uuid",
        "name": "Realmdrop Logo",
        "variant": "default",
        "dataUrl": "data:image/png;base64,..."
      }
    ],
    "voice": {
      "keywords": ["Clean", "Frictionless", "Energetic"],
      "avoid": ["Jargon", "Passive voice"],
      "toneNotes": "",
      "positioning": ""
    },
    "subBrands": [
      {
        "id": "uuid",
        "name": "CaretCart",
        "accent": "#FF9B21",
        "surface": "#FFF4E8",
        "text": "#7A4200",
        "border": "#FFD9A3",
        "notes": "E-commerce suite"
      }
    ],
    "notes": ""
  }
}
```

---

## Implementation Order

Work through these in order. Each is self-contained.

1. **Color `role` + `usage` fields** — data model + UI (dropdown badge + usage input)
2. **Voice & Personality section** — data model + chip tag UI
3. **Sub-brands section** — data model + row-based UI with color pickers
4. **Typography modal** — replace `prompt()` dialogs + add missing fields
5. **Logo variant tagging** — add `variant` dropdown to logo cards
6. **Wordmark notes field** — add textarea to Brand Identity section
7. **Export for Claude** — markdown `.md` file download
8. **Tokens JSON import/export** — connect to `/Realmdrop/Branding/tokens.json` format

---

## Design Rules

All new UI must follow the existing design system exactly:

- Use existing CSS variables — no new colors
- Chip/tag inputs: `--tag-bg` fill, `--tag-border` border, `--accent` text
- Color role badges: small pill, `font-size: 10px`, `text-transform: uppercase`,
  `letter-spacing: 0.5px`, `padding: 2px 6px`, `border-radius: 4px`
  - primary → `--type-password-bg` / `--type-password-fg`
  - accent → `--type-api-bg` / `--type-api-fg`
  - dark-bg → `--type-other-bg` / `--type-other-fg`
  - light-bg → `--type-env-bg` / `--type-env-fg`
  - custom → `--type-conn-bg` / `--type-conn-fg`
- New modals: match existing `.modal-overlay` + `.modal-box` pattern exactly
- Buttons: use existing `.btn` and `.btn-sm` classes
- Save behavior: auto-save on blur/change (same as existing brand fields)
- Use `showToast()` for success feedback, `confirmDialog()` for destructive actions

---

## Migration

Existing branding objects in localStorage only have:
`{ name, tagline, colors, typography, logos, notes }`

On vault load (in the existing migration block), add defaults for missing fields:

```js
if (!p.branding.voice) {
  p.branding.voice = { keywords: [], avoid: [], toneNotes: '', positioning: '' };
}
if (!p.branding.subBrands) p.branding.subBrands = [];
if (!p.branding.wordmarkNotes) p.branding.wordmarkNotes = '';
p.branding.colors.forEach(c => {
  if (!c.role) c.role = 'custom';
  if (!c.usage) c.usage = '';
});
p.branding.typography.forEach(t => {
  if (!t.letterSpacing) t.letterSpacing = '';
  if (!t.lineHeight) t.lineHeight = '';
  if (!t.usage) t.usage = '';
});
p.branding.logos.forEach(l => {
  if (!l.variant) l.variant = 'default';
});
```

Run this migration silently — no user-facing prompt needed.
