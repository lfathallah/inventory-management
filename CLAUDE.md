# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System Demo - Full-stack application with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

## Critical Tool Usage Rules

### Subagents
- **vue-expert**: **MANDATORY** for creating or significantly modifying any `.vue` file
- **code-reviewer**: Use after writing significant code
- **Explore**: Use for understanding codebase structure or searching for patterns
- **general-purpose**: Use for complex multi-step tasks

### Skills
- **backend-api-test**: Use when writing or modifying tests in `tests/backend/`

### MCP Tools
- **ALWAYS use GitHub MCP tools** (`mcp__github__*`) for ALL GitHub operations
  - Exception: Local branches only — use `git checkout -b`
- **ALWAYS use Playwright MCP tools** (`mcp__playwright__*`) for browser testing
  - Test against: `http://localhost:3000` (frontend), `http://localhost:8001` (API)

## Stack
- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: JSON files in `server/data/` loaded into memory at startup via `server/mock_data.py`

## Commands

```bash
# One-command startup (macOS/Linux)
./scripts/start.sh
./scripts/stop.sh

# Backend
cd server && uv run python main.py

# Frontend
cd client && npm install && npm run dev

# Build frontend
cd client && npm run build

# Run all backend tests
cd tests && uv run pytest backend/ -v

# Run a single test file
cd tests && uv run pytest backend/test_inventory.py -v

# Run a single test by name
cd tests && uv run pytest backend/test_orders.py::test_filter_by_status -v

# Run tests with coverage
cd tests && uv run pytest backend/ --cov
```

## Architecture

### Data Flow
Vue filter state (`useFilters` composable) → `client/src/api.js` (builds query params) → FastAPI endpoint → in-memory filtering → Pydantic validation → computed properties in view

### Filter System
4 global filters (Time Period, Warehouse, Category, Order Status) managed in `composables/useFilters.js`. All views watch this composable and reload data when filters change. Filters pass as query params; the value `'all'` means no filter applied.

### Frontend Composables (`client/src/composables/`)
- **useFilters.js** — shared filter state (warehouse, category, status, month); imported by all views and FilterBar
- **useAuth.js** — authentication state and user profile
- **useI18n.js** — internationalization; loads from `locales/en.js` and `locales/ja.js` (English/Japanese)

### Reactivity Pattern
Raw API data stored in `ref()` (e.g., `allOrders`, `inventoryItems`); display-ready data in `computed()`. Never store derived data in refs.

### Charts
Custom SVG charts built inline in components — no chart library. Use `computed()` to transform data for chart coordinates.

### Currency Formatting
Centralized in `client/src/utils/currency.js`. Import from there instead of inline `toLocaleString` calls.

## API Endpoints
- `GET /api/inventory` — Filters: `warehouse`, `category`
- `GET /api/inventory/{item_id}`
- `GET /api/orders` — Filters: `warehouse`, `category`, `status`, `month` (supports `Q1-2025` format)
- `GET /api/orders/{order_id}`
- `GET /api/dashboard/summary` — All filters
- `GET /api/demand` — No filters
- `GET /api/backlog` — No filters
- `GET /api/spending/summary|monthly|categories|transactions`
- `GET /api/reports/quarterly`
- `GET /api/reports/monthly-trends`

## Tests
51 tests across 4 files in `tests/backend/`. Fixtures in `conftest.py` (FastAPI `TestClient`, sample item/order fixtures). Tests validate actual calculations, not hardcoded values.

## Common Issues
1. Use unique keys in `v-for` — use `sku`, `month`, etc., never `index`
2. Validate dates before `.getMonth()` calls
3. Update Pydantic models in `server/main.py` when changing JSON data structure
4. Inventory filters don't support `month` (no time dimension on inventory)
5. Revenue goals: $800K/month (single month), $9.6M YTD (all months)
6. Data changes don't persist — server restart reloads from JSON files

## Design System
- Colors: Slate/gray (`#0f172a`, `#64748b`, `#e2e8f0`)
- Status colors: green/blue/yellow/red
- Charts: Custom SVG, CSS Grid for layouts
- No emojis in UI
