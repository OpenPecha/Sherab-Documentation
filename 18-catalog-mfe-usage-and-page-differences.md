# 18. Catalog MFE — Local & Server Setup

**Repository:** https://github.com/OpenPecha/frontend-app-catalog  
**Branches:** `wbc-ulmo1-stage` (local + staging) · `wbc-ulmo1-prod` (production)

Catalog replaces legacy LMS Home, Course Catalog, and Course About pages.

---

## Prerequisites

| Requirement | Value |
|-------------|-------|
| Tutor | **21.0.1** (Ulmo) |
| Node.js | **24** (repo `.nvmrc`) |
| `/etc/hosts` (local only) | `local.openedx.io`, `apps.local.openedx.io`, `studio.local.openedx.io` → `127.0.0.1` |

> Use the version of node in the `.nvmrc` file.

### Required plugins (enable on local and server)

| Plugin | Purpose |
|--------|---------|
| `forked-mfe.py` | Catalog MFE registry (port 1998) |
| `catalog_mfe.py` | Enable Catalog, LMS URLs, `edx-search==4.4.0` (Ulmo.1 workaround) |
| `forked-header-footer.py` | Sherab header/footer in Catalog build |

> Catalog LMS settings live in `catalog_mfe.py` only — not `configuration_plugin.yml`. Disable this plugin when a future Tutor release ships native Catalog config.

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
$TUTOR plugins list   # expect: mfe, forked-mfe, catalog_mfe, forked-header-footer
```

---

## Local setup (`tutor dev`)

### 1. Clone and install

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
cd "$($TUTOR config printroot)"

git clone https://github.com/OpenPecha/frontend-app-catalog.git
cd frontend-app-catalog
git checkout wbc-ulmo1-stage
git pull origin wbc-ulmo1-stage

nvm use && npm ci

# Sherab header/footer (once after clone or npm ci)
npm install --no-save \
  '@edx/frontend-component-footer@git+https://github.com/OpenPecha/frontend-component-footer.git#wbc-ulmo1-stage' \
  '@edx/frontend-component-header@git+https://github.com/OpenPecha/frontend-component-header.git#wbc-stage-v8.0.0' \
  '@edx/brand@npm:@openedx/brand-openedx@latest' \
  sharp #Image processing library; required by some MFE webpack/build steps on macOS
```

### 2. Mount and configure

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
$TUTOR mounts add "$($TUTOR config printroot)/frontend-app-catalog"
$TUTOR mounts list
```

Expected mount target: `catalog` service at `/openedx/app`.

### 2.1 Confirm Tutor plugin configuration

These are already set in Sherab. Verify they exist — do not hand-edit generated env files.

**`forked-mfe.py`** — Catalog registry:

```python
mfes["catalog"] = {
    "repository": "https://github.com/OpenPecha/frontend-app-catalog.git",
    "port": 1998,
    "version": "wbc-ulmo1-stage",
}
```

**`catalog_mfe.py`** — LMS settings and edx-search patch (enable flags, dev/prod URLs):

```python
from tutor import hooks

# Temporary Ulmo.1 Catalog MFE settings.
# Disable/remove this plugin when Tutor ships native Catalog config
# to avoid duplicate LMS settings.

CATALOG_ENABLE = """
FEATURES['ENABLE_CATALOG_MICROFRONTEND'] = True
ENABLE_CATALOG_MICROFRONTEND = True
"""

CATALOG_URLS_DEV = """
CATALOG_MICROFRONTEND_URL = "http://apps.local.openedx.io:1998/catalog"
MFE_CONFIG["CATALOG_MFE_URL"] = CATALOG_MICROFRONTEND_URL
MFE_CONFIG["CATALOG_MICROFRONTEND_URL"] = CATALOG_MICROFRONTEND_URL
"""

