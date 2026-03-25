# Vault — Feature Implementation Prompts

Each prompt below is a self-contained brief you can hand to an AI or use as a spec. They reference the existing `vault.html` architecture (single-file, localStorage, AES-GCM + PBKDF2, IIFE-scoped JS).

---

## 1. Password / Secret Generator

**Context:** Vault is a local-first developer secrets manager. The "Add Secret" modal currently has a plain textarea for the value field. Developers need to generate strong secrets directly in the tool.

**Task:** Add a "Generate" button next to the Value textarea in the secret modal. When clicked, it should open an inline generator panel with these controls:

- **Length slider** — range 8–128, default 32
- **Character set toggles** — Uppercase, Lowercase, Digits, Symbols (each a checkbox, all on by default)
- **Format presets dropdown** — "Random (default)", "Hex token", "Base64", "UUID v4", "Passphrase (4–8 words)"
  - When a preset is selected, disable irrelevant toggles (e.g., Hex disables everything, just generates hex)
  - Passphrase should use a bundled wordlist of ~2,000 common English words, joined with hyphens
- **"Regenerate" button** — rolls a new value with current settings
- **Live preview** of the generated value
- **"Use this" button** — populates the Value textarea and closes the panel

Use `crypto.getRandomValues()` for all randomness. Never use `Math.random()`.

The generator panel should appear inline below the textarea, not as a separate modal. Style it to match the existing dark theme using the CSS variables already defined (--bg-input, --border, --accent, etc.).

---

## 2. Encrypted Export (with Optional Password & Encrypted Import)

**Context:** The current `exportVault()` function dumps the entire vault as plaintext JSON. This is a security risk for a secrets manager. Import also exists but accepts unvalidated JSON.

**Task:** Replace the current export/import flow:

### Export
- When the user clicks "Export," show a modal asking:
  - **"Use vault password"** (default) — encrypts the export with the current AES-GCM key and salt
  - **"Set a separate export password"** — prompts for a new password, derives a fresh PBKDF2 key from it with a new random salt
  - **"Export unencrypted (not recommended)"** — plaintext JSON, but show a clear warning: "This file will contain all your secrets in plaintext."
- The encrypted export format should be a JSON file with structure: `{ version: 1, encrypted: true, salt: [...], iv: [...], ct: [...] }`
- The unencrypted format stays as-is but adds `{ version: 1, encrypted: false, projects: [...] }`
- File naming: `vault-export-YYYY-MM-DD.json`

### Import
- When importing, detect whether the file has `encrypted: true`
- If encrypted: prompt for the password, derive the key, decrypt, then merge
- If unencrypted: proceed with current merge logic
- **Add schema validation** before merging: every project must have `id` (string), `name` (string), `color` (string), `secrets` (array). Every secret must have `id`, `label`, `type`, `value` (all strings), `createdAt` (string). Reject entries that fail validation with a summary of what was skipped.
- Show a pre-merge preview: "This will import X projects with Y total secrets. Z secrets have matching labels and will be updated."

---

## 3. SHA-256 Authentication Fix

**Context:** The current vault setup stores `SHA-256(masterPassword)` in localStorage to verify the password on unlock. SHA-256 is not a password hashing function — it's fast and unsalted, making it trivially brute-forceable. Meanwhile, the app already uses PBKDF2 for key derivation, so the weak link is purely in the verification step.

**Task:** Replace the SHA-256-based password check with decrypt-based verification. Here's the approach:

### Setup (first time)
1. Generate a random 16-byte salt → store in `localStorage` as `vault_salt`
2. Derive AES-GCM key from password + salt via PBKDF2 (100k iterations, SHA-256)
3. Generate a **verification token**: a known fixed string like `"vault-verify-ok"`, encrypt it with the derived key → store as `vault_verify` (with its own IV)
4. Encrypt the empty vault → store as `vault_data`
5. **Remove** `vault_hash` from localStorage entirely — it should no longer exist

