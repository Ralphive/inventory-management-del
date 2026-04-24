# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Factory Inventory Management System — a full-stack demo app (Claude Code workshop) for inventory tracking, order management, demand forecasting, and spending analytics. Uses in-memory JSON data (no database); changes don't persist across server restarts.

## Commands

### Start/Stop Servers
```bash
./scripts/start.sh          # Starts both backend (8001) and frontend (3000)
./scripts/stop.sh            # Stops both servers

# Manual (separate terminals):
cd server && uv run python main.py    # Backend at http://localhost:8001
cd client && npm run dev              # Frontend at http://localhost:3000
```

### Tests
```bash
cd tests
uv run pytest -v                                    # All tests (55 tests, ~0.13s)
uv run pytest backend/test_inventory.py -v          # Single file
uv run pytest backend/test_inventory.py::TestInventoryEndpoints::test_get_all_inventory  # Single test
uv run pytest --cov=../server --cov-report=html     # With coverage
```

### Build
```bash
cd client && npm run build    # Production build → client/dist/
```

### Install Dependencies
```bash
cd server && uv venv && uv sync    # Python (uses uv package manager)
cd client && npm install            # Node
```

## Architecture

```
client/   → Vue 3 + Vite (Composition API, Vue Router, Axios)
server/   → Python FastAPI (Pydantic models, Uvicorn, in-memory JSON data)
tests/    → pytest with FastAPI TestClient
scripts/  → Shell scripts for start/stop automation
```

### Frontend → Backend Communication
- Frontend calls `client/src/api.js` methods which use Axios to hit `http://localhost:8001/api/*`
- All filter endpoints accept query params: `warehouse`, `category`, `status`, `month`
- Filter value `'all'` means "no filter" (skip that parameter)
- Category matching is case-insensitive on the backend

### Key Backend Patterns
- All data loaded from `server/data/*.json` at startup into memory via `mock_data.py`
- `apply_filters()` and `filter_by_month()` in `main.py` handle shared filtering logic
- Pydantic models define response types; FastAPI auto-generates docs at `/docs`

### Key Frontend Patterns
- Views in `client/src/views/` are page-level components (Dashboard, Inventory, Orders, Spending, Demand, Reports)
- Shared state via composables in `client/src/composables/` (useAuth, useFilters, useI18n)
- i18n supports English and Japanese (`client/src/locales/`)
- Currency formatting helpers in `client/src/utils/currency.js`

## Sub-Project Guidance

See `client/CLAUDE.md` for Vue 3 patterns and `server/CLAUDE.md` for FastAPI patterns. The `tests/README.md` documents test structure and conventions.

## API Endpoints

Interactive docs available at http://localhost:8001/docs when server is running.

Key endpoints: `/api/inventory`, `/api/orders`, `/api/dashboard/summary`, `/api/demand`, `/api/backlog`, `/api/spending/*`, `/api/reports/*`, `/api/purchase-orders`

## Testing Conventions

- Test files live in `tests/backend/` and follow `test_*.py` naming
- `conftest.py` provides a `client` fixture (FastAPI TestClient)
- Tests validate: status codes, response structure, filter behavior, calculation accuracy, 404 handling
- All tests use in-memory data — no external dependencies needed
