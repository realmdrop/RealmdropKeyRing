# Patterns & Principles

## Design Principles

### 1. Developer-Native
This is a tool for developers, not a consumer product. Prioritize:
- Keyboard-first interactions
- Monospace for all values and technical data
- Dense but readable information density
- No hand-holding or excessive confirmation for non-destructive actions

### 2. Dark by Default
The dark theme is the primary design surface. Light mode is supported but dark should always be designed first and look best. Dark mode uses background layering (not shadows) for depth.

### 3. Quiet Until Needed
- Delete buttons appear on hover, not by default
- History sections are collapsed
- Copy format dropdowns open on demand
- Link remove icons fade in on hover
- Modals use subtle scale animation, not dramatic slides

### 4. Security is Visible
- Values are masked by default
- Auto-lock countdown is always visible when active
- Type badges immediately identify what kind of secret you're looking at
- Danger actions are visually distinct (red text/border)

### 5. Single-Surface Simplicity
The app is a single HTML file. Design decisions should respect this constraint:
- No multi-page navigation
- Sidebar + main content is the only layout paradigm
- Modals and overlays for focused tasks
- No routing, no loading states, no spinners

---

## Layout Patterns

### Three-Zone Layout

```
+--topbar (56px fixed)---------------------------+
|  logo  |  search  |  actions (lock, theme, ?)  |
+---------+--------------------------------------+
| sidebar | main content                          |
| (280px) | (flex: 1, scrollable)                 |
| fixed   |                                       |
|         |  project header                       |
|         |  link chips                           |
|         |  tag filter bar                       |
|         |  secret card grid                     |
|         |                                       |
+---------+---------------------------------------+
```

### Progressive Disclosure

Information is revealed in layers:
1. **Sidebar:** Project name + secret count (overview)
2. **Main area:** Secret cards with labels, types, masked values (browsing)
3. **Card interaction:** Reveal value, copy, see history (working)
4. **Modal:** Full editing, generation, import/export (focused task)

### Empty States

When no content exists, show:
- A muted SVG icon (low opacity: 0.3)
- A heading in `--text-dim` (16px)
- A subtitle in `--text-muted` (13px)
- Centered vertically and horizontally in the content area

---

## Interaction Patterns

### Hover Reveals
Used for secondary actions that would clutter the default view:
- Project delete button (sidebar)
- Link chip remove icon
- Link indicator icon on project items

Implementation: `opacity: 0` by default, `opacity: 1` (or `0.5/0.7`) on parent hover, with `transition: opacity 150ms ease`.

### Focus States
- All inputs: `border-color: --accent` on focus
- No `outline` (removed globally), border is the focus indicator
- Search input gets accent border
- Buttons don't have a visible focus ring (acceptable for this mouse/keyboard-hybrid context, but consider adding `focus-visible` outlines for accessibility)

### Copy to Clipboard
- Single click: copies raw value, shows green toast
- Chevron/long-press: opens copy format dropdown
- Toast appears at bottom-right, auto-dismisses
- Clipboard clears after 30 seconds (security)

### Modals
- Open: overlay fades in, modal scales up from 0.96
- Close: click overlay background, press Escape, or click Cancel
- Submit: primary button or Cmd/Ctrl+Enter
- Stacking: confirm dialogs can appear over modals (higher z-index)

---

## Iconography

### Style
- **Source:** Inline SVG (no icon library dependency)
- **Size:** 16px for inline/toolbar, 20px for topbar, 24px for empty states
- **Stroke:** 2px stroke width, `currentColor` for color inheritance
- **Fill:** None (outline style only)
- **Viewbox:** 24x24 standard

### Common Icons Used
- Lock (padlock) - brand icon, lock/unlock actions
- Search (magnifying glass) - search input
- Plus - add actions
- Trash - delete
- Eye / Eye-off - reveal/hide values
- Copy - clipboard
- Edit (pencil) - edit actions
- Clock - history
- Download - export
- Upload - import
- Link - external links
- Key - secrets/keyring metaphor
- Sun/Moon - theme toggle
- Keyboard - shortcuts overlay

### Rules
- Always use `currentColor` so icons inherit text color
- Use `flex-shrink: 0` on icons in flex containers
- Vertical alignment: `vertical-align: middle` or flex centering
- For service-specific icons (GitHub, AWS, etc.), use simplified single-color SVGs that match the 16px/2px-stroke aesthetic

---

## Responsive Considerations

The current design is desktop-first (single HTML file, local dev tool context). However:
- The secret grid uses `auto-fill, minmax(360px, 1fr)` which naturally collapses to single column
- The sidebar is fixed-width and doesn't collapse (acceptable for this use case)
- Modals are max `480-560px` wide with `max-height: 90vh`
- Login card is fixed `400px` (centered)

If mobile support is ever needed:
- Sidebar should become a slide-out drawer
- Topbar should stack vertically
- Secret cards should go full-width single column
- Modals should go full-screen on small viewports

---

## Accessibility Notes

### Current State
- Keyboard shortcuts exist (Escape, Enter, Cmd+K, etc.)
- Color contrast is generally good in dark mode (light text on dark backgrounds)
- Focus states exist via border color changes

### Improvements to Consider
- Add `focus-visible` outlines for keyboard navigation
- Add `aria-label` attributes to icon-only buttons
- Add `role="dialog"` and `aria-modal="true"` to modals
- Ensure all interactive elements are in tab order
- Add skip-to-content link
- Test with screen reader for vault unlock flow
