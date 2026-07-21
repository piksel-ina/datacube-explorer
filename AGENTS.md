# AGENTS.md — datacube-explorer (DE Indonesia fork)

Fork of [datacube-explorer](https://github.com/opendatacube/datacube-explorer) customised for the DE Indonesia project (BIG — Badan Informasi Geospasial). Focus is UI theming and visual customisation, not upstream features.

## Architecture: thin-wrapper deployment

The `deindonesia` theme is deployed as a **thin Docker wrapper** over the upstream image. The production image (`Dockerfile.deindonesia`) extends `ghcr.io/opendatacube/explorer:3.1.5-43-g8aaa2dba` and:
- Copies in compiled CSS (`base.css` + source map) and the `cubedash/themes/deindonesia/` directory
- Applies a small number of **local Python patches** via `sed` (see the "Local patches to upstream" table in `README.md`)

Patches exist to work around specific upstream bugs; each should be removed when the corresponding upstream fix is released. The `sed` commands include a `grep -q` guard, so if a patch anchor stops matching (upstream drift or upstream fix landed) the build fails loudly.

**CI**: `.github/workflows/build-deindonesia.yml` — manual dispatch, builds `Dockerfile.deindonesia`, pushes to AWS ECR (`dc-explorer` repo). Tags are `vYYYYMMDD-HHMM` in Asia/Jakarta timezone.

**Important**: When updating the pinned upstream base image, update both `Dockerfile.deindonesia` and `docker-compose.yml`.

## Local dev

```bash
make up && make init-odc && make schema && make index   # full setup
make style                                               # compile Sass
make static                                              # compile Sass + TS
```

App at http://localhost:5000. PostgreSQL on port 35433. Dev settings in `.docker/settings_docker.py` (`CUBEDASH_THEME = "deindonesia"`, `CUBEDASH_DEFAULT_TIMEZONE = "Asia/Jakarta"`).

## Styling

**Color tokens** (single source of truth): `cubedash/static/_deindonesia-tokens.sass` — named palette (navy, accent, cream, surfaces, footer).

**Role mapping**: top of `cubedash/static/base.sass` imports tokens via `@use 'deindonesia-tokens' as *` and maps to UI roles (`$header-bg`, `$breadcrumb-bg`, `$link-color`, etc.). Edit tokens for palette changes; edit role mapping only if restructuring the UI.

**After any `.sass` edit**: run `make style`. Commit `base.css` — the production Dockerfile copies it directly.

## Theme

`cubedash/themes/deindonesia/` — `info.json` (BIG branding, map center over Indonesia), `logo.html`, `static/` (favicons, logo images). Reference `odc/`, `dea/`, `deafrica/` for structure.

## Conventions

- ruff formatter, max line 120
- Click: patched fork `pjonsson/click@one-close-only` — do not upgrade
- Base image: `ghcr.io/opendatacube/explorer:3.1.5-43-g8aaa2dba`
