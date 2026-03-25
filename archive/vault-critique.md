# Vault — Code Critique

## Overall Impression

This is a well-structured single-file local password manager with a clean dark UI, client-side AES-GCM encryption, and thoughtful UX touches (auto-lock, clipboard clearing, tag filtering). For a single HTML file, it's impressive. That said, there are several security issues — some serious — and a few architectural and UX improvements worth making.

---

## Security Issues

### CRITICAL: Password Verification via SHA-256 Hash

**Lines 1092–1112** — The master password is verified by comparing `SHA-256(password)` against a stored hash. This is a major problem:

- **SHA-256 is not a password hashing function.** It's fast and unsalted, making it trivially brute-forceable with modern GPUs (billions of hashes/second).
- The app already derives a PBKDF2 key (100k iterations) for encryption — but the *authentication check* bypasses that entirely. An attacker with access to localStorage only needs to crack the SHA-256 hash to know the password, then they can derive the PBKDF2 key themselves.

**Fix:** Remove the SHA-256 hash entirely. Instead, store a known "test ciphertext" encrypted with the derived key. On unlock, derive the key and try to decrypt that test block — if decryption fails, the password is wrong. This is how tools like 1Password and KeePass work.

### HIGH: Export Dumps Plaintext JSON

**Lines 1617–1626** — The export function writes `vault` as unencrypted JSON. Anyone who intercepts or finds this file has every secret in plaintext.

**Fix:** Encrypt the export with the same AES-GCM key (or prompt for a separate export password). At minimum, warn the user prominently.

### HIGH: Import Accepts Untrusted Data Without Validation

**Lines 1628–1669** — The import merges arbitrary JSON into the vault. It only checks that `data.projects` is an array, but doesn't validate the shape of individual secrets. Malformed data (missing `value`, wrong types, XSS payloads in `label`/`notes`) could corrupt the vault or cause runtime errors.

**Fix:** Validate each imported project and secret against a strict schema before merging. Reject or skip malformed entries.

### MEDIUM: localStorage Is Vulnerable to XSS

All encrypted data lives in `localStorage`. If any XSS vector exists (even via a browser extension), an attacker can exfiltrate the encrypted blob *and* the SHA-256 password hash. Since the hash is fast to crack (see above), this effectively leaks everything.

**Mitigation:** This is somewhat inherent to the "single HTML file" design. But removing the SHA-256 hash (see above) would at least force an attacker to brute-force the PBKDF2 key, which is far harder.

### MEDIUM: No Brute-Force Protection on Unlock

There's no rate-limiting or lockout after failed unlock attempts. An automated script interacting with the page could try thousands of passwords per second.

**Fix:** Add exponential backoff (e.g., 1s, 2s, 4s, 8s delay after consecutive failures) or a hard lockout after N attempts.

### LOW: Clipboard Clearing Is Best-Effort

**Lines 1373–1380** — `navigator.clipboard.writeText('')` after 30 seconds is a nice touch, but clipboard managers (e.g., macOS clipboard history, Windows Clipboard History) will have already captured the value. This is essentially impossible to fix in a browser, but it's worth noting in user-facing documentation.

---

## Architecture & Code Quality

### No Password Strength Enforcement

**Line 1089** — Only checks `pw.length < 8`. No checks for complexity, common passwords, or entropy. A password like `12345678` passes.

**Suggestion:** Integrate a lightweight strength estimator (even a simple entropy check) and show a strength meter on the setup screen.

### innerHTML With User Content

**Lines 1201–1208, 1327–1355, 1548–1557** — While there's an `esc()` function used in most places, the confirm dialog directly injects `title` and `message` parameters into innerHTML:

```js
overlay.innerHTML = `
  <div class="confirm-box">
    <h3>${title}</h3>
    <p>${message}</p>
  ...
```

These are called with strings like `Delete "${p.name}"?` where `p.name` has already been through `esc()` at the call site — but this pattern is fragile. If anyone adds a new `confirmDialog()` call and forgets to escape, it's an XSS vector.

**Fix:** Use `textContent` assignment instead of innerHTML for user-provided text, or escape inside `confirmDialog` itself.

### Monolithic Single File

At ~1,770 lines, this is at the upper limit of maintainability for a single file. The CSS, HTML, and JS are all inline. This makes it hard to test, lint, or review individual concerns.

**Suggestion for scaling:** If this tool is actively developed, consider splitting into separate files with a build step (even a simple one like `esbuild --bundle`). This would also let you add unit tests for the crypto functions.

### No Versioned Data Schema

The vault object `{ projects: [] }` has no version field. If you ever need to migrate the data format, there's no way to detect the old format vs. the new one.

**Fix:** Add a `version: 1` field to the vault and check it on load.

### Secret Value in Memory

When the vault is unlocked, all secrets live as plaintext JavaScript strings in `vault.projects[].secrets[].value`. This is visible in browser DevTools (just type `vault` in the console — although the IIFE scoping helps here). The `lockVault()` function does overwrite values with null bytes, which is good, but the JS garbage collector may retain the original strings.

This is largely unavoidable in a browser context, but worth acknowledging in documentation.

---

## UX Observations

### Good

- **Auto-lock with countdown warning** — Thoughtful and well-implemented.
- **Clipboard auto-clear** — Good security hygiene.
- **Tag system with filtering** — Adds real organizational value.
- **Masked values by default** — Correct default for a secrets manager.
- **Keyboard shortcuts** — Enter to submit forms, Escape to close modals.
- **Clean, cohesive dark theme** — The design is polished.

### Could Improve

- **No password generator** — A password manager without a "Generate Strong Password" button is missing a key feature. This would be easy to add.
- **No indication of password strength on setup** — Users get an 8-character minimum but no feedback on whether their password is strong.
- **Export should be more prominent about being unencrypted** — Currently it just downloads silently.
- **No way to reorder projects or secrets** — Drag-and-drop or manual sorting would help with larger vaults.
- **Search doesn't match on secret values** — Intentional (probably for security), but could be a toggle.
- **No confirmation before export** — Exporting all secrets to a plaintext file should require a confirmation step.

---

## Summary

| Area | Grade | Notes |
|------|-------|-------|
| UI/UX Design | **A-** | Polished, cohesive, well-thought-out interactions |
| Code Quality | **B** | Clean and readable, but monolithic; fragile innerHTML patterns |
| Cryptography | **C+** | AES-GCM + PBKDF2 is solid, but the SHA-256 auth check undermines it |
| Security Posture | **C** | Plaintext export, no brute-force protection, localStorage exposure |
| Feature Completeness | **B-** | Missing password generator, strength meter, encrypted export |

The #1 thing to fix is **replacing the SHA-256 password check with a decrypt-based verification**. That single change significantly raises the security bar. After that, encrypting the export and adding a password generator would round out the tool nicely.
