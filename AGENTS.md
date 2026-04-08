# AGENTS.md — datacube-explorer (DE Indonesia fork)

Fork of [datacube-explorer](https://github.com/opendatacube/datacube-explorer) customised for the DE Indonesia project. Focus is UI theming and visual customisation, not upstream features.

## Local dev environment (Docker)

The only practical way to run locally. Brings up Explorer + PostGIS together:

```bash
make up            # start app + PostGIS container (http://localhost:5000)
make init-odc      # init ODC database
make schema        # init Explorer schema
make index         # generate product summaries
```

The Docker override (`docker-compose.override.yml`) mounts `./` at `/code` and runs Flask in debug mode with auto-reload, so template and static file changes appear immediately.

PostgreSQL runs on port **35433** (not the default 5432).

To stop: `docker compose down`

## UI theming system

Explorer uses [flask-themer](https://github.com/TkTech/flask_themer). The active theme is set via `CUBEDASH_THEME` config (defaults to `"odc"`).

### Theme structure

A theme is a directory under `cubedash/themes/<name>/`:

```
cubedash/themes/<name>/
  info.json       — name, description, default map zoom/center
  logo.html       — Jinja2 fragment rendered in the header
  static/         — theme-specific static files (favicon, logo images)
```

Existing themes to reference as examples: `odc/`, `dea/`, `deafrica/`.

### To activate a theme

Set in `.docker/settings_docker.py` (used by Docker Compose override):

```python
CUBEDASH_THEME = "deindonesia"
```

Or via the `CUBEDASH_SETTINGS` env var pointing to a `.env.py` file.

### Key UI files to modify

| What | Where |
|---|---|
| Page layouts | `cubedash/templates/layout/base.html` — master layout (header, nav, footer) |
| All page templates | `cubedash/templates/*.html` — product, dataset, search, etc. |
| Main stylesheet (Sass) | `cubedash/static/base.sass` — all colors, layout, spacing |
| Compiled CSS | `cubedash/static/base.css` (generated, do not edit directly) |
| Map/overview JS | `cubedash/static/overview.ts` → compiles to `overview.js` |
| Logo in header | Theme's `logo.html` (included via `{% include theme('logo.html') %}`) |
| Favicon | Theme's `static/favicon.ico` (referenced via `{{ theme_static('favicon.ico') }}`) |
| Footer text | `cubedash/templates/include-footer.env.html` (gitignored, create locally) |
| Global includes | `cubedash/templates/include-global.env.html` (gitignored, for analytics etc.) |

### Sass color variables

The entire color scheme is controlled by Sass variables at the top of `base.sass`:

```sass
$header_base: lighten(#082e41, 5%)     // header background
$breadcrumb_base: #00718b              // breadcrumb bar
$panel_color: #e6edef                  // content panels
$bold_highlight: #212121               // primary text
```

Changing these variables recolors the whole UI. No other file should need color changes.

### Static asset compilation

Sass and TypeScript must be compiled after changes:

```bash
make static        # compile both Sass + TypeScript
make style         # Sass only (base.sass → base.css)
make js            # TypeScript only (overview.ts → overview.js)
```

Requires `npx sass` and `tsc` (installed via `npm install` in project root — see `Makefile`).

**Important**: The Docker dev mount means compiled CSS/JS in `cubedash/static/` is served directly from your local filesystem. You must run `make style` or `make js` after editing `.sass` or `.ts` files to see changes.

## Python commands

```bash
# Install (uses uv)
uv sync --locked --extra=test

# Format
ruff format cubedash integration_tests ./*.py

# Lint
ruff check .
pre-commit run -a

# Typecheck
mypy --follow-imports=silent cubedash/*.py cubedash/index/*.py cubedash/summary cubedash/testutils

# Run all tests (needs Docker for PostGIS)
pytest integration_tests/

# Run a single test file
pytest integration_tests/test_page_loads.py
```

## Test architecture

- All meaningful tests are integration tests in `integration_tests/` — they require Docker+PostGIS.
- Test harness auto-starts a PostGIS container unless inside Docker or `CUBEDASH_BYPASS_DOCKER` is set.
- Tests are parametrized across both `postgres` and `postgis` ODC index drivers.
- To load test data, declare module globals:
  ```python
  pytestmark = pytest.mark.usefixtures("auto_odc_db")
  METADATA_TYPES = ["metadata/qga_eo.yaml"]
  PRODUCTS = ["products/ga_s2_ard.odc-product.yaml"]
  DATASETS = ["s2a_ard_granule.yaml.gz"]
  ```

## Conventions

- **Formatter**: ruff (not black). Max enforced line length 120.
- **Import sorting**: enforced by ruff (`I` rules).
- **Click dependency**: patched fork (`pjonsson/click@one-close-only`) — do not upgrade.
- **Default timezone**: `Australia/Darwin` — change via `CUBEDASH_DEFAULT_TIMEZONE` config.
- **Versioning**: `setuptools_scm` from git tags → `cubedash/_version.py`.
