# 18. Catalog MFE — Local & Server Usage Guide + Page Differences

**Environment:** Sherab / OpenPecha / Ulmo.1 / Tutor 21.0.1  
**Repository:** https://github.com/OpenPecha/frontend-app-catalog  
**Branches:** `wbc-ulmo1-stage` (local + staging) · `wbc-ulmo1-prod` (production)

### Summary of updates (this revision)

- **New consolidated plugin:** All Catalog LMS settings and the `edx-search==4.4.0` workaround live in `catalog_mfe.py` — removed from `configuration_plugin.yml` and `catalog_edx_search.py` (deleted) so a future Tutor release with native Catalog config can be enabled by disabling one plugin only.
- **§1.5 expanded:** Full `catalog_mfe.py` source documented (enable flags, dev/prod URLs, `INSTALL_SEARCH_440`, and `ENV_PATCHES` hooks).
- **Plugin table updated:** Lists `catalog_mfe.py` instead of `configuration_plugin.yml` / `catalog_edx_search.py` for Catalog; `configuration_plugin.yml` remains for general Sherab flags only.
- **§2.2 server deploy:** Staging/production Catalog URL changes go in `catalog_mfe.py` (`CATALOG_URLS_PROD`), with `https://apps.your-domain.org/catalog` as the server example (local prod-mode uses `http://apps.local.openedx.io/catalog` without port).
- **Dev vs prod URLs clarified:** `CATALOG_URLS_DEV` → `tutor dev` (`:1998`); `CATALOG_URLS_PROD` → `tutor local` (proxy URL, no port).
- **Part 4 quick reference:** Key files table points to `catalog_mfe.py` as the single Catalog + edx-search plugin.

---

## Before you start

### Prerequisites

| Requirement | Sherab value |
|-------------|--------------|
| Open edX release | Ulmo.1 |
| Tutor version | **21.0.1** (Ulmo) |
| Tutor root | `$(tutor config printroot)` |
| Node.js (Catalog MFE) | **24** (see repo `.nvmrc`) |

> Use the version of node in the `.nvmrc` file.

### Required Tutor plugins (already configured in Sherab)

These must be **enabled** on any environment (local or server):

| Plugin file | Purpose |
|-------------|---------|
| `forked-mfe.py` | Registers Catalog in Tutor MFE registry |
| `catalog_mfe.py` | Enables Catalog MFE, sets LMS URLs, forces `edx-search==4.4.0` (Ulmo.1 workaround) |
| `forked-header-footer.py` | Installs Sherab forked header/footer in Catalog build |

> Catalog LMS settings live in `catalog_mfe.py`, **not** in `configuration_plugin.yml`. Disable/remove `catalog_mfe.py` when a future Tutor release ships native Catalog config to avoid duplicate settings.

Verify:

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
$TUTOR plugins list
```

You should see at minimum: `mfe`, `forked-mfe`, `configuration_plugin`, `forked-header-footer`, `catalog_mfe`.

### Local hostnames

> Local development only — not required on staging/production servers.

Your `/etc/hosts` must resolve:

```text
127.0.0.1  local.openedx.io
127.0.0.1  apps.local.openedx.io
127.0.0.1  studio.local.openedx.io
```

---

## Part 1 — Using Catalog MFE locally (`tutor dev`)

### 1.1 Clone the repository (first time only)

Clone into the **Tutor root** (same pattern as other Sherab MFEs):

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
cd "$($TUTOR config printroot)"

git clone https://github.com/OpenPecha/frontend-app-catalog.git
cd frontend-app-catalog
git checkout wbc-ulmo1-stage
git pull origin wbc-ulmo1-stage
```

Add upstream (optional, for reference):

```bash
git remote add upstream https://github.com/openedx/frontend-app-catalog.git
git fetch upstream --tags
```

### 1.2 Install Node dependencies

Catalog requires Node 24:

