# AmethystAnalytics — Agent Rules

> Repo-specific rules for Claude Code and Gemini CLI. Read `../CLAUDE.md` and `../GEMINI.md` first — this file extends those, it does not replace them.

---

## Purpose

AmethystAnalytics is the **Streamlit dashboard**. It is a thin UI layer — all data comes from AmethystServer's REST API via the auto-generated client. It has no direct database or Redis access.

---

## Stack

| Item | Detail |
|---|---|
| Language | Python 3.11.x |
| Package manager | Poetry 1.7.x |
| Framework | Streamlit |
| API client | `amethyst_client` — auto-generated from AmethystServer's `openapi.json` |
| Port | 8501 |
| Shared lib | `amethyst_core` (for config and logging only) |

---

## Critical Rule: Never Edit the Generated Client

The `amethyst_client` package under `src/amethyst_client/` is **auto-generated** from AmethystServer's `openapi.json`. Never edit it manually. If the API contract changes:
1. Regenerate `openapi.json` from AmethystServer.
2. Re-run `AmethystInfrastructure/scripts/generate_client.sh`.
3. Commit the regenerated client.

---

## What Belongs Here

- Streamlit page components and layout
- UI state management (Streamlit session state only)
- Data formatting and display logic
- Chart rendering (use `plotly` or `altair`)

---

## What Does NOT Belong Here

- Any business logic — zero
- Any direct database queries (asyncpg, SQLAlchemy, etc.)
- Any Redis connections
- Any Upstox API calls
- Any authentication logic beyond passing the JWT from session state

---

## Naming Conventions

- Page files: `pages/<number>_<name>.py` — e.g., `pages/01_live_feed.py`
- Component functions: noun phrases — `render_tick_chart()`, `render_instrument_selector()`
- Session state keys: `snake_case` strings — `session_state["auth_token"]`

---

## Forbidden Patterns

- No direct import from `market_monitor` or `amethyst_server` source packages
- No `requests` or `httpx` raw HTTP calls — use `amethyst_client` only
- No hardcoded API URLs — read from `AmethystBaseConfig`
- No two-way data binding hacks — Streamlit's reruns handle state
- No blocking sleep loops — use Streamlit's native refresh or `st.rerun()`

---

## Testing

- UI component functions that contain logic must have unit tests.
- Use `streamlit.testing.v1.AppTest` for integration tests.
- Mock all `amethyst_client` calls in tests.
- Coverage minimum: 80% on non-Streamlit logic (CI hard fails below this).

---

## Service Boundaries

| Direction | Protocol | Notes |
|---|---|---|
| AmethystAnalytics → AmethystServer | REST (port 8080) | All data; polls every 5 seconds |
| AmethystAnalytics → AmethystServer | SSE | Live tick stream |

AmethystAnalytics never connects to the database, Redis, or MarketMonitor directly.