### Unlock (returning user)
1. Load salt from `vault_salt`
2. Derive key from entered password + salt
3. Attempt to decrypt `vault_verify` with the derived key
4. If decryption succeeds and the plaintext matches `"vault-verify-ok"` → password is correct, proceed to decrypt vault data
5. If decryption throws (AES-GCM authentication tag mismatch) → wrong password, show error

### Migration
- On app init, check if `vault_hash` exists in localStorage. If it does, the user is on the old scheme.
- After successful unlock (using the old SHA-256 check one final time), silently migrate: create the verification token, store it, then delete `vault_hash`.
- Log the migration to console: `"Migrated auth from SHA-256 to decrypt-based verification."`

**Remove entirely:** the `sha256()` function and the `lsSet('hash', hash)` / `lsGet('hash')` pattern.

---

## 4. .env File Export Per Project

**Context:** Developers frequently need secrets in `.env` file format for local development. Currently, the only way to get secrets out of Vault is copy-one-at-a-time or the full vault JSON export.

**Task:** Add two features to each project view:

### Export .env
- Add a "Export .env" button in the project header (next to "Add Secret")
- When clicked, generate a `.env` file from the project's secrets:
  - Use the secret's **label** as the variable name, converted to SCREAMING_SNAKE_CASE (replace spaces, hyphens, dots with underscores, uppercase everything)
  - Use the secret's **value** as the value, wrapped in double quotes if it contains spaces, `#`, or newline characters
  - Add the secret's **notes** as a comment above the line (prefixed with `#`)
  - Group by secret type, with a blank line and `# --- Type: API Keys ---` header between groups
- Download as `{project-name-kebab-case}.env`
- Example output:
  ```
  # Database password for production RDS
  DATABASE_PASSWORD="s3cret!value"

  # --- API Keys ---
  # Stripe live key — rotate quarterly
  STRIPE_API_KEY="sk_live_abc123"
  ```

### Import .env
- Add an "Import .env" button next to it
- Parse standard `.env` format: `KEY=value`, `KEY="value"`, `KEY='value'`, and `# comments`
- For each parsed entry, create a secret with:
  - `label`: the original KEY name (preserve original casing)
  - `type`: auto-detect — if the key contains `PASSWORD` or `SECRET` → "password", `KEY` or `TOKEN` → "api_key", `URL` or `URI` or `DSN` → "connection_string", else → "env_var"
  - `value`: the parsed value (with quotes stripped)
  - `notes`: any comment line immediately above the key
- Show a preview before importing: list all entries with detected types, let the user deselect any they don't want

---

## 5. Copy as Different Formats

**Context:** Developers need secrets in different formats depending on context — raw value for pasting into a UI, `export VAR=val` for a shell, `VAR=val` for .env files, `--flag=val` for CLI commands, etc.

**Task:** Replace the single "copy" button on each secret card with a copy dropdown:

- **Default click** (no dropdown) — copies raw value (current behavior)
- **Long-press or click the chevron/arrow** — opens a small floating menu with these options:
  - **Raw value** — `s3cretValue123`
  - **.env format** — `DATABASE_PASSWORD="s3cretValue123"` (label → SCREAMING_SNAKE_CASE)
  - **Shell export** — `export DATABASE_PASSWORD="s3cretValue123"`
  - **Docker env flag** — `-e DATABASE_PASSWORD="s3cretValue123"`
  - **CLI flag** — `--database-password=s3cretValue123` (label → kebab-case)
  - **JSON** — `"DATABASE_PASSWORD": "s3cretValue123"`
  - **YAML** — `DATABASE_PASSWORD: "s3cretValue123"`

Each option copies to clipboard immediately on click, shows the existing "Copied!" toast, and closes the menu.

Style the dropdown as a small floating card anchored to the copy button, using existing CSS variables. Add a subtle border and shadow. The menu should close on outside click or Escape.

The label-to-variable-name conversion should be a shared utility function (reused by the .env export feature).

