# KeyRing — Brand Kit Schema: Logo Sets & Usage Rules

This document specifies two additions to the brand kit schema:

1. **Logo Sets** — named groups of logo variants (e.g. "Primary mark",
   "Wordmark light", "Wordmark dark") replacing the flat logos[] array
2. **Logo Usage Rules** — a structured field for do's, don'ts, minimum
   size, clear space, and any other usage restrictions

Read alongside CLAUDE.md and BRAND-KIT-IMPROVEMENTS.md.
This is an addendum to Task 3 (Branding Storage Per Project).

---

## Problem With Current Schema

The current logo data model is a flat array:

```json
"logos": [
  { "id": "...", "name": "KeyRing Logo", "variant": "default", "dataUrl": "..." }
]
```

This has several gaps:
- No way to group related files (e.g. the light and dark versions of the
  same wordmark are logically one asset, not two unrelated logos)
- No usage context per set — when should you use the icon-only vs wordmark?
- No structured rules — minimum size, clear space, do's and don'ts live
  only in the free-text `notes` field which Claude cannot parse reliably
- No file path reference — for assets stored on disk (not as base64),
  there's no way to point to a file path

---

## Updated Schema

### Logo Sets — replace `logos[]` with `logoSets[]`

Each "set" is a named logo concept (e.g. "Primary mark", "Wordmark").
Each set contains one or more files (the actual image assets), each
with its own variant tag.

```json
{
  "logoSets": [
    {
      "id": "uuid",
      "name": "Primary mark",
      "description": "The icon-only mark. Use for favicons, app icons, PWA manifest.",
      "files": [
        {
          "id": "uuid",
          "variant": "default",
          "label": "On Deep Violet background",
          "dataUrl": "data:image/svg+xml;base64,...",
          "filePath": "/Realmdrop/Branding/keyring-logo.svg"
        },
        {
          "id": "uuid",
          "variant": "transparent",
          "label": "Transparent background",
          "dataUrl": "data:image/svg+xml;base64,...",
          "filePath": "/Realmdrop/Branding/keyring-logo-transparent.svg"
        }
      ]
    },
    {
      "id": "uuid",
      "name": "Wordmark",
      "description": "Full lockup with KeyRing + by Realmdrop. Always use when space allows.",
      "files": [
        {
          "id": "uuid",
          "variant": "light",
          "label": "Light mode",
          "dataUrl": "data:image/svg+xml;base64,...",
          "filePath": "/Realmdrop/Branding/keyring-wordmark-light.svg"
        },
        {
          "id": "uuid",
          "variant": "dark",
          "label": "Dark mode",
          "dataUrl": "data:image/svg+xml;base64,...",
          "filePath": "/Realmdrop/Branding/keyring-wordmark-dark.svg"
        }
      ]
    }
  ]
}
```

**File fields:**
- `id` — unique identifier
- `variant` — one of: `default`, `light`, `dark`, `transparent`,
  `icon`, `wordmark`, `on-brand`, `other`
- `label` — short human description of when to use this specific file
  (e.g. "On Deep Violet background", "Light mode", "Favicon")
- `dataUrl` — base64 encoded image (nullable — may be null if only
  filePath is provided)
- `filePath` — optional absolute or relative path to the file on disk.
  Used when the asset is stored as a file rather than base64. Nullable.

At least one of `dataUrl` or `filePath` must be non-null.

**Set fields:**
- `id` — unique identifier
- `name` — the concept name, e.g. "Primary mark", "Wordmark", "Icon only"
- `description` — one sentence on when to use this set

---

### Logo Usage Rules — new top-level branding field `logoUsageRules`

A structured object at the same level as `colors`, `typography`, etc.

```json
{
  "logoUsageRules": {
    "minimumSize": "20px",
    "clearSpace": "50% of the mark's width on all sides",
    "do": [
      "Always pair the wordmark with 'by Realmdrop' attribution",
      "Use the dark wordmark on dark backgrounds (#12062E and below)",
      "Use the transparent mark when placing on custom colored surfaces",
      "Maintain the mint dot (#00FFB2) at all times — never recolor it"
    ],
    "doNot": [
      "Do not recolor the mark — Deep Violet background, white strokes, mint dot only",
      "Do not rotate or distort the mark",
      "Do not display 'KeyRing' wordmark without 'by Realmdrop'",
      "Do not use the mark below 20px",
      "Do not use on backgrounds that clash with Deep Violet"
    ],
    "backgroundGuidance": "On Deep Violet (#2E1065): use the on-brand variant with rgba(255,255,255,0.12) tile. On light surfaces: use the default (violet square) or transparent mark. On dark surfaces: use default mark, dark wordmark.",
    "notes": ""
  }
}
```

