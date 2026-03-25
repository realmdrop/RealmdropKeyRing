# KeyRing — Subproject Support

This document specifies subproject (nested project) support for vault.html.
Read this alongside CLAUDE.md and BRAND-KIT-IMPROVEMENTS.md.
Do not begin implementation without reading all three.

---

## Concept

Any project can optionally have a parent project. A project with a parent is a
"subproject." Subprojects are full projects — they have their own secrets, their
own brand kit, their own links. The only differences are:

1. They appear nested under their parent in the sidebar
2. Their parent's brand kit shows a summary view of all children instead of the
   manual sub-brands section

Nesting is unlimited depth (a subproject can itself have subprojects).

---

## Data Model

Add one field to every project object:

```json
{
  "id": "uuid",
  "name": "CaretCart",
  "parentId": "uuid-of-parent-or-null",
  "color": "#hex",
  "links": [],
  "branding": { ... },
  "secrets": []
}
```

`parentId` is `null` for top-level projects. It holds the `id` of the parent
project for subprojects.

**Migration:** On vault load, add `if (!('parentId' in p)) p.parentId = null;`
to the existing migration block for every project. Silent, no user prompt needed.

---

## Sidebar — Nested Navigation

### Rendering logic

Build a tree from the flat `vault.projects` array:

```js
function buildTree(projects) {
  const map = {};
  const roots = [];
  projects.forEach(p => { map[p.id] = { ...p, children: [] }; });
  projects.forEach(p => {
    if (p.parentId && map[p.parentId]) {
      map[p.parentId].children.push(map[p.id]);
    } else {
      roots.push(map[p.id]);
    }
  });
  return roots;
}
```

Render the sidebar recursively. Each level of nesting adds `16px` of left padding
to the project item. Use the existing `.project-item` styles — do not introduce new
ones. The indentation is the only visual difference between a parent and a child.

### Collapse/expand

Parent projects (those with at least one child) show a small chevron (`▸` / `▾`)
to the left of the project name. Clicking the chevron toggles the children's
visibility. Default state: expanded.

Store collapse state in a `Set` called `collapsedProjects` (in-memory only, not
persisted). When a project is collapsed, its children and all their descendants
are hidden.

The chevron should be `10px`, `color: var(--text-muted)`, and rotate `90deg` when
expanded. Use a CSS transition: `transform 150ms ease`.

### "+ Add subproject" button

On hover of a parent project item in the sidebar, show a small `+` icon button to
the right of the project name (after the existing action icons). Clicking it opens
the new project modal pre-filled with this project as the parent. Label the button
with `title="Add subproject"`. Use the same `btn-icon` class as other icon buttons.

### Subproject visual indicator

Subproject items in the sidebar show a small `↳` prefix in `color: var(--text-muted)`
before the project dot/avatar. This makes the hierarchy legible at a glance without
heavy indentation lines.

---

## Creating a Subproject

### Method 1: From the sidebar (+ button on hover of parent)
Opens the new project modal with the parent pre-selected (see below).

### Method 2: From the new project modal (parent picker)
Add an optional "Parent project" dropdown to the existing new project modal,
below the project name field and above the color picker.

- Label: "Parent project (optional)"
- Options: "None (top-level)" + all existing projects listed by name
- If a project is itself a subproject, show it indented under its parent in the
  dropdown (use `\u00a0\u00a0↳ ` prefix for child items, built from getProjectPath())
- Default: "None (top-level)" — existing behaviour unchanged
- When opened via the sidebar `+` button, pre-select that parent project

When saving, set `parentId` to the selected project's `id`, or `null` if "None"
is selected.

### Method 3: Re-parent existing projects
In the project settings (edit project modal), add the same "Parent project"
dropdown. This lets users re-parent an existing project or promote a subproject
to top-level by selecting "None."

**Cycle prevention:** Never show a project as its own parent. Never show a
project's descendants as parent options (would create a cycle). Filter the dropdown
to exclude the current project and all its descendants.

---

## Deleting a Project With Subprojects

When the user attempts to delete a project that has one or more direct children:

1. Show a `confirmDialog` with this message:
   ```
   "{Project name}" has {N} subproject(s).

   Would you like to promote them to top-level projects before deleting?
   ```
   Two buttons: **"Promote & Delete"** and **"Cancel"**

2. If "Promote & Delete":
   - Set `parentId = null` on all direct children of the deleted project
   - Their own children keep their existing `parentId` — they remain children
     of the now-promoted projects (grandchildren are unaffected)
   - Delete the parent project
   - `saveVault()`

3. If "Cancel":
   - Do nothing. Block the deletion.

There is no "delete all subprojects" option — data safety first.

---

## Brand Kit — Parent Project View

When a project has one or more children
(`vault.projects.some(p => p.parentId === project.id)`), its brand kit replaces
the manual sub-brands section with a live "Subprojects" summary panel.

### What replaces the sub-brands section

A section titled **"Subprojects"** with one card row per direct child project.

Each row shows (left to right):

- **Logo thumbnail** — if the child has any logo uploaded, show the first one as a
  `28×28px` thumbnail (`object-fit: contain`, `background: var(--bg-input)`).
  If no logo, show nothing in that slot.
- **Project name** — rendered in the child's primary font (`typography[0].family`
  if set, else `var(--font-sans)`) at `13px`, `font-weight: 500`.
- **Color swatches** — up to 5 small `16×16px` swatches showing the child's brand
  colors in order (`border-radius: 4px`, `border: 0.5px solid var(--border)`).
  If no colors are set, show a single muted placeholder swatch using `var(--bg-input)`.
- **"Edit brand →" button** — right-aligned, navigates to that child's brand kit.

### Row layout

```
[Logo?] [Project Name ................] [● ● ● ● ●] [Edit brand →]
```

