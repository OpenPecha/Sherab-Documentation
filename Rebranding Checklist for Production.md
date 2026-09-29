# Production Rebranding Checklist

## 1. Merge Changes to Production Branch
Merge changes into the `wbc-ulmo1-stage` & `wbc-ulmo1-prod` branches for the following repositories:
- [ ] `edx-platform`
- [ ] `sherab-custom-plugin`
- [ ] `edx-ora2`
- [ ] `brand-openedx`
- [ ] `frontend-app-account`
- [ ] `frontend-app-authn`
- [ ] `frontend-app-authoring`
- [ ] `frontend-app-catalog`
- [ ] `frontend-app-discussions`
- [ ] `frontend-app-learner-dashboard`
- [ ] `frontend-app-learning`
- [ ] `frontend-app-profile`
- [ ] `frontend-app-extension`
- [ ] `frontend-component-footer`
- [ ] `frontend-component-header-v6.6.0`
- [ ] `frontend-component-header-v8.0.0`
- [ ] `openedx-translations`

---

## 2. Fix MFE Build Issue
- [ ] Create `mfe-bullseye-eol.py` in the `tutor-plugins` directory:

```python
import os
import tempfile

import tutormfe
from tutor import hooks

TEMPLATE = ("mfe", "build", "mfe", "Dockerfile")
ORIGINAL = "RUN apt update \\\n"
PATCHED = (
    "RUN sed -i -e 's|^# deb http://snapshot|deb http://snapshot|' \\\n"
    "           -e '\\|^deb http://deb.debian.org|d' /etc/apt/sources.list \\\n"
    "  && apt -o Acquire::Check-Valid-Until=false update \\\n"
)


def _shadow_root() -> str:
    upstream = os.path.join(
        os.path.dirname(tutormfe.__file__),
        "templates",
        *TEMPLATE,
    )
    content = open(upstream, encoding="utf-8").read()

    root = os.path.join(
        tempfile.gettempdir(),
        "tutor-mfe-bullseye-eol-templates",
    )
    shadow = os.path.join(root, *TEMPLATE)

    os.makedirs(os.path.dirname(shadow), exist_ok=True)
    open(shadow, "w", encoding="utf-8").write(
        content.replace(ORIGINAL, PATCHED, 1)
    )

    return root


hooks.Filters.ENV_TEMPLATE_ROOTS.add_item(
    _shadow_root(),
    priority=hooks.priorities.HIGH,
)
```

- [ ] Enable the plugin:
```bash
tutor plugins enable mfe-bullseye-eol
```

---

## 3. Update Configuration
- [ ] Update the platform name.
- [ ] Update the URLs for LMS and CMS.

---

## 4. Update Tutor Plugins
- [ ] In `configuration_plugin.yml`, add:

```yaml
openedx-common-settings: |
  EXTENSION_BASE_URL = "{% if ENABLE_HTTPS %}https{% else %}http{% endif %}://{{ MFE_HOST }}"

openedx-development-settings: |
  EXTENSION_BASE_URL = "http://{{ MFE_HOST }}:{{ get_mfe('extension').port }}"

mfe-lms-common-settings: |
  MFE_CONFIG["CATALOG_MICROFRONTEND_URL"] = CATALOG_MICROFRONTEND_URL
  MFE_CONFIG["EXTENSION_BASE_URL"] = EXTENSION_BASE_URL
```

- [ ] Update the `catalog_customization.py` plugin:

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

# Partner-logo carousel on the catalog homepage banner, kept alongside the
# default banner content (keepDefault stays true).
CATALOG_PARTNER_CAROUSEL_SLOT = """
const { default: PartnerCarousel } = await import('./src/plugins/PartnerCarousel');
config.pluginSlots['org.openedx.frontend.catalog.home_page.banner'] = {
  keepDefault: true,
  plugins: [
    {
      op: PLUGIN_OPERATIONS.Insert,
      widget: {
        id: 'sherab_partner_carousel',
        type: DIRECT_PLUGIN,
        priority: 60,
        RenderWidget: PartnerCarousel,
      },
    },
  ],
};
"""