```bash
cd "$($TUTOR config printroot)/frontend-app-catalog"
nvm use    # reads .nvmrc → 24
npm ci
```

### 1.3 Install Sherab header/footer (required for logo + branding)

Catalog must use the OpenPecha forked header (same as learning/authoring). Run once after clone or after `npm ci`:

```bash
npm install --no-save \
  '@edx/frontend-component-footer@git+https://github.com/OpenPecha/frontend-component-footer.git#wbc-ulmo1-stage' \
  '@edx/frontend-component-header@git+https://github.com/OpenPecha/frontend-component-header.git#wbc-stage-v8.0.0' \
  '@edx/brand@npm:@openedx/brand-openedx@latest' \
  sharp #Image processing library; required by some MFE webpack/build steps on macOS
```

This ensures the logo links to the LMS homepage (which redirects to Catalog), not `/dashboard`.

### 1.4 Mount Catalog in Tutor

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
$TUTOR mounts add "$($TUTOR config printroot)/frontend-app-catalog"
$TUTOR mounts list
```

Expected mount target: `catalog` service at `/openedx/app`.

### 1.5 Confirm Tutor plugin configuration

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

> `CATALOG_URLS_DEV` is used by `tutor dev`. `CATALOG_URLS_PROD` is used by `tutor local` — update it to your real `https://apps.your-domain.org/catalog` host on staging/production servers (see §2.2). The `INSTALL_SEARCH_440` block patches the openedx Dockerfile to force `edx-search==4.4.0` (required Ulmo.1 workaround — without it, Catalog course search returns 404).

Regenerate config after any plugin change:

```bash
$TUTOR config save
```

### 1.6 Build images

Build Catalog dev image:

```bash
$TUTOR images build catalog-dev
```

Build/rebuild openedx dev image (required for `edx-search==4.4.0`):

```bash
$TUTOR images build openedx-dev
```

> `openedx-dev` rebuild can take 45–90+ minutes.

After rebuilding openedx, **recreate** LMS/CMS containers (`tutor dev restart` alone reuses the old container and will not pick up `edx-search==4.4.0`):

```bash
docker compose \
  -f "$($TUTOR config printroot)/env/local/docker-compose.yml" \
  -f "$($TUTOR config printroot)/env/dev/docker-compose.yml" \
  --project-name tutor_dev up -d --force-recreate lms cms
```

### 1.7 Start local environment

```bash
$TUTOR dev start -d
$TUTOR dev status
```

Catalog service: `tutor_dev-catalog-1` on port **1998**.

### 1.8 Local URLs

| Page | URL |
|------|-----|
| Catalog home (MFE) | http://apps.local.openedx.io:1998/catalog/ |
| Course catalog (MFE) | http://apps.local.openedx.io:1998/catalog/courses |
| Course about (MFE) | http://apps.local.openedx.io:1998/catalog/courses/{course-id}/about |
| LMS root (redirects) | http://local.openedx.io:8000/ → Catalog home |
| LMS `/courses` (redirects) | http://local.openedx.io:8000/courses → Catalog courses |
| MFE config API | http://local.openedx.io:8000/api/mfe_config/v1?mfe=catalog |

Replace `{course-id}` with the full course key, e.g. `course-v1:Org+Course+Run`.

## Part 2 — Using Catalog MFE on server (staging & production)

Server environments use **`tutor local`**, not `tutor dev`.

### 2.1 Branch mapping (Sherab standard)

| Environment | Catalog branch | Tutor command |
|-------------|----------------|---------------|
| Staging | `wbc-ulmo1-stage` | `tutor local` on staging server |
| Production | `wbc-ulmo1-prod` | `tutor local` on production server |

### 2.2 Server plugin requirements

Same plugins as local must be present in the server's `tutor-plugins/` directory and enabled:

- `forked-mfe.py`
- `catalog_mfe.py`
- `forked-header-footer.py`