---

## 6. Secret Templates Per Type

**Context:** When adding a "Connection String" secret, the user currently gets a blank textarea. But connection strings have a known structure (host, port, database, user, password) that could be assembled from individual fields — which is easier, less error-prone, and more useful.

**Task:** When the user selects certain secret types in the "Add Secret" modal, replace the single Value textarea with a structured form that auto-assembles the final value:

### Connection String
Fields: Protocol (dropdown: `postgresql`, `mysql`, `mongodb`, `redis`, `mssql`), Host, Port (auto-filled default per protocol), Database, Username, Password, Extra params (optional key=value textarea)
Assembled value: `protocol://username:password@host:port/database?params`
Show a live preview of the assembled string below the fields.

### SSH Key
Fields: Key type label (just informational), Value (large textarea, monospace, preserving newlines)
No assembly needed — but validate that it starts with `-----BEGIN` or `ssh-rsa`/`ssh-ed25519`.

### API Key
Fields: Value (single-line input), Provider (optional dropdown: AWS, Stripe, GitHub, GitLab, OpenAI, Anthropic, Custom), Key prefix indicator (auto-detect: show a badge if the key starts with `sk_`, `pk_`, `ghp_`, `AKIA`, etc.)

### Environment Variable
Fields: Variable name (auto-suggest SCREAMING_SNAKE_CASE from the label), Value

### Password
Fields: Value (single-line, masked by default with show/hide toggle), Username (optional companion field — many passwords are meaningless without knowing the associated username), URL/Service (optional)

For all template types, the assembled or entered value is what gets stored in `secret.value`. The individual field values should be stored in a new `secret.fields` object so that when editing, the structured form can be re-populated (rather than trying to parse the assembled string back into fields).

When the type dropdown changes, swap between the template form and a plain textarea (for "Other" type). If the user had typed something in the plain textarea and switches to a template type, warn before clearing.

---

## 7. Keyboard-Driven Workflow

**Context:** Vault is a developer tool. Developers live on keyboards. The current app has minimal keyboard support (Enter to submit forms, Escape to close modals).

**Task:** Add a comprehensive keyboard shortcut system:

### Global Shortcuts (when no modal is open)
- `Cmd/Ctrl + K` — Focus the search bar (like Spotlight/VS Code command palette)
- `Cmd/Ctrl + N` — Open "Add Secret" modal (if a project is selected)
- `Cmd/Ctrl + Shift + N` — Open "New Project" modal
- `Cmd/Ctrl + E` — Export .env for current project
- `Cmd/Ctrl + L` — Lock vault
- `Cmd/Ctrl + ,` — (future: open settings — for now, no-op)
- `Escape` — Clear search, or deselect project (cascade: search → project → nothing)

### Navigation
- `Up/Down arrow` (when search bar focused) — navigate through search results
- `1-9` — Quick-select project by position in sidebar (Cmd/Ctrl + 1 through 9)
- `Tab` through secret cards in the grid, `Enter` to toggle reveal on focused card, `C` to copy

### In Modals
- `Cmd/Ctrl + Enter` — Save/submit (already partially implemented, extend to all modals)
- `Escape` — Cancel/close
- `Tab` — Normal field navigation

### Visual Feedback
- Add a small "Keyboard shortcuts" button (or `?` icon) in the top bar that opens an overlay listing all shortcuts in a clean two-column layout (shortcut on left, description on right)
- Show keyboard hints on hover for buttons that have shortcuts (e.g., the lock button shows "Cmd+L" as a tooltip)

### Implementation Notes
- Use a single `keydown` listener on `document` that checks `e.metaKey || e.ctrlKey` for cross-platform support
- Prevent shortcuts from firing when the user is typing in an input/textarea (except Escape and Cmd-combos)
- All shortcuts should call `resetAutoLock()` since they indicate activity

---

## 8. Secret History / Versioning

