# KeyRing by Realmdrop — Claude Code Brief

## What This Is

KeyRing is a **local-first, offline-first developer tool** by Realmdrop Software. Single-file web app (`vault.html` + PWA assets), everything encrypted in `localStorage` using AES-GCM + PBKDF2. No server. No cloud. No dependencies.

Two purposes:
1. **Secret/credential storage** — API keys, passwords, connection strings, .env values, SSH keys, organized by project with history tracking and rotation reminders
2. **Brand kit storage** — colors with roles, typography, logo sets with usage rules, voice guidelines, and sub-brand tokens per project — exportable as markdown for Claude

---

## Brand Guide (ALWAYS FOLLOW)

Reference file: `keyring-brand.md` — read it before making any visual changes.

**Identity:**
- Product name: **KeyRing** (CamelCase — capital K, capital R)
- Always show "BY REALMDROP" below in uppercase label style (9px, letter-spacing 0.1em, 40% opacity)
- Tagline: "Your keys. Your brands. One secure place."
- Logo: ring with two keys (south + southeast 45°) + mint center dot. See `Branding/` folder for all variants.

**Colors (Realmdrop system):**

| Role | Name | Hex | Usage |
|------|------|-----|-------|
| Primary | Deep Violet | `#2E1065` | CTAs, buttons, borders, logo bg |
| Accent | Mint | `#00FFB2` | Active states, success, "Ring" in dark mode |
| Dark BG | Void | `#12062E` | Dark mode page background |
| Surface | Surface Dark | `#1E0D45` | Dark mode cards/panels |
| Light BG | Violet Mist | `#F9F7FF` | Light mode page background + surfaces |
| Highlight | Violet Tint | `#EDE8FF` | Light mode hover/active states, badges |

**Typography:**
- UI text: `Inter` (400/500/600/700/800) — variable `--font-sans`
- Code/values: `JetBrains Mono` — variable `--font-mono`
- See `keyring-brand.md` for full type scale

**Voice:** Trustworthy, Clean, Modern, Calm, Capable, Organized
**Avoid:** Alarming, Fear-based, Technical jargon, Paranoid, Corporate

---

## Architecture

### Single-file (`vault.html`)
No build system, no npm, no external dependencies. All JS/CSS/HTML inline. Servable via `python -m http.server` or opening directly.

### Data Model (version 2)

```json
{
  "version": 2,
  "projects": [{
    "id": "uuid",
    "name": "Project Name",
    "parentId": null,
    "color": "#hex",
    "avatar": null,
    "links": [{ "id", "label", "url", "icon" }],
    "branding": {
      "name": "", "tagline": "", "wordmarkNotes": "",
      "colors": [{ "id", "name", "hex", "role", "usage" }],
      "typography": [{ "id", "role", "family", "weight", "size", "letterSpacing", "lineHeight", "usage" }],
      "logoSets": [{
        "id", "name", "description",
        "files": [{ "id", "variant", "label", "dataUrl", "filePath" }]
      }],
      "logoUsageRules": { "minimumSize", "clearSpace", "do": [], "doNot": [], "backgroundGuidance", "notes" },
      "voice": { "keywords": [], "avoid": [], "toneNotes": "", "positioning": "" },
      "subBrands": [{ "id", "name", "accent", "surface", "text", "border", "notes" }],
      "logos": [],
      "notes": ""
    },
    "secrets": [{
      "id": "uuid",
      "label": "", "type": "api_key", "value": "",
      "fields": {}, "notes": "", "tags": [],
      "history": [{ "value", "changedAt", "note" }],
      "rotateDue": null,
      "createdAt": "ISO8601", "updatedAt": "ISO8601"
    }]
  }]
}
```

### Crypto
- AES-GCM 256-bit encryption, PBKDF2 (100k iterations, SHA-256) key derivation
- All randomness: `crypto.getRandomValues()` — never `Math.random()`
- Auth: decrypt-based verification (encrypts known token, no SHA-256 hash)
- CryptoKey is extractable (`true`) for session persistence feature

### Subprojects
Projects have a `parentId` field (`null` = top-level). Tree structure computed at render time via `buildTree()`. Sidebar renders recursively with indentation, chevrons, `↳` indicators. Cycle prevention in parent picker.

---

## Implemented Features

All features below are **complete and functional** in vault.html:

### Security
- Decrypt-based password verification (no SHA-256 hash)
- Encrypted export with vault password or custom password
- Schema validation on import with pre-merge preview
- Auto-lock with configurable timeout (Settings panel)
- Clipboard auto-clear with configurable timeout
- Session persistence via sessionStorage (opt-in)
- Brute-force protection implied by PBKDF2 cost

