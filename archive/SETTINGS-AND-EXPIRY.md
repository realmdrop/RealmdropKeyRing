# KeyRing — Settings Panel, Session Persistence & Secret Expiry

This document specifies three things:
1. A proper Settings panel (modal)
2. Session persistence via sessionStorage (stay unlocked on refresh)
3. Secret rotation reminders (rotateDue field + dashboard indicator)

Read alongside CLAUDE.md, FIXES.md, SUBPROJECTS.md.

---

## FEATURE 1 — Settings Panel

### Overview

Add a Settings modal accessible from a gear icon (⚙) in the top bar,
next to the existing theme toggle and shortcuts (?) button.

Keyboard shortcut: `Cmd/Ctrl + ,` (already listed in shortcuts overlay,
currently a no-op — wire it up here).

### Modal structure

Use the existing `.modal-overlay` + `.modal-box` pattern. Title: "Settings"

The modal has three sections, each visually separated by a border:

---

### Section 1 — Session

**Setting: Stay unlocked on refresh**

A toggle switch (on/off). Default: OFF.

Label: "Stay unlocked when page refreshes"

Description text below the label (font-size: 12px, color: var(--text-dim)):
"When enabled, refreshing the page will not lock your vault. Your session
key is stored in your browser's sessionStorage — it is cleared automatically
when you close this tab. This is convenient but means anyone with access to
your open browser tab can access your vault without the master password."

Security note (shown only when toggle is ON):
A small warning row with an amber warning icon and the text:
"Session key is active. Closing this tab will lock the vault."
Style: background var(--warning-bg), border var(--warning-border),
border-radius var(--radius), padding 8px 12px, font-size 12px.

**Behaviour when enabled:**
- After successful unlock, serialize and store the session key in
  sessionStorage under the key `kr_session`.
- The session key format:
  ```json
  {
    "keyData": [...],
    "salt": [...],
    "iv": [...]
  }
  ```
  Where `keyData` is the raw bytes of the exported CryptoKey
  (`await crypto.subtle.exportKey('raw', cryptoKey)`), `salt` is
  the existing vault salt, and `iv` is not needed here — store
  just keyData and salt as Arrays.

- On page load, before showing the login screen, check sessionStorage
  for `kr_session`. If found and the setting is enabled:
  1. Parse the stored data
  2. Re-import the CryptoKey:
     ```js
     const keyData = new Uint8Array(parsed.keyData);
     cryptoKey = await crypto.subtle.importKey(
       'raw', keyData,
       { name: 'AES-GCM' },
       false,
       ['encrypt', 'decrypt']
     );
     ```
  3. Attempt to decrypt the vault. If successful, skip login and go
     straight to the app screen.
  4. If decryption fails (corrupted session key), clear sessionStorage
     and show login as normal.

- On `lockVault()`: always clear `sessionStorage.removeItem('kr_session')`
  regardless of whether the setting is on or off.

- On setting toggle OFF: immediately clear `kr_session` from sessionStorage.

**Persistence of the setting itself:**
Store in localStorage as `kr_setting_session` (boolean string "true"/"false").
This is not sensitive data — it's just a preference flag.

---

### Section 2 — Auto-lock

**Setting: Auto-lock timeout**

Replace the hardcoded `const AUTO_LOCK_MS = 15 * 60 * 1000` with a
user-configurable value read from localStorage (`kr_setting_lock_timeout`).

UI: A `<select>` dropdown with these options:

| Display label          | Value (minutes) |
|------------------------|-----------------|
| 5 minutes              | 5               |
| 10 minutes             | 10              |
| 15 minutes (default)   | 15              |
| 30 minutes             | 30              |
| 1 hour                 | 60              |
| 2 hours                | 120             |
| Never (not recommended)| 0               |

Label: "Lock vault after inactivity"

Description text:
"The vault automatically locks after this period of inactivity. Shorter
timeouts are more secure. 'Never' disables auto-lock entirely — only
recommended if your device is private and you lock manually."

When "Never" is selected, show a small amber warning:
"Auto-lock is disabled. Lock your vault manually when stepping away."

