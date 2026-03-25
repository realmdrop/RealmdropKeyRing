# Components

## Buttons

### Variants

| Variant | Class | Background | Border | Text | Use Case |
|---|---|---|---|---|---|
| Default | `.btn` | `--bg-card` | `--border` | `--text` | Secondary actions (Cancel, Export) |
| Primary | `.btn-primary` | `--accent-dim` | `--accent-dim` | `#fff` | Primary actions (Save, Unlock, Create) |
| Danger | `.btn-danger` | transparent | `--border` | `--danger` | Destructive actions (Delete, Clear) |
| Icon | `.btn-icon` | transparent | none | `--text-dim` | Toolbars, inline actions (copy, edit, reveal) |

### Sizes

| Size | Class | Padding | Font Size |
|---|---|---|---|
| Default | `.btn` | 10px 20px | 13px |
| Small | `.btn-sm` | 6px 12px | 12px |
| Full Width | `.btn-full` | 10px 20px | 13px (centered) |
| Icon | `.btn-icon` | 6px | Inherits icon size |

### States

- **Hover (Default):** bg shifts to `--bg-card-hover`, border to `--border-focus`
- **Hover (Primary):** bg shifts to `--accent` (lighter)
- **Hover (Danger/dark):** bg shifts to `--danger-dim`, text to white
- **Hover (Icon):** text brightens to `--text`, bg to `--bg-card-hover`
- **All transitions:** `150ms ease`

### Rules
- Always use `font-weight: 600`
- Always use `white-space: nowrap`
- Use `inline-flex` with `align-items: center` and `gap: 6px` for icon+text buttons
- Buttons in action rows are right-aligned with `8px` gap

---

## Inputs

### Text Inputs

- Background: `--bg-input`
- Border: `1px solid --border`
- Focus border: `--accent`
- Font: `--font-sans` at 13px for general inputs
- Font: `--font-mono` at 14px for password/secret value inputs
- Padding: `10-12px` horizontal, `10-12px` vertical
- Border radius: `--radius` (8px)
- No visible outline (`outline: none`), rely on border-color change

### Search Input

- Left-aligned SVG icon with `pointer-events: none`
- Left padding: `36px` to accommodate icon
- Placeholder color: `--text-muted`
- Width: `320px` in topbar

### Select / Dropdown

- Same styling as text input
- Custom chevron via encoded SVG `background-image` (right-aligned)
- `appearance: none` for cross-browser consistency
- `padding-right: 32px` for chevron space
- Options inherit `--bg-card` background

### Textarea

- Font: `--font-mono`
- `resize: vertical`
- `min-height: 80px`
- Otherwise same as text input

### Form Layout

- `.form-group`: `margin-bottom: 16px`
- Labels: 12px, weight 600, UPPERCASE, `--text-dim`, `letter-spacing: 0.5px`, `margin-bottom: 6px`
- `.form-row`: horizontal layout with `gap: 12px`, children flex equally

---

## Cards

### Secret Card

- Background: `--bg-card`
- Border: `1px solid --border`
- Border radius: `--radius-lg` (12px)
- Padding: `20px`
- Hover: border shifts to `--border-focus`
- Grid layout: `auto-fill, minmax(360px, 1fr)`, `gap: 16px`

**Card anatomy (top to bottom):**
1. **Header row:** Label (left, 15px/600) + action buttons (right, icon buttons)
2. **Type badge:** Inline below label, colored per type
3. **Value block:** Monospace, in `--bg-input` inset box, masked by default
4. **Notes:** 12px, `--text-dim`, 1.5 line-height
5. **Meta row:** 11px monospace, `--text-muted`, flex row with gaps
6. **History toggle:** (conditional) 11px monospace, `--text-muted`, clock icon

### Login Card

- Width: `400px`
- Padding: `48px 40px`
- Background/border same as card
- Centered on viewport (flexbox)
- Light mode: add a subtle box-shadow `0 1px 3px rgba(0,0,0,0.08)` for separation from `#f4f4f5` background

**Login card composition (top to bottom):**

1. **Lock icon (standalone, centered):**
   - Pull the lock SVG OUT of the text line
   - Size: 40px, centered above the brand name
   - Wrap in a 64px circle with `--accent` at ~10% opacity as background
   - Color: `--accent`
   - Creates a visual anchor and security signal

