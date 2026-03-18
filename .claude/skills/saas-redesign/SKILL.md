---
name: saas-redesign
description: Redesigns a Vue 3 application's UI into a modern SaaS-style interface with a vertical navigation sidebar, consistent spacing, and a polished professional look.
---

# SaaS UI Redesign Skill

Transforms the current top-nav layout into a modern SaaS-style interface with a fixed vertical sidebar, integrated filters, and a polished content area. Delegate all `.vue` file changes to the **vue-expert** subagent.

## What This Skill Does

**Before:** Horizontal top nav bar + sticky filter bar below it + full-width content
**After:** Fixed dark sidebar (nav + filters + profile) + scrollable content area to the right

---

## Target Layout

```
┌──────────────────────────────────────────────────────────────┐
│  Sidebar (240px, fixed)   │  Content Area (flex: 1)          │
│  background: #0f172a      │  background: #f8fafc             │
│                           │                                  │
│  [Logo + Subtitle]        │  [Page Header]                   │
│                           │  [Stats / Tables / Charts]       │
│  Navigation               │                                  │
│  ○ Overview               │                                  │
│  ● Inventory  ← active    │                                  │
│  ○ Orders                 │                                  │
│  ○ Finance                │                                  │
│  ○ Demand Forecast        │                                  │
│  ○ Reports                │                                  │
│                           │                                  │
│  ─── Filters ───          │                                  │
│  Time Period  [select]     │                                  │
│  Location     [select]     │                                  │
│  Category     [select]     │                                  │
│  Status       [select]     │                                  │
│  [Reset]                  │                                  │
│                           │                                  │
│  ─── (spacer) ───         │                                  │
│  [Language] [Profile]     │                                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Implementation Steps

### Step 1 — Redesign `client/src/App.vue`

This is the primary structural change. The vue-expert must:

**Template changes:**
- Replace `<header class="top-nav">` with `<aside class="sidebar">`
- Move all `<router-link>` nav items into the sidebar as a `<nav>` block
- Move `<FilterBar />` inside the sidebar, below the nav
- Move `<LanguageSwitcher />` and `<ProfileMenu />` to the sidebar footer
- Remove the separate `<FilterBar />` from outside the sidebar
- Wrap everything in a new two-column layout: `<div class="app-layout">`

**Layout structure:**
```html
<div class="app">
  <div class="app-layout">
    <aside class="sidebar">
      <div class="sidebar-header">
        <!-- logo + subtitle -->
      </div>
      <nav class="sidebar-nav">
        <!-- router-links -->
      </nav>
      <div class="sidebar-filters">
        <FilterBar />
      </div>
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu />
      </div>
    </aside>
    <main class="main-content">
      <router-view />
    </main>
  </div>
  <!-- modals stay here, outside the layout -->
</div>
```

**CSS changes for `App.vue`:**

Remove `.top-nav`, `.nav-container`, `.nav-tabs` and their styles. Replace with:

```css
.app-layout {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: 240px;
  min-width: 240px;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  overflow-y: auto;
  z-index: 100;
  border-right: 1px solid #1e293b;
}

.sidebar-header {
  padding: 1.25rem 1rem 1rem;
  border-bottom: 1px solid #1e293b;
}

.sidebar-header h1 {
  font-size: 0.938rem;
  font-weight: 700;
  color: #f8fafc;
  letter-spacing: -0.02em;
}

.sidebar-header .subtitle {
  font-size: 0.688rem;
  color: #475569;
  margin-top: 2px;
  display: block;
  border-left: none;
  padding-left: 0;
}

.sidebar-nav {
  padding: 0.75rem 0.75rem 0;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-size: 0.813rem;
  font-weight: 500;
  color: #94a3b8;
  text-decoration: none;
  transition: all 0.15s ease;
}

.sidebar-nav a:hover {
  background: #1e293b;
  color: #e2e8f0;
}

.sidebar-nav a.active {
  background: #1e293b;
  color: #f8fafc;
  font-weight: 600;
}

/* Left accent bar on active nav item */
.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 2px;
  background: #3b82f6;
  border-radius: 0 2px 2px 0;
}

/* Each nav link needs position: relative for the accent bar */
.sidebar-nav a {
  position: relative;
}

.sidebar-filters {
  padding: 0.75rem;
  border-top: 1px solid #1e293b;
  margin-top: 0.75rem;
}

