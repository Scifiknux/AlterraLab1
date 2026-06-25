---
project_name: 'Lab1'
user_name: 'Scifiknux'
date: '2026-06-25'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'quality_rules', 'workflow_rules', 'anti_patterns']
status: 'complete'
rule_count: 47
optimized_for_llm: true
---

# Project Context for AI Agents

_Critical rules and patterns for consistent, high-quality code generation across all AI agents working on this project. Focused on unobvious details agents would otherwise miss._

---

## Technology Stack & Versions

| Layer | Technology | Version |
|---|---|---|
| Frontend framework | Astro | 6.1.6 (SSR, standalone Node adapter) |
| Language (client) | TypeScript | 6.0.2, strict mode |
| CSS | Tailwind CSS | 4.2.2 (Vite plugin — NOT v3) |
| Node adapter | @astrojs/node | 10.0.4, `mode: 'standalone'` |
| E2E testing | Playwright | 1.59.1 |
| Backend framework | Python Flask | unversioned in requirements.txt |
| ORM | SQLAlchemy + Flask-SQLAlchemy | unversioned |
| Database | SQLite | file: `dogshelter.db` |
| Module system | ESM | `"type": "module"` throughout client |

**Dev ports:** Flask → 5100, Astro dev → 4321

---

## Critical Implementation Rules

### Language-Specific Rules

**TypeScript (client)**
- `tsconfig.json` extends `astro/tsconfigs/strict` — no loosening of `noImplicitAny`, `strictNullChecks`, or other strict flags
- `"type": "module"` in `package.json` — ESM only; no `require()`
- Node built-ins use the `node:` prefix (`node:fs`, `node:path`, etc.)
- Astro frontmatter (`---` fences) is server-side TypeScript — interfaces and data fetching belong here
- Props interfaces declared inline in component frontmatter, not in separate `.ts` files
- Error narrowing: `err instanceof Error ? err.message : String(err)`

**Python (server)**
- Type hints required on all functions — parameters, return types, and non-obvious local variable declarations
- Union return types use `|` syntax (`tuple[Response, int] | Response`), requiring Python 3.10+
- Imports are absolute from `server/` root (e.g., `from models import init_db, db, Dog, Breed`)

### Framework-Specific Rules

**Astro**
- Output mode is `server` (SSR) — all pages render server-side; use `export const prerender = true` only when explicitly needed
- Data fetching in frontmatter only — never client-side `fetch` for initial page data
- Dynamic routes: `src/pages/dog/[id].astro` — param via `Astro.params.id`
- Query params via `Astro.url.searchParams.get('key')`, not framework router hooks
- Component hierarchy: `layouts/Layout.astro` → `pages/*.astro` → `components/*.astro`
- No React/Vue/Svelte islands — pure `.astro` components only
- `API_SERVER_URL` env var required at dev/build time; fallback is `http://localhost:5100`

**Flask**
- All routes prefixed `/api/` — no bare routes
- DB init via `init_db(app)` wrapper (not `db.init_app(app)` directly)
- Server runs on port 5100 (`debug=True`) — avoids macOS AirPlay conflict on 5000
- Query style: `db.session.query(...).join().offset().limit().all()` — do NOT use the newer `db.select()` style
- 404 responses: `return jsonify({"error": "..."}), 404` — tuple with integer status
- Pagination response must include all five keys: `dogs`, `page`, `per_page`, `total`, `total_pages`
- New routes go above `if __name__ == '__main__':` at the `## HERE` marker in `app.py`

**SQLAlchemy Models**
- All models inherit from `BaseModel` (`models/base.py`), not directly from `db.Model`
- String validation via `@validates` + `BaseModel.validate_string_length()` — follow this pattern for new fields
- Enum serialization: use `.name` (→ `'AVAILABLE'`) not `.value` (→ `'Available'`) for API responses
- Relationships defined on models but joined explicitly in queries; do not rely on lazy-loaded relationship attributes in API routes

### Testing Rules

