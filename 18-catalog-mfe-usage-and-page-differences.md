# 18. Catalog MFE — Local & Server Setup

**Repository:** [https://github.com/OpenPecha/frontend-app-catalog](https://github.com/OpenPecha/frontend-app-catalog)
**Branches:** `wbc-ulmo1-stage` (local + staging) · `wbc-ulmo1-prod` (production)

Catalog replaces the legacy LMS Home, Course Catalog, and Course About pages.

---

## Prerequisites

| Requirement | Value |
| --- | --- |
| Tutor | **21.0.1** (Ulmo) |
| Node.js | **24** (use the version in the repo `.nvmrc`) |
| `/etc/hosts` (local only) | `local.openedx.io`, `apps.local.openedx.io`, `studio.local.openedx.io` → `127.0.0.1` |

### Required plugins (enable on local and server)

| Plugin | Purpose |
| --- | --- |
| `catalog_mfe.py` | Enables Catalog, sets LMS URLs, pins `edx-search==4.4.0` (Ulmo.1 workaround), Registers Catalog MFE (port 1998)  |
| `catalog_customization.py` | Injects the Buy Course slot into the generated `env.config.jsx` (catalog-scoped) |

> Catalog LMS settings live in `catalog_mfe.py` only — **not** `configuration_plugin.yml`. Disable these plugins when a future Tutor release ships native Catalog config.

```bash
tutor plugins list   # expect: mfe, catalog_mfe, catalog_customization
```

---

> **Before you begin**, make sure your Tutor virtual environment is activated and you're working inside it. Every command below calls `tutor` directly, so it must be on your PATH.

## Local setup (`tutor dev`)

### 1. Clone and install

```bash
cd "$(tutor config printroot)"

git clone https://github.com/OpenPecha/frontend-app-catalog.git
cd frontend-app-catalog
git checkout wbc-ulmo1-stage
git pull origin wbc-ulmo1-stage

nvm use && npm ci
```

### 2. Mount

```bash
tutor mounts add "$(tutor config printroot)/frontend-app-catalog"
tutor mounts list   # expect: catalog service mounted at /openedx/app
```

### 3. Verify the Tutor plugins

These are already set in Sherab — verify they exist, don't hand-edit generated env files.

`catalog_mfe.py` — LMS settings, the edx-search patch and the catalog registry:

```python
from tutor import hooks
from tutormfe.hooks import MFE_APPS

# Temporary Ulmo.1 Catalog MFE settings.
# Disable/remove this plugin when Tutor ships native Catalog config
# to avoid duplicate LMS settings.

# Register the Catalog MFE (it is NOT a core Tutor MFE, so it must be
# registered here for the catalog service + CORS/CSRF entries to be
# generated in both dev and prod).
@MFE_APPS.add()
def _add_catalog_mfe(mfes):
    mfes["catalog"] = {
        "repository": "https://github.com/OpenPecha/frontend-app-catalog.git",
        "port": 1998,
        "version": "wbc-ulmo1-stage",
    }
    return mfes


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

> `INSTALL_SEARCH_440` forces `edx-search==4.4.0` in the openedx image — a required Ulmo.1 workaround. Without it, Catalog course search returns 404.

`catalog_customization.py` — injects the Buy Course slot into the server's generated `env.config.jsx`, scoped to the catalog MFE only:

```python
from tutor import hooks

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

> **Do not** use the global `mfe-env-config-buildtime-imports` patch for Buy Course — a static import there applies to *every* MFE and breaks `tutor images build mfe` for other apps. The catalog-scoped `mfe-env-config-runtime-definitions-catalog` patch only runs inside the `APP_ID == 'catalog'` block.

### 4. Create the local dev `env.config.jsx`

`env.config.jsx` is gitignored and **never committed**. On the server, `catalog_customization.py` injects the slot; but `tutor dev` bind-mounts the repo over `/openedx/app` and hides Tutor's generated file — so for local dev you create it yourself in the repo root:

```bash
cd "$(tutor config printroot)/frontend-app-catalog"
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

> Keep this file local. Do not commit it or copy it to servers.

Regenerate config after any plugin change:

```bash
tutor config save
```

### 5. Build, start, and verify

```bash
tutor images build catalog-dev
tutor images build openedx-dev    # required for edx-search 4.4.0; can take 45–90+ min

# Force-recreate LMS/CMS so they pick up edx-search==4.4.0
# (`tutor dev restart` alone reuses the old container):
docker compose \
  -f "$(tutor config printroot)/env/local/docker-compose.yml" \
  -f "$(tutor config printroot)/env/dev/docker-compose.yml" \
  --project-name tutor_dev up -d --force-recreate lms cms

tutor dev start -d
tutor dev status
```

Verify (Catalog service runs as `tutor_dev-catalog-1` on port **1998**):

- [ ] [http://apps.local.openedx.io:1998/catalog/](http://apps.local.openedx.io:1998/catalog/) loads
- [ ] [http://local.openedx.io:8000/](http://local.openedx.io:8000/) redirects to Catalog
- [ ] `docker exec tutor_dev-lms-1 pip show edx-search` → **4.4.0**
- [ ] Paid course about page → **Buy Course** (redirects to WooCommerce store → enrollment)
- [ ] Free course about page → **Enroll now**
- [ ] Enrolled user → **View course**

> The Buy Course behavior is already wired by the 2 plugins above plus the committed catalog branch — no extra steps. If a check fails, see [Reference Only - Not a Setup Step: Buy Course Customization Explained](#reference-only---not-a-setup-step-buy-course-customization-explained).

---

## Server setup (staging & production)

Servers use **`tutor local`**, not `tutor dev`.

### Before deploy

1. Ensure `forked-mfe.py`, `catalog_mfe.py`, and `catalog_customization.py` are on the server and enabled.
2. Update `CATALOG_URLS_PROD` in `catalog_mfe.py` to your real MFE URL (e.g. `https://apps.your-domain.org/catalog`).
3. Set the `catalog_mfe.py` version to `wbc-ulmo1-stage` (staging) or `wbc-ulmo1-prod` (production).
4. Confirm the pinned branch includes `src/plugins/BuyCourseEnrollmentButton.tsx` and `EnrolledStatus.tsx`. No `env.config.jsx` is committed — the server slot comes from `catalog_customization.py`.

### Deploy

```bash
tutor config save
tutor images build mfe
tutor images build openedx      # if edx-search or LMS settings changed
tutor local stop && tutor local start -d
tutor local restart lms cms mfe
```

If course search fails after restart, force-recreate LMS/CMS (same as local).

---

## Reference Only - Not a Setup Step: Buy Course Customization Explained

> New devs can skip this — after steps 1–5 the Buy Course fix is already active. Read on only if you need to **change** Buy Course behavior: switch the store/redirect, change the paid-vs-free rule, edit `EnrolledStatus`, debug a wrong button, or retire these plugins after a future native-Catalog Tutor release.

Sherab uses **WordPress/WooCommerce** for paid courses (see [04 — E-commerce Setup](04-sherab-ecommerce-setup.md)). The legacy LMS about page showed **Buy Course** via `CourseMode.product_url`. The default Catalog MFE instead shows **Enroll now** (Oscar checkout) — wrong for WooCommerce.

| Layer | Legacy LMS | Default Catalog |
| --- | --- | --- |
| Paid course button | **Buy Course** → WooCommerce | **Enroll now** |
| Data source | `CourseMode.product_url` | Courseware API (Oscar) |
| Enrolled state | **View course** visible | hidden unless `showCoursewareLink` |

**Button logic:** `GET /api/courses/v1/courses/{courseId}/` → if `purchase_link` is set, show **Buy Course** and redirect to the WooCommerce store; otherwise **Enroll now**. `EnrolledStatus.tsx` always shows **View course** when enrolled.

**How the slot reaches each environment:**

- **Local (`tutor dev`):** bind-mounted repo — the gitignored `env.config.jsx` (step 4) registers the slot.
- **Staging / prod (`tutor local`):** no mount — `catalog_customization.py` injects the catalog-scoped slot during the MFE Docker build.

**Prerequisites (already on Sherab):**

- `ENABLE_EXTENDED_COURSE_DETAILS = True` in `configuration_plugin.yml`
- CourseMode **Product URL** verified per paid course in Studio
- WordPress + `openedx-commerce` OAuth
- Oscar checkout disabled in LMS admin

**Known issues (out of scope):**

- **Guest checkout** false error on WooCommerce — separate WordPress `openedx-commerce` fix; enrollment still succeeds if the billing email matches the LMS account.
- **LMS redirect after checkout** — not required.

---

## Key files

| File | Path |
| --- | --- |
| Catalog clone | `$(tutor config printroot)/frontend-app-catalog` |
| Buy Course component | `frontend-app-catalog/src/plugins/BuyCourseEnrollmentButton.tsx` |
| Enrolled status component | `frontend-app-catalog/src/course-about/course-intro/components/EnrolledStatus.tsx` |
| Enrolled status tests | `.../course-intro/components/__tests__/EnrolledStatus.test.tsx` |
| Dev slot config (gitignored) | `frontend-app-catalog/env.config.jsx` |
| MFE registry | `~/Library/Application Support/tutor-plugins/forked-mfe.py` |
| Catalog plugin (setup) | `~/Library/Application Support/tutor-plugins/catalog_mfe.py` |
| Catalog customization plugin | `~/Library/Application Support/tutor-plugins/catalog_customization.py` |

**Related:** [04 — E-commerce Setup](04-sherab-ecommerce-setup.md) · [14 — Header, Footer & Brand Overrides](14-header-footer-brand-override.md) · [16 — Deployment Workflow](16-development-testing-deployment-workflow.md)