On the server, update **`catalog_mfe.py` production URLs** (`CATALOG_URLS_PROD`) to match the real domain. Local dev uses:

```python
# development
CATALOG_MICROFRONTEND_URL = "http://apps.local.openedx.io:1998/catalog"
```

On staging/production, replace with your real hosts, for example:

```python
# Example — use your actual LMS/MFE hostnames from server config.yml
CATALOG_MICROFRONTEND_URL = "https://apps.your-domain.org/catalog"
MFE_CONFIG["CATALOG_MFE_URL"] = CATALOG_MICROFRONTEND_URL
MFE_CONFIG["CATALOG_MICROFRONTEND_URL"] = CATALOG_MICROFRONTEND_URL
```

Also ensure `forked-mfe.py` points to the correct branch:

```python
# Staging server
"version": "wbc-ulmo1-stage",

# Production server
"version": "wbc-ulmo1-prod",
```

### 2.3 Server deployment steps

Run on the **target server** (staging or production):

```bash
# 1. Activate tutor venv on server
source /path/to/tutor-venv/bin/activate

# 2. Pull latest plugin + config changes (via your normal deploy process)
tutor config save

# 3. Build MFE production image (includes catalog-prod)
tutor images build catalog-prod

# 4. Build openedx image (edx-search 4.4.0 patch applies here)
tutor images build openedx

# 5. Rebuild/start local (production) stack
tutor local stop
tutor local start -d

# 6. If openedx was rebuilt, recreate LMS to pick up new image
tutor local restart lms cms
```

If course search still fails after restart, force-recreate LMS (same lesson as local):

```bash
docker compose \
  -f "$(tutor config printroot)/env/local/docker-compose.yml" \
  -f "$(tutor config printroot)/env/local/docker-compose.prod.yml" \
  --project-name tutor_local up -d --force-recreate lms cms
```

### 2.4 Server URLs (pattern)

| Legacy LMS route | Behavior with Catalog enabled |
|------------------|-------------------------------|
| `https://lms.your-domain.org/` | Redirect → `https://apps.your-domain.org/catalog/` |
| `https://lms.your-domain.org/courses` | Redirect → `https://apps.your-domain.org/catalog/courses` |
| `https://lms.your-domain.org/courses/{id}/about` | Redirect → `https://apps.your-domain.org/catalog/courses/{id}/about` |

### 2.5 Server verification checklist

After deploy, confirm:

1. `https://lms.your-domain.org/` redirects to Catalog home
2. Catalog home loads course cards (not error box)
3. `/catalog/courses` loads searchable course list
4. A course about page opens from catalog
5. Logo click goes to homepage (LMS root → Catalog)
6. Inside LMS container: `pip show edx-search` → **Version: 4.4.0**

```bash
tutor local run lms pip show edx-search | grep Version
```

## Part 3 — Differences: new Catalog MFE pages vs legacy LMS pages

When `ENABLE_CATALOG_MICROFRONTEND = True`, the LMS **redirects** legacy public routes to the Catalog MFE. The old Django/HTML pages are no longer rendered for those routes.

### 3.1 Page mapping overview

```text
LEGACY (edx-platform LMS)              NEW (Catalog MFE)
─────────────────────────              ───────────────────
/                                      /catalog/
/courses                               /catalog/courses
/courses/{course_id}/about             /catalog/courses/{course_id}/about
```

### 3.2 Detailed comparison

#### A. Home / Index page

| Aspect | Legacy LMS | Catalog MFE |
|--------|------------|-------------|
| **URL** | `http://local.openedx.io:8000/` | `http://apps.local.openedx.io:1998/catalog/` |
| **Technology** | Django template (`branding/views.py`) | React + Paragon (`frontend-app-catalog`) |
| **Content** | Static marketing/home template | Hero banner, promo video slot, featured courses |
| **Course data** | Legacy course discovery APIs / templates | `POST /search/unstable/v0/course_list_search/` |