**Behaviour:**
- On change, save to `localStorage.setItem('kr_setting_lock_timeout', value)`
- Immediately call `resetAutoLock()` with the new timeout value
- If value is 0 (Never): clear all auto-lock timers and do not start new ones
- The `AUTO_LOCK_MS` constant should become a function `getAutoLockMs()`:
  ```js
  function getAutoLockMs() {
    const mins = parseInt(localStorage.getItem('kr_setting_lock_timeout') || '15');
    return mins === 0 ? 0 : mins * 60 * 1000;
  }
  ```
- Update `resetAutoLock()` to call `getAutoLockMs()` at runtime rather than
  using the hardcoded constant.
- On app init, read the stored value and populate the dropdown accordingly.

---

### Section 3 — Clipboard

**Setting: Clipboard clear timeout**

A `<select>` dropdown.

Label: "Clear clipboard after copying a secret"

Options:

| Display label    | Value (seconds) |
|------------------|-----------------|
| 15 seconds       | 15              |
| 30 seconds (default) | 30          |
| 60 seconds       | 60              |
| 2 minutes        | 120             |
| Never clear      | 0               |

Description text:
"After copying a secret value, KeyRing clears your clipboard after this
delay to prevent accidental exposure. 'Never clear' leaves it to you."

Store as `kr_setting_clipboard_timeout` in localStorage.

Update the existing clipboard-clear logic to read this value at copy time:
```js
function getClipboardClearMs() {
  const secs = parseInt(localStorage.getItem('kr_setting_clipboard_timeout') || '30');
  return secs === 0 ? 0 : secs * 1000;
}
```
If value is 0: skip the `setTimeout` that clears the clipboard entirely.

---

### Settings modal CSS (new, minimal)

```css
.settings-section {
  padding: 16px 0;
  border-bottom: 0.5px solid var(--border);
}
.settings-section:last-child { border-bottom: none; }
.settings-section-title {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.5px;
  text-transform: uppercase;
  color: var(--text-dim);
  margin-bottom: 14px;
}
.settings-row {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 12px;
}
.settings-row:last-child { margin-bottom: 0; }
.settings-label-group { flex: 1; }
.settings-label {
  font-size: 13px;
  font-weight: 500;
  color: var(--text);
  margin-bottom: 3px;
}
.settings-desc {
  font-size: 12px;
  color: var(--text-dim);
  line-height: 1.5;
}
.settings-warning {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 8px 12px;
  background: var(--warning-bg);
  border: 0.5px solid var(--warning-border);
  border-radius: var(--radius);
  font-size: 12px;
  color: var(--text);
  margin-top: 8px;
  line-height: 1.5;
}

/* Toggle switch */
.toggle-switch {
  position: relative;
  width: 36px;
  height: 20px;
  flex-shrink: 0;
  margin-top: 1px;
}
.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
  position: absolute;
}
.toggle-track {
  position: absolute;
  inset: 0;
  background: var(--border-focus);
  border-radius: 10px;
  transition: background var(--transition);
  cursor: pointer;
}
.toggle-switch input:checked + .toggle-track { background: var(--accent); }
.toggle-thumb {
  position: absolute;
  top: 3px;
  left: 3px;
  width: 14px;
  height: 14px;
  background: white;
  border-radius: 50%;
  transition: transform var(--transition);
  pointer-events: none;
}
.toggle-switch input:checked ~ .toggle-thumb { transform: translateX(16px); }
```

---

## FEATURE 2 — Secret Rotation Reminders

### Data model

Add an optional `rotateDue` field to each secret:

```json
{
  "id": "...",
  "label": "Stripe API Key",
  "type": "api_key",
  "value": "sk_live_...",
  "rotateDue": "2026-06-01T00:00:00.000Z",
  "history": [...],
  ...
}
```

`rotateDue` is either an ISO 8601 date string or `null` (default, no reminder set).

**Migration:** On vault load, add `if (!('rotateDue' in s)) s.rotateDue = null;`
to the existing secret migration block. Silent, no prompt.

### UI — setting rotation on a secret

In the **edit secret modal**, add a new optional field below the Notes field:

Label: "Rotation reminder"
Input: `<input type="date">` — shows a date picker
Helper text: "Set a date to be reminded to rotate this secret."

If a date is set, show a small pill on the secret card:
- Not due yet (> 7 days away): no indicator
- Due soon (≤ 7 days): amber pill "Due soon"
- Overdue (past date): red pill "Overdue"