**Fields:**
- `minimumSize` — string, e.g. "20px", "16px", "0.5 inch"
- `clearSpace` — string describing the required padding around the mark
- `do` — array of strings, positive usage rules
- `doNot` — array of strings, negative usage rules / prohibitions
- `backgroundGuidance` — string, guidance on which variant to use on
  which background
- `notes` — free-form overflow text

---

## Migration

### Backward compatibility

The old flat `logos[]` array must be preserved during migration to avoid
data loss. On vault load, if a project has `logos[]` but no `logoSets[]`:

```js
if (!p.branding.logoSets) {
  p.branding.logoSets = [];
  // Migrate existing logos into a single set called "Logos"
  if (p.branding.logos && p.branding.logos.length > 0) {
    p.branding.logoSets.push({
      id: uid(),
      name: 'Logos',
      description: '',
      files: p.branding.logos.map(l => ({
        id: uid(),
        variant: l.variant || 'default',
        label: l.name || '',
        dataUrl: l.dataUrl || null,
        filePath: null
      }))
    });
  }
  // Keep logos[] for one version, then deprecate
}
if (!p.branding.logoUsageRules) {
  p.branding.logoUsageRules = {
    minimumSize: '',
    clearSpace: '',
    do: [],
    doNot: [],
    backgroundGuidance: '',
    notes: ''
  };
}
```

The old `logos[]` field should remain in the data model (just hidden from
the UI) for one version so that any exports made before this update can
still be imported without data loss. After the next stable release, it
can be removed from the migration and schema.

---

## UI Changes

### Logo Sets section

Replace the current "Logos & Assets" section in the Brand Kit tab with
a "Logo Sets" section.

**Set-level UI:**
- Each set renders as a collapsible card with the set name as the header
- Below the name: a short `description` text input (one line)
- Below that: a horizontal row of file thumbnails (one per file in the set)
- A "+ Add file" upload button at the end of the file row
- A remove (×) button on the set card header to delete the whole set
- An "+ Add logo set" button at the bottom of the section

**File-level UI (inside a set):**
- Each file shows: a thumbnail preview (48×48px, object-fit: contain,
  background: var(--bg-input), border-radius: var(--radius))
- Below the thumbnail: a `variant` dropdown (default, light, dark,
  transparent, icon, wordmark, on-brand, other)
- Below that: a `label` text input, placeholder "e.g. Light mode"
- If `filePath` is set (no dataUrl): show the path as muted mono text
  instead of a thumbnail, with a small folder icon
- A (×) remove button on each file thumbnail

**Set naming:**
When "+ Add logo set" is clicked, show an inline input for the set name
before creating it. Don't create unnamed sets.

---

### Logo Usage Rules section

Add a new collapsible section below "Logo Sets" titled "Logo usage rules".

**UI layout:**

Row 1 — two fields side by side:
- Minimum size: text input, placeholder "e.g. 20px"
- Clear space: text input, placeholder "e.g. 50% of mark width on all sides"

Row 2 — Do's (tag/chip input, same pattern as voice keywords):
- Label: "Do"
- Chip input: type rule and press Enter to add
- Each chip shown in green-tinted style (var(--type-ssh-bg) / var(--type-ssh-fg))

Row 3 — Don'ts (tag/chip input):
- Label: "Don't"
- Chip input: type rule and press Enter to add
- Each chip shown in red-tinted style (var(--type-password-bg) / var(--type-password-fg))

Row 4 — Background guidance:
- Textarea, 2 rows, placeholder "Guidance on which variant to use on which background color or surface"

Row 5 — Notes:
- Textarea, 2 rows, placeholder "Any other usage rules or restrictions"

All fields auto-save on blur, same pattern as rest of brand kit.

---

## Export Changes

### exportBrandKit()

Add `logoSets` and `logoUsageRules` to the exported JSON:

```js
const data = {
  format: 'keyring-brandkit',
  version: 3,
  projectName: project.name,
  exportedAt: new Date().toISOString(),
  branding: {
    name: b.name,
    tagline: b.tagline,
    wordmarkNotes: b.wordmarkNotes,
    colors: b.colors.map(c => ({ name: c.name, hex: c.hex, role: c.role, usage: c.usage })),
    typography: b.typography.map(t => ({ ... })),
    logoSets: (b.logoSets || []).map(set => ({
      name: set.name,
      description: set.description,
      files: set.files.map(f => ({
        variant: f.variant,
        label: f.label,
        dataUrl: f.dataUrl || null,
        filePath: f.filePath || null
      }))
    })),
    logoUsageRules: b.logoUsageRules || {
      minimumSize: '', clearSpace: '', do: [], doNot: [],
      backgroundGuidance: '', notes: ''
    },
    voice: b.voice,
    subBrands: b.subBrands,
    notes: b.notes
  }
};
```

