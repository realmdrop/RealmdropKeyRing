# KeyRing — Post-Review Fixes

Three issues found during code review of the completed vault.html.
Work through these in order. Each is a small, targeted fix — do not
refactor surrounding code unless specifically noted.

---

## FIX 1 — Remove dead wordlist code

**Problem:** A large wordlist array (used for a passphrase generator that was
descoped) is still present in vault.html. It is referenced nowhere in the active
codebase and adds unnecessary bulk to the file.

**How to find it:** Search for the string `'sauce','sausage'` — this is near the
start of the wordlist array. The array spans several hundred lines.

**Fix:** Delete the entire wordlist array and any associated variable declaration.
Verify nothing references it after removal by searching for the variable name.

**Test:** The file should load and function identically after removal. All features
should work. File size should drop noticeably (est. 200–300 lines removed).

---

## FIX 2 — Replace remaining `prompt()` with a proper modal (logo name on upload)

**Problem:** One `prompt()` dialog remains in the codebase. When a user uploads a
logo image, the logo name is collected via a browser `prompt()` call:

```js
const name = prompt('Logo name:', file.name.replace(/\.[^.]+$/, '')) || file.name;
```

This is inconsistent with the rest of the app — typography entry, confirmations,
and alerts all use proper in-app modals. The `prompt()` blocks the browser thread
and looks out of place in an otherwise polished UI.

**Fix:** Replace the `prompt()` with a small inline modal using the existing modal
pattern (`modal-overlay` + `modal-box`). The modal should:

- Have a single text input pre-populated with the filename (without extension),
  same as the current `prompt()` default
- Have a "Cancel" button (discards the upload) and a "Add Logo" button (confirms)
- Focus the input automatically on open (`setTimeout(() => input.focus(), 100)`)
- Submit on `Enter` key, cancel on `Escape`
- Follow the exact same HTML structure and CSS classes as the Typography modal
  (`typo-modal`) — reuse those styles, do not introduce new ones
- After the user confirms, proceed with the existing logo push and `saveBrand()` /
  `renderBrandingTab()` calls

**Note:** There should be zero `prompt()` calls remaining in the file after this fix.
Verify with a search for `prompt(` before committing.

---

## FIX 3 — Unify version numbers

**Problem:** The vault's internal data model and the export files use inconsistent
version numbers, which will cause confusion if a future migration needs to detect
which format a file came from.

Current state:
- Internal vault object: `version: 1`
- Brand kit JSON export: `version: 2`
- Encrypted vault export: `version: 1`
- Plaintext vault export: `version: 1`
- `.vault-share` export: `version: 1`

The brand kit expanded the data model significantly (voice, subBrands, color roles,
typography extras, logo variants, wordmarkNotes). This warrants bumping the internal
vault version to `2` so future migrations can detect old vaults and apply defaults.

**Fix:**

1. Change the internal vault initialisation from `version: 1` to `version: 2`:
   - `let vault = { version: 2, projects: [] }`
   - All `vault = { version: 1, projects: [] }` reset statements → `version: 2`

2. Change all vault export payloads from `version: 1` to `version: 2`:
   - Encrypted export (vault-pw mode)
   - Encrypted export (custom-pw mode)
   - Plaintext export

3. The `.vault-share` format is its own independent format — leave it at `version: 1`
   since it has its own format identifier (`"format": "vault-share"`).

4. The brand kit JSON export (`version: 2`) is already correct — no change needed.

5. Add a one-time migration check on vault load: if `vault.version === 1`, run the
   existing branding field migration (add voice, subBrands, wordmarkNotes defaults,
   backfill color/typography/logo fields) and set `vault.version = 2`. This ensures
   existing users' data is silently upgraded on first load after the update.

   ```js
   // In the load/decrypt block, after parsing vault JSON:
   if (!vault.version || vault.version < 2) {
     // run existing branding migration block here
     vault.version = 2;
     saveVault(); // persist the upgraded version
   }
   ```

**Test:** After fix, search the file for `version: 1` — only `.vault-share` payloads
should remain. All other version references should read `version: 2`.

---

## Verification Checklist

After all three fixes, confirm:

- [ ] No `prompt(` calls exist anywhere in vault.html
- [ ] No wordlist array exists (search for `'sauce','sausage'`)
- [ ] `version: 1` only appears in `.vault-share` related code
- [ ] `version: 2` appears in vault init, vault resets, and all vault exports
- [ ] App loads, unlocks, and all features work as before
- [ ] Logo upload shows a proper modal instead of a browser prompt
- [ ] File size is smaller than before (wordlist removal)