**E2E (Playwright)**
- Test files in `client/e2e-tests/` with `.spec.ts` naming
- All tests grouped with `test.describe('...', () => { ... })`
- Primary selection strategy: `page.getByTestId()` — `data-testid` is the test/markup contract
- Every UI element a test interacts with or asserts on must have a `data-testid`
- Playwright config auto-starts both servers (Flask :5100, Astro :4321) — tests never assume pre-running servers
- E2E uses a separate `e2e_test_dogshelter.db`; never use production `dogshelter.db`
- `API_SERVER_URL` injected via `cross-env` in webServer config — never hardcode ports in test files
- Only Chromium configured; do not add browser projects without explicit instruction
- CI: `forbidOnly: true`, `retries: 2`, `workers: 1`

**Unit (Python)**
- File: `server/test_app.py` using `unittest.TestCase`; run with `python -m unittest`
- Setup: `app.test_client()` + `app.config['TESTING'] = True`
- Patch DB at app module level: `@patch('app.db.session.query')` — not at SQLAlchemy source
- Reuse `_create_mock_dog()` and `_setup_query_mock()` helpers for new tests
- Always use `MagicMock(spec=[...])` with constrained spec to avoid false passes
- No real database in unit tests — all DB interaction mocked

### Code Quality & Style Rules

**TypeScript / Astro**
- No ESLint/Prettier config — TypeScript strict mode is the primary quality gate
- No comments by default — self-documenting naming is expected
- Props interfaces declared inline in `.astro` frontmatter, not in separate type files
- Tailwind classes applied directly on elements — no CSS modules, no `@apply`, no separate stylesheets
- Color palette: `slate-100/300/400/700/800` for text/backgrounds, `blue-400/500` for interactive accents
- Interactive cards use Tailwind `group` / `group-hover:` utilities — follow this pattern for new interactive elements

**Python**
- No linter config found — follow PEP 8
- Type hints on every function: parameters, return types, and non-obvious local variables
- `__repr__` on all models: `<ClassName field, ID: id, Status: status>` format
- `to_dict()` exists on models but API routes build dicts manually — pick one approach per feature and stay consistent

**File & Folder Structure**
- Client: `src/components/` (reusable), `src/pages/` (routes), `src/layouts/` (wrappers)
- Server: `models/` (SQLAlchemy), `utils/` (scripts), flat root for `app.py` and tests
- No barrel files (`index.ts`) — import directly from source files

### Development Workflow Rules

**Running the App**
- Launch both servers via `app/scripts/start-app.sh` (or `.ps1` on Windows)
- Seed database first with `app/scripts/seed-database.sh`
- E2E tests: `npm run test:e2e` from `client/` — Playwright starts both servers automatically

**Azure Deployment**
- Target: Azure Container Apps via `azd up` (provision + deploy in one step)
- `DATABASE_PATH` env var sets SQLite path in production — never hardcode
- `azd infra gen` only if Bicep files need inspection/modification; once on disk they become source of truth
- Both client and server have `Dockerfile`s for containerized builds

**Git**
- Commit style: short imperative subject lines
- No branch naming conventions or CI workflows defined yet

### Critical Don't-Miss Rules

**Anti-Patterns**
- Never use `status.value` in API responses — use `status.name` (`'AVAILABLE'` not `'Available'`); mixing these breaks client comparisons silently
- Never fetch data in Astro `<script>` blocks for initial render — all data fetching belongs in frontmatter (server-side)
- Never add a `tailwind.config.js` — Tailwind v4 uses Vite plugin only; a config file breaks the build
- Never patch `sqlalchemy.orm.Session.query` in tests — patch `app.db.session.query` (module level)
- Never add Flask routes outside the `/api/` prefix
- Never use `dogshelter.db` in tests — unit tests and E2E both use separate test databases

**Edge Cases**
- Paginated endpoints must clamp: `page = max(1, page)`, `per_page = max(1, min(per_page, 100))`
- `total_pages` must never be zero: use `max(1, -(-total // per_page))` ceiling-division
- `Dog.description` is nullable — do not add `nullable=False`; `validate_description` already handles `allow_none=True`

**Security**
- `debug=True` in `app.run()` — must be env-gated before production
- `DATABASE_PATH` env var controls DB path — never construct DB paths from user input
- No auth layer exists — all new endpoints are publicly accessible by default

---

## Usage Guidelines

**For AI Agents:**
- Read this file before implementing any code in this project
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Flag conflicts between these rules and a story's requirements rather than silently overriding

**For Humans:**
- Keep this file lean and focused on agent needs — no obvious rules
- Update when the technology stack or conventions change
- Review periodically and remove rules that have become obvious over time

_Last Updated: 2026-06-25_
