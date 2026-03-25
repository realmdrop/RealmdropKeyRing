# Foundations

## Brand Identity

**Product Name:** KeyRing by Realmdrop
**Tagline:** Local Secrets Manager
**Positioning:** Offline-first, zero-server, zero-cloud developer secrets manager. Security through simplicity.

**Brand Personality:**
- Technical but approachable
- Secure and trustworthy
- Minimal and focused
- Developer-native (not enterprise)

---

## Color System

### Dark Theme (Default)

| Token | Hex | Usage |
|---|---|---|
| `--bg` | `#0a0a0b` | Page background, deepest layer |
| `--bg-card` | `#131316` | Cards, sidebar items, modals |
| `--bg-card-hover` | `#1a1a1f` | Hover states on cards/list items |
| `--bg-input` | `#0e0e11` | Input fields, code blocks, inset surfaces |
| `--border` | `#1e1e24` | Default borders, dividers |
| `--border-focus` | `#3a3a44` | Focused/hovered borders |

#### Text

| Token | Hex | Usage |
|---|---|---|
| `--text` | `#e4e4e7` | Primary text (headings, body) |
| `--text-dim` | `#71717a` | Secondary text (labels, descriptions) |
| `--text-muted` | `#52525b` | Tertiary text (placeholders, timestamps) |

#### Accent & Semantic

| Token | Hex | Usage |
|---|---|---|
| `--accent` | `#a78bfa` | Primary accent (violet 400) - links, focus rings, active states |
| `--accent-dim` | `#7c3aed` | Accent backgrounds - primary buttons, fills |
| `--danger` | `#ef4444` | Destructive actions, errors |
| `--danger-dim` | `#991b1b` | Danger hover backgrounds (dark mode) |
| `--success` | `#22c55e` | Confirmations, copy feedback toast |
| `--warning` | `#f59e0b` | Auto-lock countdown, caution states |

#### Secret Type Colors

Each secret type has a background/foreground pair for badges:

| Type | BG | FG | Vibe |
|---|---|---|---|
| Password | `#7c3aed22` | `#a78bfa` | Violet - core identity color |
| API Key | `#2563eb22` | `#60a5fa` | Blue - trust, integration |
| SSH Key | `#05966922` | `#34d399` | Green - terminal, secure access |
| Env Var | `#d9770622` | `#fb923c` | Orange - config, infrastructure |
| Connection | `#db277722` | `#f472b6` | Pink - relational, linked |
| Other | `#52525b22` | `#a1a1aa` | Gray - neutral fallback |

### Light Theme

The light theme inverts the surface hierarchy while keeping semantic colors consistent. Key shifts:
- Backgrounds go from near-black to near-white (`#f4f4f5`)
- Cards become pure white (`#ffffff`)
- Accent shifts slightly deeper (`#7c3aed` instead of `#a78bfa`) for contrast on light surfaces
- Type badge backgrounds become opaque pastels instead of translucent overlays

---

## Typography

### Font Stack

| Role | Family | Weight Range | Fallback |
|---|---|---|---|
| **Sans (UI)** | Inter | 400-800 | system-ui, sans-serif |
| **Mono (Code)** | JetBrains Mono | 400-600 | Fira Code, monospace |

### Type Scale

| Element | Size | Weight | Font | Notes |
|---|---|---|---|---|
| App logo (topbar) | 16px | 800 | Inter | letter-spacing: -0.3px |
| Login logo | 28px | 800 | Inter | letter-spacing: -0.5px |
| Page heading (h1) | 24px | 700 | Inter | letter-spacing: -0.5px |
| Modal heading (h2) | 18px | 700 | Inter | |
| Confirm heading (h3) | 16px | 600 | Inter | |
| Sidebar section heading | 12px | 700 | Inter | UPPERCASE, letter-spacing: 1px |
| Secret card label | 15px | 600 | Inter | |
| Body text | 13px | 400 | Inter | Default paragraph size |
| Small text / meta | 11-12px | 400-600 | Inter | Labels, descriptions |
| Monospace values | 13px | 400 | JetBrains Mono | Secret values, code |
| Monospace small | 11-12px | 400 | JetBrains Mono | Badges, timestamps, counters |
| Type badges | 10px | 600 | JetBrains Mono | UPPERCASE, letter-spacing: 0.5px |

### Type Rules

- **Headings** use negative letter-spacing for a tight, modern feel
- **Uppercase labels** use positive letter-spacing (0.5-1px) for readability at small sizes
- **Monospace** is used for anything that represents a value, secret, count, or technical metadata
- **Sans** is used for all UI chrome, navigation, and human-readable text
- Line height: 1.5 for body text, tighter for headings and badges

---

## Spacing & Layout

### Spacing Scale

Use multiples of 4px as the base unit:

| Token | Value | Usage |
|---|---|---|
| 2px | 2px | Margin between stacked sidebar items |
| 4px | 4px | Inner gaps (icon groups, checkbox rows) |
| 6px | 6px | Input-to-label gap |
| 8px | 8px | Standard gap between related elements |
| 10px | 10px | Inset padding (inputs, cards inner elements) |
| 12px | 12px | Padding inside compact components |
| 14px | 14px | Card inner padding gaps |
| 16px | 16px | Standard section spacing, form group margin |
| 20px | 20px | Card padding |
| 24px | 24px | Topbar horizontal padding, modal actions top margin |
| 32px | 32px | Main content area padding, modal inner padding |
| 48px | 48px | Login box top/bottom padding |

### Layout Structure

- **Topbar:** 56px height, full-width, border-bottom divider
- **Sidebar:** 280px fixed width, left-aligned, border-right divider
- **Main content:** Fills remaining space, scrolls independently
- **Secret grid:** CSS Grid with `auto-fill, minmax(360px, 1fr)`, 16px gap

---

## Border Radius

| Token | Value | Usage |
|---|---|---|
| `--radius` | 8px | Buttons, inputs, cards (compact), badges, dropdowns |
| `--radius-lg` | 12px | Cards, modals, login box, confirm dialogs |
| 50% | Circle | Color swatches, project dots, avatars |
| 99px | Pill | Link chips, tags |
| 4px | Subtle | Type badges, dropdown items, scrollbar thumb |
| 2px | Minimal | Password strength bar |

---

## Shadows & Depth

The design uses minimal shadows, relying primarily on borders and background contrast for depth:

| Element | Shadow |
|---|---|
| Dropdowns | `0 8px 24px rgba(0,0,0,0.4)` |
| Copy feedback toast | None (uses color + position) |
| Modals | None (uses overlay + border) |
| Cards | None (uses border only) |

**Depth model:** Background color layering (dark < darker < darkest) rather than elevation shadows. This keeps the interface flat and reduces visual noise.

---

## Motion & Transitions

| Token | Value | Usage |
|---|---|---|
| `--transition` | `150ms ease` | Default for all micro-interactions (hover, focus, color changes) |
| Modal open | `200ms ease` | Overlay fade + modal scale (0.96 -> 1.0) |
| Toast | `200ms ease` | Slide up + fade in (translateY: 10px -> 0) |
| Theme switch | `300ms ease` | Background and color transitions |
| Strength bar | `300ms ease` | Width + color animation |

**Motion principles:**
- Keep transitions fast (150-200ms) - this is a productivity tool
- Use ease timing, never bounce or spring
- Scale transforms for modals (subtle 0.96 -> 1.0)
- Opacity transitions for progressive disclosure (delete buttons, link indicators)