**Context:** When developers rotate API keys or update passwords, they often need to reference the previous value (e.g., to revoke it, to update a deployment that's still using the old one). Currently, updating a secret overwrites the old value permanently.

**Task:** Add version history to secrets:

### Data Model Change
Add a `history` array to each secret object:
```json
{
  "id": "...",
  "label": "Stripe API Key",
  "value": "sk_live_new456",
  "history": [
    { "value": "sk_live_old123", "changedAt": "2025-01-15T...", "note": "" }
  ],
  ...
}
```

When `saveSecret()` detects that `value` has changed on an existing secret, push the old value onto the `history` array (capped at 20 entries, FIFO).

### UI
- On the secret card, add a small "History" icon/button (clock icon) that only appears if `history.length > 0`
- Clicking it expands a collapsible section below the current value showing past values:
  - Each entry shows: the masked value (with show/hide toggle), the date it was replaced, and a copy button
  - Most recent change first
  - Optional: a small "note" field shown inline if present (for things like "rotated due to breach")
- Add a "Clear history" link at the bottom of the history section with a confirmation dialog

### On Rotation
When editing a secret and changing the value, show a subtle prompt below the value field: "Add a note about this change? (optional)" — a small text input that gets stored as the `note` on the history entry.

### History in Export
Encrypted exports should include history. `.env` exports should NOT (only current values).

---

## 9. Shareable Encrypted Vaults

**Context:** Developers frequently need to share credentials with teammates — staging environment variables for a new hire, API keys for a contractor, production database credentials during an incident. The current options are terrible: Slack DMs, email, sticky notes. Vault should solve this without requiring a server.

**Task:** Add a "Share Project" feature that creates an encrypted, self-contained file:

### Creating a Share
- Add a "Share" button in the project header
- When clicked, show a modal:
  - **Select secrets to include** — checklist of all secrets in the project (all selected by default)
  - **Set a share password** — required field (separate from vault password, since the recipient won't know your vault password)
  - **Password strength indicator** below the field
  - **Expiry metadata** — optional "expires after" dropdown: 1 hour, 24 hours, 7 days, 30 days, never. This is advisory only (stored as metadata in the file, enforced on import — since it's a local file, it's trust-based)
  - **"Create Share"** button
- Generates a `.vault-share` file (actually JSON with structure):
  ```json
  {
    "format": "vault-share",
    "version": 1,
    "projectName": "AWS Staging",
    "createdAt": "...",
    "expiresAt": "..." or null,
    "secretCount": 5,
    "salt": [...],
    "iv": [...],
    "ct": [...]
  }
  ```
- The `ct` contains the AES-GCM encrypted JSON of `{ secrets: [...] }`, keyed from the share password via PBKDF2

### Receiving a Share
- Add an "Import Share" option (in the import button dropdown, or as a separate button)
- Accept `.vault-share` files
- Prompt for the share password
- Decrypt and show a preview of the secrets (labels and types only, not values)
- Check expiry — if expired, show warning: "This share expired on [date]. The sender may have intended for these credentials to be rotated. Import anyway?"
- Let the user choose which project to import into (existing or create new)

### UX Details
- The share file should be small enough to send via any channel (Slack, email, AirDrop)
- Show a "Share password" copy button in the creation modal so the user can send the password via a separate channel from the file
- The share does NOT include secret history — only current values

---

## 10. Offline-First PWA

**Context:** Vault is a single HTML file with no server dependency. This makes it a perfect candidate for a Progressive Web App — installable from the browser, works offline, feels like a native app. This reinforces the "no cloud, no server" positioning.

**Explanation for the developer (Warda):**

A Progressive Web App (PWA) is a way to make a website installable on a user's device — it shows up in the dock/taskbar, launches in its own window (no browser chrome), and works fully offline. It requires two things:

1. **A Web App Manifest** (`manifest.json`) — a JSON file that tells the browser the app's name, icons, theme color, and display mode. This is what triggers the browser's "Install App" prompt.

2. **A Service Worker** (`sw.js`) — a JavaScript file that runs in the background and intercepts network requests. For Vault, the service worker would cache the HTML file itself so it loads instantly even with no internet. Since Vault has zero server calls, the service worker is simple: cache everything on install, serve from cache always.

### What to implement

**manifest.json:**
```json
{
  "name": "Vault — Local Secrets Manager",
  "short_name": "Vault",
  "description": "Offline-first developer secrets manager. No server, no cloud.",
  "start_url": "/vault.html",
  "display": "standalone",
  "background_color": "#0a0a0b",
  "theme_color": "#0a0a0b",
  "icons": [
    { "src": "icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

**Service Worker (sw.js):**
- On `install`: cache `vault.html`, `manifest.json`, and icons
- On `fetch`: serve from cache first, fall back to network (cache-first strategy)
- On `activate`: clean up old caches if the version changes
- Version the cache name (e.g., `vault-v1`) so updates can bust the old cache

**In vault.html:**
- Add `<link rel="manifest" href="manifest.json">` in the `<head>`
- Add a service worker registration script:
  ```js
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js');
  }
  ```
- Add `<meta name="theme-color" content="#0a0a0b">` for mobile browsers
- Add `<meta name="apple-mobile-web-app-capable" content="yes">` for iOS

**Icons:** You'll need to create a simple lock icon in the Vault purple (#a78bfa) at 192x192 and 512x512. Can be an SVG converted to PNG, or generated programmatically.

**Trade-off:** This moves Vault from a single HTML file to a small folder (vault.html, manifest.json, sw.js, icons). That's fine — it's still zero-server, zero-build. You just serve the folder from anywhere (even `python -m http.server`). Alternatively, you could inline the manifest as a data URI and the service worker as a blob URL to keep it single-file, but that's fragile and not worth it.

---

## 11. Project Links to External Spaces

**Context:** A project in Vault (e.g., "AWS Production") usually corresponds to real infrastructure — a GitHub repo, an AWS console, a Vercel dashboard, a Notion doc. Currently, there's no way to link a Vault project to these external resources.

**Task:** Add a "Links" section to each project:

### Data Model
Add a `links` array to each project:
```json
{
  "id": "...",
  "name": "AWS Production",
  "links": [
    { "id": "...", "label": "GitHub Repo", "url": "https://github.com/org/repo", "icon": "github" },
    { "id": "...", "label": "AWS Console", "url": "https://console.aws.amazon.com/...", "icon": "aws" },
    { "id": "...", "label": "Vercel Dashboard", "url": "https://vercel.com/...", "icon": "link" }
  ],
  ...
}
```

### UI — Project Header
- Below the project name (and above the tag filter bar), show a horizontal row of link chips
- Each chip shows: a small icon (favicon-style), the label, and is clickable (opens URL in new tab)
- Hovering a chip shows the full URL in a tooltip
- An "+" button at the end of the row opens an inline form to add a new link

### Add/Edit Link Form
- **Label** — text input (e.g., "GitHub Repo")
- **URL** — text input with validation (must start with `http://` or `https://`)
- **Icon** — auto-detect from URL domain:
  - `github.com` → GitHub icon
  - `gitlab.com` → GitLab icon
  - `aws.amazon.com` or `console.aws` → AWS icon
  - `vercel.com` → Vercel icon
  - `notion.so` → Notion icon
  - `linear.app` → Linear icon
  - `jira.atlassian.com` → Jira icon
  - `slack.com` → Slack icon
  - Everything else → generic link icon
- Allow manual icon override from the list above

### Quick Access
- In the sidebar project item, show a small link icon indicator if the project has links (subtle, doesn't clutter)
- Consider: `Cmd/Ctrl + Shift + L` to open links panel for current project

### Icons
Use simple inline SVGs for the service icons (not external dependencies). Keep them minimal — 16x16, single-color, matching the existing icon style.
