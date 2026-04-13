# 02. Local Tutor Setup and Customisation Guide

Up to this point, you have a vanilla (plain vanilla) Open edX installation running. Now, it's time to make it the Sherab platform by applying our custom theme, custom core platform, and plugins.

---

## 1. Verify Your Environment
Before we begin, ensure your python virtual environment is active, and confirm your tutor version is currently **19.0.2** (Sumac).
```bash
tutor --version
```

---

## 2. Install the Sherab Theme
We need to clone our custom CSS/UI repo into Tutor's theme directory.
1. Change into the correct directory:
   ```bash
   cd "$(tutor config printroot)/env/build/openedx/themes"
   ```
2. Clone the theme repository from GitHub:
   ```bash
   git clone https://github.com/OpenPecha/Sherab-theme.git
   ```

---

## 3. Use Our Custom `edx-platform`
The `edx-platform` is the massive core codebase for Open edX. 

1. Navigate back to the Tutor "root" directory:
   ```bash
   cd "$(tutor config printroot)"
   ```
2. Clone our custom edx-platform:
   ```bash
   git clone --depth=1 -b sherab-dev https://github.com/Esukhia/edx-platform.git
   ```
   *(The `--depth=1` flag saves you hundreds of megabytes by not downloading the entire history.)*
3. "Mount" the platform. This tells Tutor to look at your folder instead of its internal code!
   ```bash
   tutor mounts add ./edx-platform
   ```

---
## 4. Relaunch Tutor with Custom Platform
   ```bash
   tutor dev launch
   ```
This will now use your custom edx-platform and theme.

## 5. Add the Configuration Plugin
We have specific Feature Flags and configurations required.

1. Create a folder for local tutor plugins inside the tutor root directory:
   ```bash
   mkdir -p tutor-plugins/configuration_plugin
   cd tutor-plugins/configuration_plugin
   ```
2. Create a file named `plugin.yml` in this folder:
   *(Copy this configuration exactly as is. **Do not modify this** unless instructed.)*
   ```yaml
   name: configuration_plugin
   version: 0.0.1
   patches:
      openedx-lms-common-settings: |
         FEATURES['ALWAYS_REDIRECT_HOMEPAGE_TO_DASHBOARD_FOR_AUTHENTICATED_USER'] = False
         FEATURES['CUSTOM_COURSES_EDX'] = True
         FEATURES['ENABLE_CHANGE_USER_PASSWORD_ADMIN'] = True
         FEATURES['ENABLE_SPECIAL_EXAMS'] = True
         FEATURES['MILESTONES_APP'] = True
         FEATURES['ENABLE_PREREQUISITE_COURSES'] = True
         FEATURES['ENABLE_DISCUSSION_HOME_PANEL'] = True
         FEATURES['ENABLE_DISCUSSION_EMAIL_DIGEST'] = True
         FEATURES['ENABLE_ACCOUNT_DELETION'] = False
         FEATURES['CUSTOM_CERTIFICATE_TEMPLATES_ENABLED'] = True
         FEATURES['SHOW_HEADER_LANGUAGE_SELECTOR'] = True
         FEATURES['SHOW_FOOTER_LANGUAGE_SELECTOR'] = False
         FEATURES['ORGANIZATIONS_APP'] = True
         FEATURES['ENABLE_COURSE_DISCOVERY'] = True
         FEATURES['ENABLE_EXTENDED_COURSE_DETAILS'] = True
         FEATURES['ENABLE_COSMETIC_DISPLAY_PRICE'] = True
         FEATURES['COURSE_PRODUCT_FALLBACK_URL'] = 'http://localhost:8080/'
         DEFAULT_FROM_EMAIL = "contact@sherab.org"
         CONTACT_EMAIL = "contact@sherab.org"
         DEFAULT_FEEDBACK_EMAIL = "contact@sherab.org"
         TECH_SUPPORT_EMAIL = "contact@sherab.org"
         COMMUNITY_FORUM_URL = ""
         SEARCH_SKIP_ENROLLMENT_START_DATE_FILTERING = True

      openedx-cms-common-settings: |
         FEATURES['ENABLE_SPECIAL_EXAMS'] = True
         FEATURES['MILESTONES_APP'] = True
         FEATURES['ENABLE_PREREQUISITE_COURSES'] = True
         FEATURES['ALLOW_PUBLIC_ACCOUNT_CREATION'] = False
         FEATURES['ENABLE_EXTENDED_COURSE_DETAILS'] = True
         ENABLE_CUSTOM_COURSE_CONFIG = True
         DEFAULT_FROM_EMAIL = "contact@sherab.org"
         CONTACT_EMAIL = "contact@sherab.org"
         DEFAULT_FEEDBACK_EMAIL = "contact@sherab.org"
         TECH_SUPPORT_EMAIL = "contact@sherab.org"
         DEFAULT_MOBILE_AVAILABLE = True
         GOOGLE_SHEETS_CREDENTIALS_FILE = "/edx/etc/google-sheet-credential/sherab-store-email-service-67f755324915.json"
         GOOGLE_SHEETS_SPREADSHEET_ID = "1oRsL7GsvzDzQIlAHNT8zfbJgWcsDPrfM3wGkUpO2dNk"
         GOOGLE_SHEETS_SHEET_NAME = "Form responses 1"
         FEATURES['ENABLE_VIDEO_UPLOAD_PIPELINE'] = True
         FEATURES['AUTO_REGISTER_SOURCE_VIDEO'] = True

   ```