### Secrets
- Per-project secret storage with types: password, API key, SSH key, env var, connection string, other
- Secret templates per type (connection string builder, SSH key validator, API key prefix detection)
- Password/secret generator (Random, Hex, Base64, UUID)
- Copy as 7 formats (raw, .env, shell export, Docker, CLI flag, JSON, YAML)
- Secret history/versioning (up to 20 entries, change notes)
- Rotation reminders (rotateDue date, overdue/due-soon pills, filter pills)
- .env export and import per project with type auto-detection
- Shareable encrypted vault files (.vault-share) with expiry

### Brand Kit (per project)
- Brand identity: name, tagline, wordmark rules
- Color palette with roles (primary/accent/dark-bg/light-bg/surface/custom) and usage notes
- Typography entries with full type scale (role, family, weight, size, letter-spacing, line-height, usage)
- Logo sets: grouped by concept (e.g. "Primary mark", "Wordmark"), multiple files per set with variant tags
- Logo usage rules: minimum size, clear space, do's, don'ts, background guidance
- Voice & personality: keywords, avoid words, tone notes, positioning
- Sub-brands: name + 4 color swatches (accent, surface, text, border) + notes
- Auto-replaced by live subproject summary when project has children
- Import/export as JSON (version 3)
- Export for Claude as markdown (.md file)

### Navigation & UI
- Dark/light theme toggle with system preference detection
- Subproject hierarchy with collapsible sidebar tree
- Secrets/Brand Kit tabs per project in sidebar
- Keyboard shortcuts (Cmd+K search, Cmd+N add, Cmd+L lock, Cmd+E .env export, Cmd+[ parent nav, etc.)
- Shortcuts overlay (? key)
- Settings panel (Cmd+,) with auto-lock, clipboard, session options
- Project links with auto-detected service icons (GitHub, GitLab, AWS, Vercel, etc.)
- Search with full hierarchy path labels
- PWA (manifest.json, sw.js, installable)

---

## File Structure

```
PasswordManager/
├── vault.html              ← main app (all code here)
├── manifest.json           ← PWA manifest
├── sw.js                   ← service worker
├── icon-192.png            ← PWA icon
├── icon-512.png            ← PWA icon
├── keyring-brand.md        ← portable brand reference
├── CLAUDE.md               ← this file
├── Branding/
│   ├── keyring-logo.svg              ← mark on Deep Violet
│   ├── keyring-logo-transparent.svg  ← mark only, no bg
│   ├── keyring-wordmark-light.svg    ← full lockup, light mode
│   └── keyring-wordmark-dark.svg     ← full lockup, dark mode
├── styleguide/
│   ├── 01-foundations.md   ← design tokens, typography, spacing
│   ├── 02-components.md    ← buttons, inputs, cards, modals
│   ├── 03-patterns.md      ← layout, interaction, principles
│   └── 04-prompt.md        ← condensed design system for Claude
└── archive/                ← completed spec files (reference only)
    ├── vault-critique.md
    ├── vault-feature-prompts.md
    ├── FIXES.md
    ├── BRAND-KIT-IMPROVEMENTS.md
    ├── LOGO-SCHEMA.md
    ├── SETTINGS-AND-EXPIRY.md
    └── SUBPROJECTS.md
```

---

## Design System (Quick Reference)

Full details in `styleguide/04-prompt.md`. Key rules:

1. **No external dependencies.** Everything inline. No npm, no CDN.
2. **Inter for UI, JetBrains Mono for code.** Loaded from Google Fonts.
3. **Background layering, not shadows.** Only shadow: dropdown `0 8px 24px`.
4. **Both themes.** Every new CSS must use variables, never hardcode colors.
5. **4px base unit.** All spacing: 8, 12, 16, 20, 24, 32px.
6. **Inline SVG only.** 16-20px, 2px stroke, `currentColor`, viewBox 24x24.
7. **Fast transitions.** 150ms ease default, 200ms for modals, never >300ms.
8. **Monospace for data.** Values, counts, timestamps, hex codes — always `--font-mono`.
9. **Hide on hover.** Secondary actions (delete, remove) use opacity transitions.
10. **No prompt() dialogs.** Use proper modals matching existing pattern.

---

## Rules

1. **Stay single-file** — all JS/CSS/HTML in vault.html
2. **All randomness via `crypto.getRandomValues()`** — never `Math.random()`
3. **Use `textContent` not `innerHTML`** for user-generated content
4. **Preserve the IIFE scope** — all JS inside the existing wrapper
5. **Version the data model** — current: version 2. Migrations run silently on load.
6. **Branding is encrypted** — stored inside the vault blob alongside secrets
7. **Brand Kit export version** — currently 3 (logoSets + logoUsageRules)
8. **No new CSS variables** — use existing tokens from the design system
9. **Follow the brand guide** — Deep Violet + Mint, Inter font, "BY REALMDROP" attribution
10. **Archive, don't delete** — completed specs go to `archive/`, not trash
