# KeyRing by Realmdrop

**Your keys. Your brands. One secure place.**

KeyRing is a free, offline-first password manager and brand kit organizer built as a single HTML file. No server, no cloud, no account required. Everything is encrypted locally in your browser using AES-256-GCM.

Built by [Realmdrop Software](https://realmdrop.com).

---

## Features

### Secrets Management
- **Per-project storage** for API keys, passwords, SSH keys, connection strings, environment variables
- **Secret generator** with Random, Hex, Base64, and UUID v4 presets
- **Copy as 7 formats** — raw value, .env, shell export, Docker flag, CLI flag, JSON, YAML
- **Type-specific templates** — connection string builder with live preview, SSH key validator, API key prefix detection
- **Version history** — tracks up to 20 previous values per secret with optional rotation notes
- **Rotation reminders** — set a due date, get visual indicators when secrets need rotating
- **.env export/import** per project with auto-detected types
- **Shareable encrypted files** (`.vault-share`) with optional expiry

### Brand Kit (per project)
- **Brand identity** — name, tagline, wordmark usage rules
- **Color palette** — with roles (Primary, Accent, Dark BG, Light BG, Surface, Custom) and usage notes
- **Typography** — full type scale with font, weight, size, letter spacing, line height, usage context
- **Logo sets** — grouped by concept (e.g. "Primary Mark", "Wordmark") with multiple file variants per set
- **Logo usage rules** — minimum size, clear space, do's, don'ts, background guidance
- **Voice & personality** — brand keywords, words to avoid, tone notes, positioning statement
- **Sub-brands** — child brand color tokens (accent, surface, text, border)
- **Export for Claude** — one-click markdown export for use in Claude Projects, Notion, or repo docs
- **Import template** — download a pre-structured JSON template that Claude or a designer can fill in

### Organization
- **Subprojects** — unlimited nesting with collapsible sidebar tree
- **Project links** — quick-access chips to GitHub, AWS, Vercel, Notion, Linear, Jira, Slack (auto-detected icons)
- **Tag filtering** — tag secrets and filter by tag within a project
- **Search** — full-text search across all secrets with hierarchy path labels

### Security
- **AES-256-GCM** encryption with PBKDF2 key derivation (100,000 iterations)
- **Decrypt-based authentication** — no password hash stored, verification via encrypted token
- **Auto-lock** with configurable timeout (5 min to 2 hours, or manual only)
- **Clipboard auto-clear** after configurable delay
- **Data loss protection** — warns before overwriting existing vault data, recovers from missing verify tokens
- **Encrypted exports** with vault password or separate export password

### Quality of Life
- **Dark / light theme** with system preference detection
- **Session persistence** (opt-in) — stay unlocked on page refresh, cleared on tab close
- **Keyboard shortcuts** — Cmd+K search, Cmd+N new secret, Cmd+L lock, Cmd+E .env export, and more
- **Progressive Web App** — installable from browser, works fully offline
- **Settings panel** (Cmd+,) — auto-lock timeout, clipboard timeout, session toggle

---

## Getting Started

### Option 1: Open directly
Double-click `vault.html` to open in your browser. Everything works — no server needed.

### Option 2: Serve locally (recommended for PWA)
```bash
cd PasswordManager
python3 -m http.server 8080
```
Then open `http://localhost:8080/vault.html`. This enables the service worker for offline support and the "Install App" prompt.

### First run
1. Choose a strong master password (8+ characters)
2. Create your first project
3. Start adding secrets or set up your brand kit

---

## Important Notes

### Your data lives in your browser
KeyRing stores everything in your browser's `localStorage`, encrypted with your master password. This means:

- **Each browser profile is a separate vault.** Chrome, Firefox, Safari, and different browser profiles each have their own isolated storage. A vault created in one is invisible to another.
- **Clearing browser data will delete your vault.** If you clear localStorage or site data, your encrypted vault is gone.
- **Private/incognito mode may not persist.** Some browsers clear localStorage when private windows close.

### Backup your vault
Use **Export** (in the top bar) to create an encrypted backup file regularly. Store it somewhere safe — a cloud drive, USB stick, or another device. You can import it into any browser to restore your vault.

To move your vault to a different browser or profile:
1. Export (encrypted with your vault password)
2. Open KeyRing in the new browser/profile
3. Create a new vault (any password)
4. Import your backup file
5. Enter the original export password

### Master password
There is no password recovery. If you forget your master password, your data cannot be decrypted. KeyRing does not store your password anywhere — it derives an encryption key from it.

---

## File Structure

```
PasswordManager/
├── vault.html              ← the entire app (single file)
├── manifest.json           ← PWA manifest
├── sw.js                   ← service worker for offline support
├── icon-192.png            ← app icon (192x192)
├── icon-512.png            ← app icon (512x512)
├── keyring-brand.md        ← KeyRing's own brand reference
├── CLAUDE.md               ← developer brief for Claude Code
├── README.md               ← this file
├── Branding/
│   ├── keyring-logo.svg              ← logo mark (Deep Violet bg)
│   ├── keyring-logo-transparent.svg  ← logo mark (no bg)
│   ├── keyring-wordmark-light.svg    ← full lockup (light mode)
│   └── keyring-wordmark-dark.svg     ← full lockup (dark mode)
├── styleguide/
│   ├── 01-foundations.md   ← design tokens
│   ├── 02-components.md    ← component specs
│   ├── 03-patterns.md      ← design patterns
│   └── 04-prompt.md        ← condensed design system ref
└── archive/                ← completed spec files
```

---

## Tech Stack

- **Zero dependencies.** No npm, no build tools, no frameworks.
- **Single HTML file** with inline CSS and JavaScript (~6800 lines)
- **Web Crypto API** for all encryption (AES-GCM, PBKDF2)
- **Inter** (UI) + **JetBrains Mono** (code) from Google Fonts
- **PWA** with service worker (cache-first, offline capable)

---

## Brand

KeyRing is part of the Realmdrop Software family. It inherits the Realmdrop color system:

| Color | Hex | Role |
|-------|-----|------|
| Deep Violet | `#2E1065` | Primary — CTAs, buttons, logo background |
| Mint | `#00FFB2` | Accent — active states, success, "Ring" in dark mode |
| Void | `#12062E` | Dark mode background |
| Violet Mist | `#F9F7FF` | Light mode background |

Logo: a ring with two keys (south + 45° southeast) and a mint center dot.

---

## License

Free to use. Built by Realmdrop Software.

---

## Contributing

KeyRing is developed using [Claude Code](https://claude.ai/claude-code). To make changes:

1. Read `CLAUDE.md` for the full architecture brief
2. Read `styleguide/04-prompt.md` for the design system
3. Read `keyring-brand.md` for brand guidelines
4. All changes go in `vault.html` — keep it single-file
5. Test in both dark and light themes
6. Test the unlock → lock → unlock cycle after any auth changes