3. Create the `mount_python_package.py` file to automatically register our next plugin:
   ```python
   from tutor import hooks
   
   hooks.Filters.MOUNTED_DIRECTORIES.add_item(
       ("openedx", "sherab-custom-plugin")
   )
   ```

---

## 6. Clone and Mount Our Custom Plugin
The actual backend Python logic specific to Sherab lives in `sherab-custom-plugin`.

1. Go back to Tutor Root:
   ```bash
   cd "$(tutor config printroot)"
   ```
2. Clone the plugin (this contains features like wishlists):
   ```bash
   git clone https://github.com/OpenPecha/sherab-custom-plugin.git
   ```
3. Mount it to Tutor:
   ```bash
   tutor mounts add ./sherab-custom-plugin
   ```

---

## 7. Verify and Enable Plugins
Let's make sure Tutor sees all of this.
1. Check your mounts. You should see both `edx-platform` and `sherab-custom-plugin`.
   ```bash
   tutor mounts list
   ```
2. Enable our configuration plugin we made in step 5:
   ```bash
   tutor plugins enable configuration_plugin
   ```

---

## 8. Rebuild and Restart
Because we fundamentally changed the codebase and settings, we must rebuild the Docker image from scratch.

1. Rebuild the Open edX Docker container image (this will take time):
   ```bash
   tutor images build openedx --no-cache
   ```
2. Shut down the old version and launch the new one:
   ```bash
   tutor dev stop
   tutor dev start -d
   ```
   *(`-d` runs it in detached mode so you get your terminal back).*

---

## 9. Verify the Plugin Installed in the Container
Sometimes python packages fail to attach. Let's physically check the docker container!

1. Find the **LMS container ID**.
   ```bash
   docker ps 
   ```
   *(Look for something named `tutor_dev_lms_1` or similar. Copy the ID under the `CONTAINER ID` column).*
2. Enter the container:
   ```bash
   docker exec -it <your_container_id> bash
   ```
3. Check if python successfully installed the plugin package:
   ```bash
   pip list | grep sherab
   ```
   *(If you see `sherab-custom-plugin` listed, it worked!)*
4. **IF IT IS NOT INSTALLED**: Manually install it via the mount directory.
   ```bash
   # Inside the docker container
   pip install -e /mnt/sherab-custom-plugin
   ```
   *(If it says folder not found, double check it by running `cd /mnt && ls`)*
5. Exit the container back to your machine terminal:
   ```bash
   exit
   ```

---

## 10. Apply the Sherab Theme
Let's turn the website from default branding to Sherab.
1. Apply the theme via Tutor:
   ```bash
   tutor dev do settheme Sherab-theme
   ```
2. Restart once more:
   ```bash
   tutor dev restart
   ```

🎉 **You are completely done setting up the core Backend & Database!**

---
**Next Step:** Are you working on the frontend parts (like dashboards and learner portals)? If so, head to [03-mounting-frontend-mfe.md](03-mounting-frontend-mfe.md). If you're setting up eCommerce, head to [04-sherab-ecommerce-setup.md](04-sherab-ecommerce-setup.md).