2. **Brand name (centered, two-tier):**
   - Line 1: "KeyRing" — 28px, weight 800, `--text`, `letter-spacing: -0.5px`
   - Line 2: "by Realmdrop" — 13px, weight 400, `--text-dim`
   - `text-align: center`
   - Gap between icon and brand: 16px
   - Do NOT put both on one line — "KeyRing by Realmdrop" wraps badly at card width

3. **Subtitle:**
   - "Local secrets manager. No server, no cloud."
   - 13px, `--text-dim`, centered
   - `margin-bottom: 32px` to breathe before the form

4. **Form fields** (left-aligned from here down)

5. **Error message:**
   - Remove `min-height: 20px` when empty — it creates dead space
   - Only add vertical margin when error text is present

6. **Submit button** (full width)

**Theme toggle:**
- Do NOT place inside the login card
- Position it absolute on the login SCREEN: `top: 24px; right: 24px`
- Keeps the card layout clean and uncluttered

**Optional polish:**
- Fade-in animation on page load (opacity 0→1, translateY 8px→0, 400ms ease)
- Add a show/hide toggle (eye icon) inside the password input

---

## Modals

### Structure

- **Overlay:** Fixed, full-screen, `rgba(0,0,0,0.6)`, `backdrop-filter: blur(4px)`
- **Modal body:** `--bg-card`, border, `--radius-lg`, `480px` wide (or `560px` for `.modal-wide`)
- **Max height:** `90vh` with `overflow-y: auto`
- **Padding:** `32px`
- **Heading:** h2 at 18px/700, `margin-bottom: 24px`
- **Actions row:** Right-aligned flex, `gap: 8px`, `margin-top: 24px`

### Animation

- Overlay: `opacity 0 -> 1` over `200ms`
- Modal: `scale(0.96) -> scale(1)` over `200ms`
- Controlled via `.open` class toggle

### Confirm Dialog

- Higher z-index than modals (950 vs 900)
- Narrower: `420px`
- Less padding: `28px`
- Heading: h3 at 16px
- Body: 13px, `--text-dim`, `line-height: 1.5`

---

## Badges

### Secret Type Badge

- `display: inline-block`
- Padding: `3px 8px`
- Border radius: `4px`
- Font: `--font-mono`, 10px, weight 600
- UPPERCASE, `letter-spacing: 0.5px`
- Colors: per-type bg/fg pairs (see Foundations)

### Key Prefix Badge (API Key)

- Same style as type badge but uses API key colors specifically
- Appears contextually when a key prefix is detected (sk_, pk_, ghp_, AKIA)

---

## Tags

- Background: `--tag-bg` (translucent accent)
- Border: `1px solid --tag-border`
- Border radius: pill (99px)
- Padding: `4px 10px`
- Font: 12px, weight 500
- Active state: accent background fill

---

## Link Chips

- Pill-shaped (`border-radius: 99px`)
- Padding: `4px 12px`
- Background: `--bg-input`
- Border: `1px solid --border`
- Font: 12px, weight 500, `--text-dim`
- Hover: border + text shift to `--accent`
- Remove "x" fades in on hover (opacity 0 -> 0.7)
- "Add" button: 28px circle, dashed border, "+" icon

---

## Sidebar

### Project Item

- Padding: `10px 12px`
- Border radius: `--radius`
- Hover: `--bg-card-hover`
- Active: `--bg-card`
- Contains: color dot (10px circle) or avatar (22px circle), name (14px/500, truncated), secret count (11px mono)
- Delete button: absolute-positioned right, fades in on hover

### Sidebar Header

- Padding: `16px`
- Heading: 12px, weight 700, UPPERCASE, `letter-spacing: 1px`, `--text-dim`
- Border-bottom divider

---

## Toast / Feedback

- Fixed position: `bottom: 24px, right: 24px`
- Background: `--success` green
- Text: black, 13px, weight 600
- Border radius: `--radius`
- Appears via opacity + translateY animation
- Auto-dismisses (controlled via JS)

---

## Scrollbar

- Width: `6px`
- Track: transparent
- Thumb: `--border` color, `3px` radius
- Thumb hover: `--border-focus`
