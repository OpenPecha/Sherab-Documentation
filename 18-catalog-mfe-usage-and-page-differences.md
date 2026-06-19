# 18. Catalog MFE — Local & Server Setup

**Repository:** [https://github.com/OpenPecha/frontend-app-catalog](https://github.com/OpenPecha/frontend-app-catalog)  
**Branches:** `wbc-ulmo1-stage` (local + staging) · `wbc-ulmo1-prod` (production)

Catalog replaces legacy LMS Home, Course Catalog, and Course About pages.

---

## Prerequisites


| Requirement               | Value                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------ |
| Tutor                     | **21.0.1** (Ulmo)                                                                    |
| Node.js                   | **24** (repo `.nvmrc`)                                                               |
| `/etc/hosts` (local only) | `local.openedx.io`, `apps.local.openedx.io`, `studio.local.openedx.io` → `127.0.0.1` |


> Use the version of node in the `.nvmrc` file.

### Required plugins (enable on local and server)


| Plugin                     | Purpose                                                                                       |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| `forked-mfe.py`            | Catalog MFE registry (port 1998)                                                             |
| `catalog_mfe.py`           | Enable Catalog, LMS URLs, `edx-search==4.4.0` (Ulmo.1 workaround) — setup only               |
| `catalog_customization.py` | Inject the Buy Course enrollment slot into the generated `env.config.jsx` (catalog-scoped)   |


> Catalog LMS settings live in `catalog_mfe.py` only — not `configuration_plugin.yml`. Disable these plugins when a future Tutor release ships native Catalog config.

```bash
TUTOR=~/openedx/tutor-venv/bin/tutor
$TUTOR plugins list   # expect: mfe, forked-mfe, catalog_mfe, catalog_customization
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

`**forked-mfe.py**` — Catalog registry:

```python
mfes["catalog"] = {
    "repository": "https://github.com/OpenPecha/frontend-app-catalog.git",
    "port": 1998,
    "version": "wbc-ulmo1-stage",
}
```

`**catalog_mfe.py**` — LMS settings and edx-search patch (setup only):

```python
from tutor import hooks

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

`**catalog_customization.py**` — injects the Buy Course slot into Tutor's generated `env.config.jsx`, scoped to the catalog MFE only (the component ships in catalog Git):

```python
from tutor import hooks

# Sherab Catalog MFE customizations (update as needed).
# WooCommerce Buy Course: inject the enrollment-button slot into Tutor's
# generated env.config.jsx, scoped to the catalog MFE only. The component
# ships in catalog Git (src/plugins/BuyCourseEnrollmentButton.tsx).
CATALOG_BUY_COURSE_SLOT = """
const { default: BuyCourseEnrollmentButton } = await import('./src/plugins/BuyCourseEnrollmentButton');
config.pluginSlots['org.openedx.frontend.catalog.course_about_page.enrollment_button'] = {
  keepDefault: false,
  plugins: [
    {
      op: PLUGIN_OPERATIONS.Insert,
      widget: {
        id: 'sherab_buy_course_enrollment_button',
        type: DIRECT_PLUGIN,
        RenderWidget: BuyCourseEnrollmentButton,
      },
    },
  ],
};
"""

hooks.Filters.ENV_PATCHES.add_items([
    ("mfe-env-config-runtime-definitions-catalog", CATALOG_BUY_COURSE_SLOT),
])
```

> The `INSTALL_SEARCH_440` block patches the openedx Dockerfile to force `edx-search==4.4.0` (required Ulmo.1 workaround — without it, Catalog course search returns 404).

> **Do not** use the global `mfe-env-config-buildtime-imports` patch for Buy Course — a static import there applies to *every* MFE and breaks `tutor images build mfe` for other apps. We use the catalog-scoped `mfe-env-config-runtime-definitions-catalog` patch instead, which only runs inside the `APP_ID == 'catalog'` block.

### 2.2 Create the local dev `env.config.jsx` (gitignored)

`env.config.jsx` is **not** committed to the catalog repo (it's gitignored). The server gets its slot config from `catalog_customization.py`, but `tutor dev` bind-mounts the repo over `/openedx/app` and hides Tutor's generated file — so for local dev you create `env.config.jsx` in the repo root:

```bash
cd "$($TUTOR config printroot)/frontend-app-catalog"
cat > env.config.jsx <<'EOF'
import { DIRECT_PLUGIN, PLUGIN_OPERATIONS } from '@openedx/frontend-plugin-framework';

import BuyCourseEnrollmentButton from './src/plugins/BuyCourseEnrollmentButton';

const ENROLLMENT_BUTTON_SLOT =
  'org.openedx.frontend.catalog.course_about_page.enrollment_button';

export default {
  pluginSlots: {
    [ENROLLMENT_BUTTON_SLOT]: {
      keepDefault: false,
      plugins: [
        {
          op: PLUGIN_OPERATIONS.Insert,
          widget: {
            id: 'sherab_buy_course_enrollment_button',
            type: DIRECT_PLUGIN,
            RenderWidget: BuyCourseEnrollmentButton,
          },
        },
      ],
    },
  },
};
EOF
```

This file is local only (listed in `.gitignore`); do not commit it.

Regenerate config after any plugin change:

```bash
$TUTOR config save
```

Verify `forked-mfe.py` has Catalog on port **1998** / branch `wbc-ulmo1-stage`, and that `catalog_mfe.py` and `catalog_customization.py` exist in `tutor-plugins/`.

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

- [http://apps.local.openedx.io:1998/catalog/](http://apps.local.openedx.io:1998/catalog/)
- [http://local.openedx.io:8000/](http://local.openedx.io:8000/) → redirects to Catalog
- `docker exec tutor_dev-lms-1 pip show edx-search` → **4.4.0**

---

## Server setup (staging & production)

Servers use `**tutor local`**, not `tutor dev`.


| Environment | Branch            | Command       |
| ----------- | ----------------- | ------------- |
| Staging     | `wbc-ulmo1-stage` | `tutor local` |
| Production  | `wbc-ulmo1-prod`  | `tutor local` |


### Before deploy

1. Ensure `forked-mfe.py`, `catalog_mfe.py`, and `catalog_customization.py` are on the server and enabled.
2. Update `CATALOG_URLS_PROD` in `catalog_mfe.py` to your real MFE URL (e.g. `https://apps.your-domain.org/catalog`).
3. Set `forked-mfe.py` version to `wbc-ulmo1-stage` (staging) or `wbc-ulmo1-prod` (production).
4. Ensure the catalog branch pinned by `forked-mfe.py` includes `src/plugins/BuyCourseEnrollmentButton.tsx` and `EnrolledStatus.tsx`. No `env.config.jsx` is committed — the server slot is injected by `catalog_customization.py`.

### Deploy

```bash
tutor config save
tutor images build mfe
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

## WooCommerce Buy Course (catalog course about page)

Sherab uses **WordPress/WooCommerce** for paid courses (see [04 — E-commerce Setup](04-sherab-ecommerce-setup.md)). The legacy LMS course-about page showed **Buy Course** via `CourseMode.product_url`. The default Catalog MFE shows **Enroll now** (Oscar checkout) — wrong for WooCommerce.

### Problem


| Layer              | Legacy LMS                   | Default Catalog                                    |
| ------------------ | ---------------------------- | -------------------------------------------------- |
| Paid course button | **Buy Course** → WooCommerce | **Enroll now**                                     |
| Data source        | `CourseMode.product_url`     | Courseware API (Oscar)                             |
| Enrolled state     | **View course** visible      | **View course** hidden unless `showCoursewareLink` |


### Solution (plugin slot)

The button component lives in catalog Git; the slot is registered differently per environment (local file for dev, Tutor plugin for server):


| File / plugin                                  | Purpose                                                                                   |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `src/plugins/BuyCourseEnrollmentButton.tsx`    | Course API `purchase_link` → **Buy Course** or **Enroll now** (committed)                  |
| `EnrolledStatus.tsx`                           | Always show **View course** when enrolled (committed)                                      |
| `env.config.jsx` (dev only, gitignored)        | Registers the slot for `tutor dev`; **not** committed                                      |
| `catalog_customization.py` (Tutor plugin)      | Injects the same slot into the generated `env.config.jsx` for the server (catalog-scoped)  |


**Button logic:** `GET /api/courses/v1/courses/{courseId}/` → if `purchase_link` is set, show **Buy Course** and redirect to the WooCommerce store; otherwise **Enroll now**.

**How it reaches each environment:**

- **Local (`tutor dev`):** bind-mounted catalog repo — a local, gitignored `env.config.jsx` registers the slot directly.
- **Staging / prod (`tutor local`):** no mount. Tutor generates `env.config.jsx`, and `catalog_customization.py` injects the catalog-scoped slot via `mfe-env-config-runtime-definitions-catalog` during the MFE Docker build. Nothing is committed for the server.

> **Do not** commit `env.config.jsx` to the catalog repo or copy it to servers. The server slot is injected by `catalog_customization.py`; keep the dev `env.config.jsx` local and gitignored.

### Catalog source files

The Buy Course feature lives in the catalog branch (`wbc-ulmo1-stage` / `wbc-ulmo1-prod`):

- `src/plugins/BuyCourseEnrollmentButton.tsx` — Buy Course / Enroll now logic
- `src/course-about/course-intro/components/EnrolledStatus.tsx` — always show View course when enrolled
- `src/course-about/course-intro/components/__tests__/EnrolledStatus.test.tsx` — tests
- `env.config.jsx` — **not committed** (gitignored, dev-only); the server slot is injected by `catalog_customization.py`

### Prerequisites (already on Sherab)

- `ENABLE_EXTENDED_COURSE_DETAILS = True` in `configuration_plugin.yml`
- Verified CourseMode **Product URL** per paid course in Studio
- WordPress + `openedx-commerce` OAuth
- Oscar checkout disabled in LMS admin

### Local verify

```bash
cd "$($TUTOR config printroot)/frontend-app-catalog"
git pull

# Ensure the local dev env.config.jsx exists (see step 2.2) — it is gitignored.

$TUTOR dev restart catalog
```

Checklist:

- [ ] Paid course about page → **Buy Course**
- [ ] **Buy Course** → WooCommerce store → enrollment
- [ ] Enrolled → **View course**
- [ ] Free course → **Enroll now**

### Known issues (out of scope)

- **Guest checkout** false error on WooCommerce — separate WordPress `openedx-commerce` fix; enrollment still succeeds if billing email matches LMS account.
- **LMS redirect after checkout** — not required.

---

## Key files


| File                         | Path                                                                  |
| ---------------------------- | --------------------------------------------------------------------- |
| Catalog clone                | `$(tutor config printroot)/frontend-app-catalog`                      |
| Buy Course component         | `frontend-app-catalog/src/plugins/BuyCourseEnrollmentButton.tsx`      |
| Dev slot config (gitignored) | `frontend-app-catalog/env.config.jsx`                                 |
| MFE registry                 | `~/Library/Application Support/tutor-plugins/forked-mfe.py`           |
| Catalog plugin (setup)       | `~/Library/Application Support/tutor-plugins/catalog_mfe.py`          |
| Catalog customization plugin | `~/Library/Application Support/tutor-plugins/catalog_customization.py` |


**Related:** [04 — E-commerce Setup](04-sherab-ecommerce-setup.md) · [14 — Header, Footer & Brand Overrides](14-header-footer-brand-override.md) · [16 — Deployment Workflow](16-development-testing-deployment-workflow.md)