CATALOG_URLS_PROD = """
CATALOG_MICROFRONTEND_URL = "http://apps.local.openedx.io/catalog"
MFE_CONFIG["CATALOG_MFE_URL"] = CATALOG_MICROFRONTEND_URL
MFE_CONFIG["CATALOG_MICROFRONTEND_URL"] = CATALOG_MICROFRONTEND_URL
"""

INSTALL_SEARCH_440 = r"""
RUN --mount=type=cache,target=/openedx/.cache/pip,sharing=shared \
    pip install "edx-search==4.4.0"
"""

hooks.Filters.ENV_PATCHES.add_items([
    ("openedx-lms-common-settings", CATALOG_ENABLE),
    ("openedx-lms-development-settings", CATALOG_URLS_DEV),
    ("openedx-lms-production-settings", CATALOG_URLS_PROD),
    ("openedx-dockerfile-post-python-requirements", INSTALL_SEARCH_440),
    ("openedx-dev-dockerfile-post-python-requirements", INSTALL_SEARCH_440),
])
```

> The `INSTALL_SEARCH_440` block patches the openedx Dockerfile to force `edx-search==4.4.0` (required Ulmo.1 workaround — without it, Catalog course search returns 404).

Regenerate config after any plugin change:

```bash
$TUTOR config save
```

Verify `forked-mfe.py` has Catalog on port **1998** / branch `wbc-ulmo1-stage`, and `catalog_mfe.py` exists in `tutor-plugins/`.

### 3. Build and start

```bash
$TUTOR images build catalog-dev
$TUTOR images build openedx-dev    # required for edx-search 4.4.0; can take 45–90+ min

# After openedx-dev rebuild, force-recreate LMS (`tutor dev restart` alone reuses 
the old container and will not pick up `edx-search==4.4.0`):

docker compose \
  -f "$($TUTOR config printroot)/env/local/docker-compose.yml" \
  -f "$($TUTOR config printroot)/env/dev/docker-compose.yml" \
  --project-name tutor_dev up -d --force-recreate lms cms
```

### 3.1 Start local environment

```bash
$TUTOR dev start -d
$TUTOR dev status
```

Catalog service: `tutor_dev-catalog-1` on port **1998**.

- http://apps.local.openedx.io:1998/catalog/
- http://local.openedx.io:8000/ → redirects to Catalog
- `docker exec tutor_dev-lms-1 pip show edx-search` → **4.4.0**

---

## Server setup (staging & production)

Servers use **`tutor local`**, not `tutor dev`.

| Environment | Branch | Command |
|-------------|--------|---------|
| Staging | `wbc-ulmo1-stage` | `tutor local` |
| Production | `wbc-ulmo1-prod` | `tutor local` |

### Before deploy

1. Ensure `forked-mfe.py`, `catalog_mfe.py`, and `forked-header-footer.py` are on the server and enabled.
2. Update `CATALOG_URLS_PROD` in `catalog_mfe.py` to your real MFE URL (e.g. `https://apps.your-domain.org/catalog`).
3. Set `forked-mfe.py` version to `wbc-ulmo1-stage` (staging) or `wbc-ulmo1-prod` (production).

### Deploy

```bash
tutor config save
tutor images build catalog-prod
tutor images build openedx
tutor local stop && tutor local start -d
tutor local restart lms cms
```

If course search fails after restart, force-recreate LMS/CMS (same as local).

### Verify on server

- LMS root redirects to Catalog home
- Catalog loads course data (not error box)
- `tutor local run lms pip show edx-search` → **4.4.0**

---

## Key files

| File | Path |
|------|------|
| Catalog clone | `$(tutor config printroot)/frontend-app-catalog` |
| MFE registry | `~/Library/Application Support/tutor-plugins/forked-mfe.py` |
| Catalog plugin | `~/Library/Application Support/tutor-plugins/catalog_mfe.py` |
| Header/footer | `~/Library/Application Support/tutor-plugins/forked-header-footer.py` |

**Related:** [14 — Header, Footer & Brand Overrides](14-header-footer-brand-override.md) · [16 — Deployment Workflow](16-development-testing-deployment-workflow.md)