.sidebar-footer {
  margin-top: auto;
  padding: 0.75rem;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.main-content {
  flex: 1;
  margin-left: 240px;    /* offset for fixed sidebar */
  padding: 1.75rem 2rem;
  min-width: 0;          /* prevents flex overflow */
  background: #f8fafc;
}
```

### Step 2 — Redesign `client/src/components/FilterBar.vue`

The FilterBar moves from a horizontal strip to a vertical stack inside the sidebar.

**Template:** Change `.filters-grid` from a horizontal flex row to a vertical stack. Remove the outer `.filters-bar` wrapper (sidebar provides the container now). Keep `.filter-group`, labels, and selects.

**New FilterBar template structure:**
```html
<template>
  <div class="sidebar-filter-panel">
    <div class="filter-panel-header">
      <span class="filter-panel-title">Filters</span>
      <button class="reset-btn" @click="resetFilters" :disabled="!hasActiveFilters" title="Reset filters">
        <!-- reset SVG icon -->
      </button>
    </div>
    <div class="filter-stack">
      <div class="filter-group"> ... </div>
      <!-- repeat for each filter -->
    </div>
  </div>
</template>
```

**CSS for the dark-sidebar FilterBar:**
```css
.sidebar-filter-panel {
  /* no background needed — inherits sidebar dark bg */
}

.filter-panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.625rem;
}

.filter-panel-title {
  font-size: 0.688rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #475569;
}

.filter-stack {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.filter-group label {
  font-size: 0.688rem;
  font-weight: 500;
  color: #64748b;
}

.filter-select {
  width: 100%;
  padding: 0.375rem 0.5rem;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 5px;
  font-size: 0.75rem;
  color: #cbd5e1;
  cursor: pointer;
  transition: border-color 0.15s;
  appearance: none;
  -webkit-appearance: none;
}

.filter-select:hover {
  border-color: #475569;
}

.filter-select:focus {
  outline: none;
  border-color: #3b82f6;
}

/* Style the native select options (limited cross-browser) */
.filter-select option {
  background: #1e293b;
  color: #cbd5e1;
}

.reset-btn {
  background: none;
  border: none;
  padding: 2px;
  color: #475569;
  cursor: pointer;
  transition: color 0.15s;
  display: flex;
  align-items: center;
}

.reset-btn:hover:not(:disabled) {
  color: #94a3b8;
}

.reset-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.reset-btn svg {
  width: 14px;
  height: 14px;
}
```

**Important:** Remove the old `position: sticky; top: 70px` — that was specific to the top-nav layout.

### Step 3 — Update `client/src/components/ProfileMenu.vue`

The ProfileMenu likely opens a dropdown upward now (it's at the bottom of the sidebar). Ensure:
- The dropdown/popover opens **upward** (`bottom: 100%` instead of `top: 100%`)
- Width fits within the sidebar (max `200px`)
- Background uses `#1e293b`, text uses `#e2e8f0` to match sidebar theme

### Step 4 — Update `client/src/components/LanguageSwitcher.vue`

Fits into the sidebar footer. Should be compact. Use the same dark-theme styles (background `#1e293b`, text `#94a3b8`).

---

## Design Tokens (Dark Sidebar)

| Token | Value | Usage |
|-------|-------|-------|
| `--sidebar-bg` | `#0f172a` | Sidebar background |
| `--sidebar-border` | `#1e293b` | Dividers, item hover bg |
| `--sidebar-text` | `#94a3b8` | Inactive nav items |
| `--sidebar-text-active` | `#f8fafc` | Active nav item |
| `--sidebar-accent` | `#3b82f6` | Active indicator bar |
| `--content-bg` | `#f8fafc` | Main content area |

---

## Common Pitfalls to Avoid

1. **FilterBar `position: sticky; top: 70px`** — Must be removed. It was relative to the old top nav height.
2. **`max-width: 1600px` on `.main-content`** — Remove or adjust; the sidebar already constrains width.
3. **Modal `z-index`** — Modals should remain `z-index: 200+` so they appear above the sidebar (`z-index: 100`).
4. **`margin-left: 240px` on `.main-content`** — Required because sidebar is `position: fixed`.
5. **Scrolling** — The sidebar should scroll independently if content overflows; `overflow-y: auto` on `.sidebar`.
6. **Nav link active detection** — Keep `:class="{ active: $route.path === '/' }"` pattern; just update the CSS target from `.nav-tabs a.active` to `.sidebar-nav a.active`.

---

## Verification

After implementation:

1. **Start the dev server** — `cd client && npm run dev`
2. **Open** `http://localhost:3000` — sidebar should be visible on the left
3. **Check each route** — navigate to all 6 pages; active nav item should highlight correctly
4. **Check filters** — changing a filter should still trigger data reload in all views
5. **Check modals** — ProfileDetailsModal and TasksModal should appear above the sidebar
6. **Check scroll** — long pages should scroll in the content area; sidebar stays fixed
7. **Use Playwright MCP** (`mcp__playwright__*`) to take a screenshot and verify the layout visually