### Interaction

- **Clicking "Edit brand →"**: set active project to the child, set active tab to
  `'branding'`, scroll sidebar to show the child.
- **Clicking anywhere else on the row**: select the child project (secrets tab),
  same as clicking it in the sidebar.

### When a project has both children AND manual sub-brand entries

Manual `branding.subBrands[]` data is preserved but hidden from the UI when the
project has children. If all children are later removed or re-parented, the manual
sub-brands section reappears automatically. Never delete `subBrands[]` data.

---

## CSS — New Classes

Add these to the existing `<style>` block. Do not introduce new CSS variables.

```css
/* Sidebar subproject indicator */
.subproject-indent { padding-left: 16px; }
.subproject-arrow {
  font-size: 10px;
  color: var(--text-muted);
  margin-right: 4px;
  flex-shrink: 0;
}

/* Sidebar collapse chevron */
.project-chevron {
  font-size: 10px;
  color: var(--text-muted);
  margin-right: 4px;
  flex-shrink: 0;
  transition: transform 150ms ease;
  cursor: pointer;
}
.project-chevron.expanded { transform: rotate(90deg); }

/* Subproject summary rows in brand kit */
.subproject-summary-row {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 12px;
  border-radius: var(--radius);
  background: var(--bg-card);
  border: 0.5px solid var(--border);
  cursor: pointer;
  transition: border-color var(--transition);
  margin-bottom: 6px;
}
.subproject-summary-row:hover { border-color: var(--border-focus); }
.subproject-logo-thumb {
  width: 28px;
  height: 28px;
  border-radius: 4px;
  object-fit: contain;
  background: var(--bg-input);
  flex-shrink: 0;
}
.subproject-name {
  font-size: 13px;
  font-weight: 500;
  color: var(--text);
  flex: 1;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.subproject-swatches {
  display: flex;
  gap: 4px;
  flex-shrink: 0;
}
.subproject-swatch {
  width: 16px;
  height: 16px;
  border-radius: 4px;
  border: 0.5px solid var(--border);
  flex-shrink: 0;
}
.subproject-edit-btn {
  font-size: 11px;
  color: var(--accent);
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 4px;
  white-space: nowrap;
  flex-shrink: 0;
}
.subproject-edit-btn:hover { background: var(--bg-card-hover); }
```

---

## Search Result Path Labels

When a search result belongs to a subproject, show the full hierarchy path in the
project label: **"Realmdrop Software › CaretCart"** instead of just "CaretCart".

```js
function getProjectPath(project, allProjects) {
  const parts = [project.name];
  let current = project;
  while (current.parentId) {
    const parent = allProjects.find(p => p.id === current.parentId);
    if (!parent) break;
    parts.unshift(parent.name);
    current = parent;
  }
  return parts.join(' › ');
}
```

Use `getProjectPath()` wherever the project name is shown in search results.
In non-search mode, show the project name alone (no path) — the sidebar hierarchy
already makes context clear.

---

## Export for Claude — Subproject Handling

**On a parent project:** include a "Subprojects" section in the markdown:

```markdown
## Subprojects
| Name | Primary Color | Accent | Font |
|------|--------------|--------|------|
| CaretCart | #FF9B21 | #FFF4E8 | Inter |
| AuthRealm | #7C3AED | #F0E8FF | Syne |
```

Pull `primary` color from `colors.find(c => c.role === 'primary')?.hex`,
`accent` from `colors.find(c => c.role === 'accent')?.hex`,
font from `typography[0]?.family`. Use `—` for any missing values.

**On a child project:** add one line to the Identity section:

```markdown
- **Parent project:** Realmdrop Software
```

---

## Keyboard Shortcuts

Add to the existing keyboard shortcut system and update the `?` overlay:

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl+[` | Navigate to parent project (no-op if already top-level) |
| `Cmd/Ctrl+Shift+P` | Open new project modal with parent picker focused |

Add under a new **"Navigation"** section heading in the shortcuts overlay.

---

## Implementation Order

Work through these in order. Test after each step before moving on.

1. **Data model + migration** — add `parentId: null` to all projects on load
2. **`buildTree()` utility** — flat array → tree, used by sidebar render
3. **`getProjectPath()` utility** — used by search results and export
4. **Sidebar tree rendering** — recursive render with `subproject-indent` class
5. **Collapse/expand chevron** — `collapsedProjects` Set, chevron toggle
6. **Sidebar `↳` indicator** — prefix on subproject items
7. **Sidebar "+ Add subproject" hover button** — opens modal pre-filled
8. **New project modal parent picker** — dropdown, indented options, cycle guard
9. **Edit project modal parent picker** — re-parent or promote existing projects
10. **Delete with subprojects dialog** — "Promote & Delete" or "Cancel"
11. **Brand kit subproject summary panel** — replaces sub-brands when children exist
12. **Brand kit edit button** — navigate to child brand kit on click
13. **Export for Claude updates** — parent/child lines in markdown
14. **Search result path labels** — `getProjectPath()` in search result project tag
15. **Keyboard shortcuts** — `Cmd+[` and `Cmd+Shift+P`, update overlay

---

## Invariants — Never Violate These

- The flat `vault.projects[]` array stays flat. Tree structure is computed at
  render time only, never persisted.
- A project's `parentId` must either be `null` or the `id` of an existing project.
  Orphaned `parentId` values (pointing to deleted projects) should be cleaned up
  to `null` during any vault load migration.
- A project can never be its own ancestor. The parent picker must enforce this.
- All existing features (secrets, brand kit, links, sharing, .env export, keyboard
  shortcuts) work identically on subprojects — no special casing required.
- Subprojects appear in the export modal and can be exported independently.
- Deleting a project never silently deletes its subprojects. Always ask first.
