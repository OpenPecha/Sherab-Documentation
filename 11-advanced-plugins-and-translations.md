# 11. Advanced Plugins and Translations

Welcome to the advanced features guide! At this point, you have a working Open edX Sherab platform. Now, we are going to add three powerful features: 
1. **Cairn** (for Analytics & Dashboards)
2. **Discovery** (for Course Programs)
3. **Translations** (for supporting multiple languages)

Don't worry, we will go through this step-by-step!

---

## 📊 1. Cairn: The Analytics Plugin
Cairn is a plugin that gives instructors and administrators beautiful dashboards to see data about learners. 

### Step 1: Install and Enable Cairn
Cairn used to be a commercial plugin, but is now available for free to all Tutor users.
First, install the plugin with:

```bash
tutor plugins install cairn
```

Then, we need to tell Tutor to activate this plugin. Open your terminal where Tutor is installed and run:

```bash
tutor plugins enable cairn
```

### Step 2: Launch the Platform
Any time we add a new plugin, we have to restart the platform so it registers. 
**If you are developing on your computer (Development):**
```bash
tutor dev launch
```
**If you are on a live server (Production):**
```bash
tutor local launch
```

### Step 3: Access the Analytics Dashboard
Once launched, you can view the data!
- **Local Development URL:** [http://data.local.openedx.io:2247](http://data.local.openedx.io:2247)
- **Production URL:** `http://data.<YOUR_LMS_HOST>`

You authenticate using your normal LMS user account. By default, you can only see data for courses where you are assigned the "Staff" role.

### Step 4: Give Users Access (Admin Commands)
Sometimes you need to give a specific user admin access to see everything.

**For Local Development:**
```bash
tutor dev do cairn-createuser --admin YOURUSERNAME YOURUSERNAME@YOUREMAIL.COM
# Adding default dashboards:
tutor dev do cairn-createuser --bootstrap-dashboards YOURUSERNAME YOURUSERNAME@YOUREMAIL.COM
```

**For Production:**
```bash
tutor local do cairn-createuser --admin YOURUSERNAME YOURUSERNAME@YOUREMAIL.COM
# Adding default dashboards:
tutor local do cairn-createuser --bootstrap-dashboards YOURUSERNAME YOURUSERNAME@YOUREMAIL.COM
```

### Step 5: Restrict a User to specific courses
If you only want a user to see data for one specific course:

**Local Development:**
```bash
tutor dev do cairn-createuser --course-id='course-v1:edX+DemoX+Demo_Course' YOURUSERNAME YOURUSERNAME@YOUREMAIL.COM
```
**Production:**
```bash
tutor local do cairn-createuser --course-id='course-v1:edX+DemoX+Demo_Course' YOURUSERNAME YOURUSERNAME@YOUREMAIL.COM
```

### Step 6: Collecting Past Events (Optional)
When you first install Cairn, it only tracks *new* data. If you want it to process old data from before you installed it, you must run an import tool.

**Local Development:**
```bash
tutor dev start -d cairn-clickhouse
tutor dev run \
  --volume="$(tutor config printroot)/data/lms/logs/:/var/log/openedx/:ro" \
  --volume="$(tutor config printroot)/env/plugins/cairn/apps/vector/file.toml:/etc/vector/file.toml:ro" \
  -e VECTOR_CONFIG=/etc/vector/file.toml cairn-vector
```

**Production:**
```bash
tutor local start -d cairn-clickhouse
tutor local run \
  --volume="$(tutor config printroot)/data/lms/logs/:/var/log/openedx/:ro" \
  --volume="$(tutor config printroot)/env/plugins/cairn/apps/vector/file.toml:/etc/vector/file.toml:ro" \
  -e VECTOR_CONFIG=/etc/vector/file.toml cairn-vector
```
*(Note: This might take a very long time if you have a lot of history!)*

---

## 🎓 2. Discovery: The Programs Feature
The Discovery service allows Open edX to bundle multiple courses together into "Programs".

### Step 1: Install and Enable Discovery
Open your terminal and run these commands to install and enable it:
```bash
tutor plugins install discovery
tutor plugins enable discovery
```

### Step 2: Launch the Platform
**For Local Development:**
```bash
tutor dev launch
```
**For Production:**
```bash
tutor local launch
```

### Step 3: Create a Discovery Admin User
Discovery has its own admin panel. We need to create a superuser for it.

**For Local Development:**
```bash
tutor dev run discovery ./manage.py createsuperuser
```
**For Production:**
```bash
tutor local run discovery ./manage.py createsuperuser
```
*(Now use these credentials to log in at `http://discovery.local.openedx.io/admin` locally, or `http(s)://discovery.<your lms host>/admin` in production).*

### Step 4: Re-Index Courses
Discovery needs to pull all your courses from the LMS. Let's force it to update.

**For Local Development:**
```bash
tutor dev run discovery ./manage.py refresh_course_metadata --partner_code=dev
tutor dev run discovery ./manage.py update_index --disable-change-limit
```

**For Production:**
```bash
tutor local run discovery ./manage.py refresh_course_metadata --partner_code=openedx
tutor local run discovery ./manage.py update_index --disable-change-limit
```

### Step 5: Caching Programs
We also need the LMS to pull the Programs back from Discovery! 
*(Note: For this to work, ensure the user `lms_catalog_service_user` is a Staff user in the LMS Admin panel first).*

Create the catalog integration (Run this only once):
**Local Development:**
```bash
tutor dev run lms ./manage.py lms create_catalog_integrations --enabled --internal_api_url="" --service_username=lms_catalog_service_user
```
**Production:**
```bash
tutor local run lms ./manage.py lms create_catalog_integrations --enabled --internal_api_url="" --service_username=lms_catalog_service_user
```

Finally, run the caching command. You must run this every time you create a new Program!
**Local Development:**
```bash
tutor dev run lms ./manage.py lms cache_programs --domain="local.openedx.io:8000"
```
**Production:**
```bash
tutor local run lms ./manage.py lms cache_programs --domain="local.openedx.io"
```

### Step 6: Show the "Programs" Tab in LMS
By default, the Programs tab is hidden from learners. Let's make it visible!
1. Go to your LMS admin panel (`/admin/programs/programsapiconfig/`).
2. Add a new configuration. Set `Marketing path` to `/programs` and check the **Enabled** box.
3. Now, if you want the "Explore Programs" button to link to your custom website, go to `Site Configurations` (`/admin/site_configuration/siteconfiguration/`).
4. In the JSON **site values** field, add the following dictionary:
   ```json
   "MKTG_URLS": {
       "ROOT": "https://custom-marketing-site-here.com"
   }
   ```

---

## 🌐 3. Handling Translations
Sherab is a multi-lingual platform. We manage translations in a custom centralized repository: `openedx-translations`.

### Which Languages Are Used?
These are the active language codes in our system:
- **bo** (Tibetan)
- **en** (English)
- **es_419** (Latin American Spanish)
- **fr** (French)
- **id** (Indonesian)
- **vi** (Vietnamese)
- **zh_CN** (Simplified Chinese)

### How to Add a New Translation String
When developers create new text in the code, they wrap it in translation functions. But you still need to provide the actual translated strings to the system!

Our system uses `.po` and `.json` files.

#### Scenario A: Adding a translation for Django (Backend/LMS)
If you add text to a Django template or Python file, you must add it to the `.po` files.
Here is what you do:
1. Open the `openedx-translations` repository.
2. Navigate to `translations/edx-platform/conf/locale/<LANGUAGE>/LC_MESSAGES/django.po`.
3. Add an empty line at the bottom, then provide the `msgid` (the english variable) and the `msgstr` (the translated text).

**Example:**
```po
msgid "Hi {username}, Welcome to {platform_name}"
msgstr "{username} ཁྱོད་ལ་ {platform_name} ཕེབས་པར་དགའ་བསུ་ཞུ།"
```

#### Scenario B: Adding a translation for MFE (Frontend React)
If you add text to a Micro-Frontend (like the Footer or Profile), you must add it to the `.json` files.
1. Open the `openedx-translations` repository.
2. Navigate to the specific MFE folder, e.g., `translations/frontend-component-footer/src/i18n/messages/<LANGUAGE>.json`.
3. Always make sure to also update the English base file, `translations/frontend-component-footer/src/i18n/transifex_input.json`.
4. Add your key-value pair to the dictionary.

**Example:**
```json
  "footer.colophon.about": "ང་ཚོ་སྐོར།",
  "footer.colophon.contact": "འབྲེལ་བ་གནང་བ།",
  "footer.colophon.privacy": "གསང་སྲུང་ཞིབ་འཇུག"
```

By adding these strings, the platform knows exactly what text to swap in when a user switches their language preference!

---
**Congratulations!** You have now configured advanced analytics, program bundling, and multi-lingual translations!