Bump version to `3` to signal the schema change.

### importBrandKit()

Add handling for `logoSets` and `logoUsageRules` in the merge logic:

```js
// Logo sets (skip duplicate sets by name)
(incoming.logoSets || []).forEach(set => {
  if (!b.logoSets) b.logoSets = [];
  const existing = b.logoSets.find(s => s.name === set.name);
  if (!existing) {
    b.logoSets.push({
      id: uid(),
      name: set.name,
      description: set.description || '',
      files: (set.files || []).map(f => ({
        id: uid(),
        variant: f.variant || 'default',
        label: f.label || '',
        dataUrl: f.dataUrl || null,
        filePath: f.filePath || null
      }))
    });
  } else {
    // Merge files into existing set (skip duplicate variants+labels)
    (set.files || []).forEach(f => {
      const dupFile = existing.files.find(
        ef => ef.variant === f.variant && ef.label === f.label
      );
      if (!dupFile) {
        existing.files.push({
          id: uid(),
          variant: f.variant || 'default',
          label: f.label || '',
          dataUrl: f.dataUrl || null,
          filePath: f.filePath || null
        });
      }
    });
  }
});

// Logo usage rules (overwrite if incoming has values)
if (incoming.logoUsageRules) {
  if (!b.logoUsageRules) {
    b.logoUsageRules = { minimumSize: '', clearSpace: '', do: [], doNot: [],
                         backgroundGuidance: '', notes: '' };
  }
  const r = b.logoUsageRules;
  const ir = incoming.logoUsageRules;
  if (ir.minimumSize) r.minimumSize = ir.minimumSize;
  if (ir.clearSpace) r.clearSpace = ir.clearSpace;
  (ir.do || []).forEach(rule => { if (!r.do.includes(rule)) r.do.push(rule); });
  (ir.doNot || []).forEach(rule => { if (!r.doNot.includes(rule)) r.doNot.push(rule); });
  if (ir.backgroundGuidance) r.backgroundGuidance = ir.backgroundGuidance;
  if (ir.notes) r.notes = ir.notes;
}
```

### exportBrandForClaude() — markdown output

Update the Logos section in the markdown export:

```markdown
## Logo sets

### Primary mark
Icon-only mark. Use for favicons, app icons, PWA manifest.

| Variant | Label | File |
|---------|-------|------|
| Default | On Deep Violet background | keyring-logo.svg |
| Transparent | Transparent background | keyring-logo-transparent.svg |

### Wordmark
Full lockup with KeyRing + by Realmdrop. Always use when space allows.

| Variant | Label | File |
|---------|-------|------|
| Light | Light mode | keyring-wordmark-light.svg |
| Dark | Dark mode | keyring-wordmark-dark.svg |

## Logo usage rules
- **Minimum size:** 20px
- **Clear space:** 50% of mark width on all sides
- **Background guidance:** On Deep Violet: use on-brand variant...

**Do:**
- Always pair the wordmark with 'by Realmdrop' attribution
- ...

**Don't:**
- Do not recolor the mark
- ...
```

For the file column: use `filePath` basename if available, else use
the `label`, else use the `variant`.

---

## Implementation Order

1. **Data model + migration** — add `logoSets[]` and `logoUsageRules` to
   branding schema, migrate existing `logos[]` on vault load
2. **Logo Sets UI** — replace Logos section with set-grouped UI
3. **Logo Usage Rules UI** — new section below Logo Sets
4. **exportBrandKit()** — add new fields, bump to version 3
5. **importBrandKit()** — handle logoSets merge and logoUsageRules merge
6. **exportBrandForClaude()** — update markdown output

---

## Verification Checklist

- [ ] Existing logos[] data migrates into a "Logos" set on first load
- [ ] No data lost during migration
- [ ] Can create named logo sets with description
- [ ] Can upload multiple files per set with variant + label
- [ ] filePath field saves and displays correctly
- [ ] Logo Usage Rules section saves do/don't chips, min size, clear space
- [ ] Export JSON contains logoSets and logoUsageRules at version 3
- [ ] Import merges logoSets by name, files by variant+label
- [ ] Import merges logoUsageRules (appends do/don't, overwrites scalar fields)
- [ ] Export for Claude markdown includes both sections with correct format
- [ ] Old version 2 imports still work (logoSets treated as empty, logos[]
      migrated if present in incoming file)
