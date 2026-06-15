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
| `catalog_mfe.py` | Enable Catalog, LMS URLs, `edx-search==4.4.0` (Ulmo.1 workaround), **WooCommerce Buy Course plugin slot** (staging/prod image build) |
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

**`catalog_mfe.py`** — LMS settings, edx-search patch, and **WooCommerce Buy Course** plugin-slot wiring for server builds:

```python
from tutor import hooks
from tutormfe.hooks import PLUGIN_SLOTS

# ... CATALOG_ENABLE, CATALOG_URLS_DEV, CATALOG_URLS_PROD, INSTALL_SEARCH_440 ...

MFE_CATALOG_BUY_COURSE_BUILDTIME_IMPORTS = """
import BuyCourseEnrollmentButton from './src/plugins/BuyCourseEnrollmentButton';
"""

PLUGIN_SLOTS.add_items([
    (
        "catalog",
        "org.openedx.frontend.catalog.course_about_page.enrollment_button",
        """
        {
            op: PLUGIN_OPERATIONS.Hide,
            widgetId: 'default_contents',
        },
        {
            op: PLUGIN_OPERATIONS.Insert,
            widget: {
                id: 'sherab_buy_course_enrollment_button',
                type: DIRECT_PLUGIN,
                RenderWidget: BuyCourseEnrollmentButton,
            },
        },
        """,
    ),
])

hooks.Filters.ENV_PATCHES.add_items([
    # ... openedx-lms-* and edx-search patches ...
    ("mfe-env-config-buildtime-imports", MFE_CATALOG_BUY_COURSE_BUILDTIME_IMPORTS),
])
```

> The `INSTALL_SEARCH_440` block patches the openedx Dockerfile to force `edx-search==4.4.0` (required Ulmo.1 workaround — without it, Catalog course search returns 404).

> **WooCommerce Buy Course:** On staging/production, Tutor **replaces** the catalog repo’s `env.config.jsx` during the MFE Docker build. The Buy Course button is injected via `PLUGIN_SLOTS` + `mfe-env-config-buildtime-imports` above (same pattern as tutor-indigo). The component source is `src/plugins/BuyCourseEnrollmentButton.tsx` in the catalog Git branch — not duplicated in the Python plugin file.

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

## WooCommerce Buy Course (catalog course about page)

Sherab uses **WordPress/WooCommerce** for paid courses (see [04 — E-commerce Setup](04-sherab-ecommerce-setup.md)). The legacy LMS course-about page showed **Buy Course** via `CourseMode.product_url`. The default Catalog MFE shows **Enroll now** (Oscar/ecommerce checkout path) — wrong for WooCommerce.

### What we changed

| Piece | Location | Local (`tutor dev`) | Staging / prod (`tutor local`) |
|-------|----------|---------------------|--------------------------------|
| Buy Course button | `src/plugins/BuyCourseEnrollmentButton.tsx` | Mounted repo | Git clone in Docker build |
| Plugin slot registration | Thin `env.config.jsx` in catalog repo | **Used** (bind-mount) | **Not used** — Tutor overwrites `env.config.jsx` at image build |
| Plugin slot injection | `catalog_mfe.py` `PLUGIN_SLOTS` | Optional (local uses `env.config.jsx`) | **Required** for Buy Course button |
| View course when enrolled | `EnrolledStatus.tsx` | Mounted repo | Shipped in catalog bundle after merge |

**Button logic:** `GET /api/courses/v1/courses/{courseId}/` → if `purchase_link` is set (from CourseMode `product_url`), show **Buy Course** and redirect to the WooCommerce store; otherwise show **Enroll now**.

**Prerequisites (already on Sherab):**

- `ENABLE_EXTENDED_COURSE_DETAILS = True` in `configuration_plugin.yml` (exposes `purchase_link` on the Course API)
- Verified CourseMode with **Product URL** set per paid course in Studio
- WordPress store + `openedx-commerce` OAuth ([doc 04](04-sherab-ecommerce-setup.md))
- Oscar checkout disabled in LMS admin

### Local dev checkout

After merging the catalog branch (or checking out `feature/catalog-woocommerce-buy-course-enrollment`):

```bash
cd "$($TUTOR config printroot)/frontend-app-catalog"
git pull   # branch with src/plugins/BuyCourseEnrollmentButton.tsx + thin env.config.jsx

$TUTOR dev restart catalog
```

Verify on a paid course about page: **Buy Course** → store checkout → enrollment → **View course** when enrolled.

> **Do not** copy `env.config.jsx` manually to staging/production servers. Deploy `catalog_mfe.py` and rebuild the MFE image instead.

---

## Server setup (staging & production)

Servers use **`tutor local`**, not `tutor dev`.

| Environment | Branch | Command |
|-------------|--------|---------|
| Staging | `wbc-ulmo1-stage` | `tutor local` |
| Production | `wbc-ulmo1-prod` | `tutor local` |

### Before deploy

1. Ensure `forked-mfe.py`, `catalog_mfe.py` (with **PLUGIN_SLOTS** for Buy Course), and `forked-header-footer.py` are on the server and enabled.
2. Update `CATALOG_URLS_PROD` in `catalog_mfe.py` to your real MFE URL (e.g. `https://apps.your-domain.org/catalog`).
3. Set `forked-mfe.py` version to `wbc-ulmo1-stage` (staging) or `wbc-ulmo1-prod` (production).
4. Merge the catalog PR that includes `src/plugins/BuyCourseEnrollmentButton.tsx` and `EnrolledStatus.tsx` into the branch pinned by `forked-mfe.py`.

### Deploy

```bash
tutor config save
tutor images build mfe          # rebuilds catalog-prod with PLUGIN_SLOTS + catalog Git branch
tutor images build openedx      # if edx-search or LMS settings changed
tutor local stop && tutor local start -d
tutor local restart lms cms mfe
```

If course search fails after restart, force-recreate LMS/CMS (same as local).

### Verify on server

- LMS root redirects to Catalog home
- Catalog loads course data (not error box)
- `tutor local run lms pip show edx-search` → **4.4.0**
- Paid course **about** page in Catalog shows **Buy Course** (not Enroll now)
- Enrolled user sees **View course** on the about page

---

## Key files

| File | Path |
|------|------|
| Catalog clone | `$(tutor config printroot)/frontend-app-catalog` |
| Buy Course component | `frontend-app-catalog/src/plugins/BuyCourseEnrollmentButton.tsx` |
| Local plugin slot only | `frontend-app-catalog/env.config.jsx` (thin; **local dev bind-mount only**) |
| MFE registry | `~/Library/Application Support/tutor-plugins/forked-mfe.py` |
| Catalog plugin | `~/Library/Application Support/tutor-plugins/catalog_mfe.py` |
| Header/footer | `~/Library/Application Support/tutor-plugins/forked-header-footer.py` |

**Related:** [04 — E-commerce Setup](04-sherab-ecommerce-setup.md) · [14 — Header, Footer & Brand Overrides](14-header-footer-brand-override.md) · [16 — Deployment Workflow](16-development-testing-deployment-workflow.md)