# Category-tabbed course grid on the catalog homepage, replacing the default
# flat course list (keepDefault is false).
CATALOG_COURSE_CATEGORIES_SLOT = """
const { default: CourseCategories } = await import('./src/plugins/CourseCategories');
config.pluginSlots['org.openedx.frontend.catalog.home_page.courses_list'] = {
  keepDefault: false,
  plugins: [
    {
      op: PLUGIN_OPERATIONS.Insert,
      widget: {
        id: 'sherab_course_categories',
        type: DIRECT_PLUGIN,
        RenderWidget: CourseCategories,
      },
    },
  ],
};
"""

hooks.Filters.ENV_PATCHES.add_items([
    ("mfe-env-config-runtime-definitions-catalog", CATALOG_BUY_COURSE_SLOT),
    ("mfe-env-config-runtime-definitions-catalog", CATALOG_PARTNER_CAROUSEL_SLOT),
    ("mfe-env-config-runtime-definitions-catalog", CATALOG_COURSE_CATEGORIES_SLOT),
])
```

- [ ] Configure `frontend-app-extension` in `forked-mfe.py`:

```python
mfes["extension"] = {
    "repository": "https://github.com/Webuddhist-tech/frontend-app-extension.git",
    "port": 2003,
    "version": "wbc-ulmo1-prod",
}
```

- [ ] Configure `frontend-app-extension` in `forked-brand.py`:
```python
    ("mfe-dockerfile-post-npm-install-extension",
     """
RUN npm install '@edx/brand@git+https://github.com/OpenPecha/brand-openedx.git#wba-rebranding-main'
     """),
```

- [ ] Configure `frontend-app-extension` in `forked-header-footer.py`:
```python
    ("mfe-dockerfile-post-npm-install-extension",
     """
RUN npm install '@edx/frontend-component-footer@git+https://github.com/OpenPecha/frontend-component-footer.git#wba-rebranding-main'
RUN npm install '@edx/frontend-component-header@git+https://github.com/OpenPecha/frontend-component-header.git#wba-rebranding-v8-main'
     """),
```

---

## 5. Build Production Images
- [ ] Pull latest changes in `Sherab-theme`.
- [ ] Build the Open edX image:
- [ ] Build the MFE image:

---

## 6. Run Database Migrations
- [ ] Run migrations for `course_partnerships`:

```bash
tutor local run lms ./manage.py lms migrate course_partnerships
```

- This command also covers the schema changes from PR #16 (catalog/course-about backend changes) — no separate step needed for these:
  - [ ] `0013_partner_invite_instructions` — adds the `invite_instructions` field to `Partner`
  - [ ] `0010`–`0012_herocourse*` — adds the `HeroCourse` table (see section 8)

---

## 7. Update Google Auth Icon
**URL:** `/admin/third_party_auth/oauth2providerconfig/`

- [ ] Clear the **Icon class** field.
- [ ] Upload the SVG below into **Icon image**:

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24">
  <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"></path>
  <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"></path>
  <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z" fill="#FBBC05"></path>
  <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z" fill="#EA4335"></path>
</svg>
```
- [ ] Save changes.

---

## 8. Configure Homepage Hero Courses
**URL:** `/admin/course_partnerships/herocourse/`

- [ ] Add 2 course entries for the homepage.

---

### 9. Update the s3 config?
To be updated...

---

## 10. Add Invite Instructions for Palpung Courses
Palpung's courses are invite-only, so their **Invite Instructions** message needs to be set via the admin panel (School / Partner model) before launch.

- [ ] Confirm the exact wording with Menla, after Menla has checked with the Palpung admin.
- [ ] Add the confirmed message to Palpung's Partner record in the admin panel.
- [ ] Note: if a partner has no message set, a default fallback message is shown instead — so this isn't blocking, but should be done before launch for a proper Palpung-specific message.
