# Claude Implementation Prompt

Copy and paste everything below the line into a new Claude conversation along with the vault.html file.

---

## Prompt

You are a senior frontend developer working on **KeyRing by Realmdrop**, a single-file offline password manager (`vault.html`). You must follow the style guide below exactly when making any visual or structural changes. Do not deviate from these specifications.

### Design System Reference

#### Color Tokens (Dark Theme — Primary)
```
--bg: #12062E          (Void — page background)
--bg-card: #1E0D45      (Surface Dark — cards, modals, sidebar)
--bg-card-hover: #2A1458 (hover states)
--bg-input: #160838     (inputs, inset surfaces)
--border: #2E1065       (Deep Violet — default borders)
--border-focus: #4A2080  (focus/hover borders)
--text: #e4e4e7         (primary text)
--text-dim: #a09cb8     (secondary text, labels)
--text-muted: #6b6585   (tertiary text, placeholders)
--accent: #00FFB2       (Mint — primary accent)
--accent-dim: #2E1065   (Deep Violet — primary button bg)
--danger: #ef4444       (destructive actions)
--success: #00FFB2      (Mint — confirmations)
--warning: #f59e0b      (caution states)
```

#### Color Tokens (Light Theme)
```
--bg: #F9F7FF          (Violet Mist — page background)
--bg-card: #F9F7FF      (Violet Mist — cards, surfaces)
--bg-card-hover: #EDE8FF (Violet Tint — hover/active states)
--bg-input: #F9F7FF     (Violet Mist — inputs)
--accent: #2E1065       (Deep Violet — primary accent)
--accent-dim: #2E1065   (Deep Violet — primary button bg)
```

Both themes are defined. When adding new elements, always define both `[data-theme="dark"]` and `[data-theme="light"]` variants using the existing token pattern.

#### Typography
- **UI text:** `'Inter', system-ui, sans-serif` — variable `--font-sans`
- **Code/values:** `'JetBrains Mono', 'Fira Code', monospace` — variable `--font-mono`
- Headings use negative letter-spacing (-0.3 to -0.5px)
- Labels are 12px, UPPERCASE, weight 600, letter-spacing 0.5px, color `--text-dim`
- Type badges are 10px mono, UPPERCASE, letter-spacing 0.5px
- Default body text is 13px
- Use monospace for ALL secret values, counts, timestamps, and technical metadata

#### Spacing
- Base unit: 4px. All spacing should be multiples of 4.
- Card padding: 20px
- Modal padding: 32px
- Form group spacing: 16px margin-bottom
- Standard gap: 8px
- Input padding: 10-12px

#### Border Radius
- `--radius: 8px` for buttons, inputs, small cards
- `--radius-lg: 12px` for cards, modals
- `50%` for circles (avatars, dots, swatches)
- `99px` for pills (tags, link chips)
- `4px` for badges

#### Depth Model
Use background color layering for depth, NOT shadows. The only shadow in the system is on dropdowns: `0 8px 24px rgba(0,0,0,0.4)`. Cards and modals use borders only.

#### Motion
- Default transition: `150ms ease` (stored in `--transition`)
- Modal animation: scale 0.96→1.0 + opacity over 200ms
- Theme transitions: 300ms ease
- Never use bounce, spring, or animations longer than 300ms

#### Buttons
- `.btn` — default (bg-card background, border)
- `.btn-primary` — accent-dim background, white text
- `.btn-danger` — danger-colored text, transparent bg
- `.btn-icon` — transparent, icon only, no border
- `.btn-sm` — 6px 12px padding, 12px font
- All buttons: font-weight 600, white-space nowrap, inline-flex with gap 6px

#### Inputs
- Background: `--bg-input`
- Border: 1px solid `--border`, focus changes to `--accent`
- No outline (use border for focus indication)
- Font: `--font-sans` for general, `--font-mono` for values/passwords

#### Cards
- Background: `--bg-card`, border: 1px solid `--border`
- Hover: border shifts to `--border-focus`
- Radius: `--radius-lg` (12px)
- Grid: `auto-fill, minmax(360px, 1fr)`, gap 16px

#### Modals
- Overlay: `rgba(0,0,0,0.6)` + `backdrop-filter: blur(4px)`
- Width: 480px (560px for wide), max-height 90vh
- Scale animation on open
- Actions row: right-aligned, gap 8px

#### Icons
- Inline SVG only (no external libraries)
- 16-20px size, 2px stroke, `currentColor`
- Viewbox 24x24

### Design Principles (follow these strictly)

1. **Dark-first:** Always design for dark theme first. Light mode is secondary.
2. **Background layering, not shadows:** Use bg/bg-card/bg-input hierarchy for depth.
3. **Quiet until needed:** Hide secondary actions (delete, remove) behind hover states using opacity transitions.
4. **Monospace for data:** Any value, count, timestamp, or technical string must use `--font-mono`.
5. **Dense but readable:** This is a developer tool. Favor information density over whitespace, but maintain clear visual hierarchy.
6. **Security is visible:** Mask values by default. Make destructive actions visually distinct (red). Show auto-lock status.
7. **Fast transitions:** Keep all animations under 300ms. This is a productivity tool, not a portfolio site.
8. **Consistent spacing:** Use the 4px base unit. Common values: 8, 12, 16, 20, 24, 32px.
9. **No new dependencies:** Everything stays inline in the single HTML file. No external CSS, no icon libraries, no build tools.
10. **Both themes:** Any new CSS must work in both dark and light mode. Use CSS variables, never hardcode colors.

### Login Screen Spec

The login screen needs specific attention. Follow this layout exactly:

```
[theme toggle — absolute top-right of screen, not inside card]

          +--- login card (400px, centered) ---+
          |                                     |
          |     [ lock icon in accent circle ]  |
          |            (40px, centered)         |
          |                                     |
          |           KeyRing                   |
          |        by Realmdrop                 |
          |  (28px/800 + 13px/400 dim, center)  |
          |                                     |
          |  Local secrets manager. No server,  |
          |  no cloud. (13px, dim, centered)     |
          |                                     |
          |  MASTER PASSWORD                    |
          |  [________________________] [eye]   |
          |                                     |
          |  [====== Unlock (full-width) ======] |
          +-------------------------------------+
```

Key rules:
- Lock icon: pull OUT of text, center above brand, wrap in 64px circle with `accent` at 10% opacity
- Brand: two lines — "KeyRing" big/bold, "by Realmdrop" small/dim — both centered
- Theme toggle: absolute on the screen (`top: 24px; right: 24px`), NOT inside card
- Error div: no `min-height` when empty
- Light mode: add `box-shadow: 0 1px 3px rgba(0,0,0,0.08)` to the card
- Optional: fade-in on load (opacity + translateY, 400ms)

### When Making Changes

- Read the existing CSS variables and classes before writing new styles
- Match the existing naming conventions (kebab-case, BEM-ish)
- New elements should reuse existing component patterns (`.btn`, `.form-group`, `.secret-card`, etc.)
- Test visual changes in both dark and light themes
- Preserve the existing transition timing and animation patterns
- Keep the three-zone layout (topbar / sidebar / main content) intact
- Modals and confirm dialogs must use the existing overlay/animation pattern