**Redirect:** LMS `/` → `{CATALOG_MICROFRONTEND_URL}/` (301)

> 301 = permanent redirect; the browser follows to the Catalog URL.

#### B. Course catalog page

| Aspect | Legacy LMS | Catalog MFE |
|--------|------------|-------------|
| **URL** | `http://local.openedx.io:8000/courses` | `http://apps.local.openedx.io:1998/catalog/courses` |
| **Technology** | Django `/courses` view + HTML | React catalog page with data table/cards |
| **Search/filters** | Legacy discovery/search UI | Faceted search via `course_list_search` API |

**Redirect:** LMS `/courses` → `{CATALOG_MICROFRONTEND_URL}/courses` (301)

#### C. Course About page

| Aspect | Legacy LMS | Catalog MFE |
|--------|------------|-------------|
| **URL** | `/courses/{course_id}/about` | `/catalog/courses/{course_id}/about` |
| **Technology** | Django `courseware/views/about` template | React course-about components |

**Redirect:** When Catalog MFE is enabled, LMS about view redirects via `get_link_for_about_page()` to:

```text
{CATALOG_MICROFRONTEND_URL}/courses/{course_id}/about
```

### 3.3 Pages that are NOT replaced (still existing)

These continue to work as before — different MFEs or LMS views:

| Page / feature | Still handled by |
|----------------|------------------|
| Learner dashboard | `frontend-app-learner-dashboard` MFE / LMS |
| Login / Register | `frontend-app-authn` MFE |
| Account / Profile | `frontend-app-account` / `frontend-app-profile` |
| Courseware (learning) | `frontend-app-learning` MFE |
| Studio (authoring) | `frontend-app-authoring` MFE |
| Programs, certificates, admin | LMS / other MFEs |

### 3.4 Backend / API differences

| Item | Legacy | Catalog MFE |
|------|--------|-------------|
| Course list search | `/courses` page rendering + discovery | `POST /search/unstable/v0/course_list_search/` |
| edx-search version | Ulmo.1 default 4.3.0 | Requires **4.4.0** (`catalog_mfe.py` plugin) |
| MFE runtime config | N/A for legacy pages | `GET /api/mfe_config/v1?mfe=catalog` |
| Enable flag | N/A | `ENABLE_CATALOG_MICROFRONTEND` + `FEATURES['ENABLE_CATALOG_MICROFRONTEND']` |

> You must enable Catalog mode in LMS via `catalog_mfe.py` (`ENABLE_CATALOG_MICROFRONTEND` + `FEATURES['ENABLE_CATALOG_MICROFRONTEND']`) so redirects and config point at the MFE.

---

## Part 4 — Quick reference

### Key files (local)

| File | Path |
|------|------|
| Catalog clone | `$(tutor config printroot)/frontend-app-catalog` |
| MFE registry | `~/Library/Application Support/tutor-plugins/forked-mfe.py` |
| Catalog LMS + edx-search plugin | `~/Library/Application Support/tutor-plugins/catalog_mfe.py` |
| Header/footer plugin | `~/Library/Application Support/tutor-plugins/forked-header-footer.py` |

### Related Sherab documentation

- [03 — Mounting Frontend MFEs](03-mounting-frontend-mfe.md)
- [14 — Header, Footer & Brand Overrides](14-header-footer-brand-override.md)
- [16 — Development, Testing & Deployment Workflow](16-development-testing-deployment-workflow.md)
- [17 — MFE Header Versioning](17-mfe-header-versioning.md)

### Official upstream references

- Catalog MFE: https://github.com/openedx/frontend-app-catalog
- Ulmo Catalog release notes: https://docs.openedx.org/en/latest/community/release_notes/ulmo/ulmo_catalog.html
- edx-search 4.4.0 requirement: documented in Catalog MFE README (Ulmo.1 workaround)