Pill styling uses existing type badge CSS — add two new type classes:
```css
.rotate-soon { background: var(--warning-bg); color: var(--warning); border: 0.5px solid var(--warning-border); }
.rotate-overdue { background: var(--type-password-bg); color: var(--danger); border: 0.5px solid var(--danger); }
```

### Dashboard indicator

In the project header (where "X secrets" count is shown), add a secondary
indicator if any secrets in the project have rotation reminders due:

- If any are overdue: show a small red dot + "N overdue" text in danger color
- If any are due within 7 days (and none overdue): show amber dot + "N due soon"
- If none are due: show nothing

Keep it subtle — same size as existing meta text (font-size: 12px).

### Filter by rotation status

In the existing tag filter bar, add two special filter pills at the end:
- "Overdue" — filters to secrets where `rotateDue` is in the past
- "Due soon" — filters to secrets where `rotateDue` is within 7 days

Only show these pills if at least one secret in the project has a `rotateDue`
set. Don't show them if all secrets have `rotateDue: null`.

### On rotate (history entry)

When a secret's value is changed (triggering a history entry), if `rotateDue`
is set, prompt: "Reset rotation reminder?" with a new date input pre-filled
to 90 days from today. The user can accept, change the date, or dismiss.
If accepted, update `rotateDue` to the new date. If dismissed, leave it as-is.

### Export for Claude — rotation data

In the `exportBrandForClaude` markdown output, do NOT include rotation dates —
those are operational data, not brand data. No change needed there.

In the `.env` export, do NOT include rotation dates either. Rotation data
stays inside the vault only.

---

## Implementation Order

1. **Settings modal shell** — gear icon in top bar, modal opens, Cmd+, wires up
2. **Section 2: Auto-lock timeout** — replace hardcoded constant, dropdown, persist
3. **Section 3: Clipboard timeout** — dropdown, update copy logic
4. **Section 1: Session persistence** — toggle, sessionStorage read/write/clear,
   page load check, lockVault() clear (do this last — most complex, test thoroughly)
5. **Secret rotateDue field** — data model + migration
6. **Edit modal date picker** — rotateDue input in edit secret modal
7. **Secret card rotation pills** — due soon / overdue badges
8. **Project header indicator** — N overdue / N due soon
9. **Tag filter rotation pills** — overdue / due soon filter options
10. **Rotation reset prompt** — on secret value change with rotateDue set

---

## Security Notes for Implementation

**sessionStorage safety rules:**
- NEVER store the master password — only the derived CryptoKey bytes
- NEVER store vault data in sessionStorage — only the key
- Always clear on lockVault() regardless of setting state
- If sessionStorage throws (private mode, quota), fail silently and show login
- The exported raw key bytes are AES-256 — 32 bytes. This is safe to store in
  sessionStorage because sessionStorage is tab-scoped and cleared on tab close.
  It is NOT safe to store in localStorage (persists indefinitely).

**Auto-lock "Never" warning:**
- When Never is selected, show the amber warning in the settings modal
- Also update the top bar auto-lock warning span to show "Auto-lock disabled"
  in muted text so the user is always aware of the state
- The lock button still works manually — Never only disables the timer

---

## Verification Checklist

After implementation, confirm:

- [ ] Gear icon opens settings modal, Cmd+, works
- [ ] Auto-lock timeout dropdown persists across page reloads
- [ ] Changing timeout immediately resets the auto-lock timer
- [ ] Never timeout disables timer, shows warning, lock button still works
- [ ] Clipboard timeout dropdown persists, copy logic respects new value
- [ ] Session toggle OFF (default): refresh always shows login
- [ ] Session toggle ON: refresh restores vault without login prompt
- [ ] Session toggle ON: closing tab and reopening shows login
- [ ] Session toggle ON: clicking Lock clears session and shows login
- [ ] Session toggle switched from ON to OFF: immediately clears sessionStorage
- [ ] Session key failure (corrupted): graceful fallback to login, no crash
- [ ] rotateDue null by default on all secrets after migration
- [ ] Date picker in edit modal saves and loads correctly
- [ ] Due soon pill appears ≤ 7 days, overdue pill appears when past date
- [ ] Project header indicator shows correct count
- [ ] Filter pills appear only when rotation dates exist
- [ ] Rotation reset prompt appears on value change when rotateDue is set
