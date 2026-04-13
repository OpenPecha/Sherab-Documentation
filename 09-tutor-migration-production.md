# 09. Tutor Server Migration Guide

This document provides explicit instructions for migrating an existing, customized local or production Sherab environment across multiple Tutor releases.

> [!WARNING]
> Since we use a customized `edx-platform` fork, upgrading Tutor requires manually checking out the matching release branch in our git repository so the code versions stay perfectly in sync.

---

## Part 1: Migration from Palm (v16.1.8) to Sumac (v19.0.2)

### 💠 Tutor Migration Overview
1. Upgrade Tutor and its environment from v16.1.8 to v19.0.2
2. Migrate the Open edX codebase from Palm.4 to Sumac.2 (version-by-version)
3. Migrate the theme to be compatible with newer Django versions
4. Upgrade MFE (Micro Frontend) codebase to Sumac.2

### ✅ Detailed Migration Steps

#### 1. Take Full Backup
Backup necessary components:
- MySQL
- MongoDB (if used)
- Redis (optional)
- `$(tutor config printroot)/env`
- `$(tutor config printroot)/data`
- Custom plugins/themes
- `.local/share/tutor` directory

```bash
tutor local do mysql-backup
```

#### 2. Disable Customizations Temporarily
Disable custom themes and plugins to avoid conflicts during migration.
```bash
tutor config save --set CMS_THEME_NAME=dummy LMS_THEME_NAME=dummy
```
Disable non-essential plugins:
```bash
tutor plugins disable <plugin_name>
```

#### 3. Stop the Server
```bash
tutor local stop
```

#### 4. Upgrade Tutor Version by Version
Incremental upgrade steps (Palm → Quince → Redwood → Sumac):
```bash
pip install --upgrade "tutor==17.0.0"
tutor config save
tutor local quickstart
tutor local launch
```
Test if the server works fine after each version upgrade (login, course access, etc.). Repeat increments until Sumac.

#### 5. Upgrade Open edX Codebase
- Tutor automatically handles Open edX upgrades.
- If using a custom fork of `edx-platform`, update the fork's `open-release/<tag>`.
- Build a custom Docker image if needed:
```bash
tutor images build openedx
```

#### 6. Migrate Custom Themes
Replace deprecated template tags and static files. After upgrading to Sumac, re-enable the theme:
```bash
tutor config save --set LMS_THEME_NAME=mytheme CMS_THEME_NAME=mytheme
tutor images build openedx
```

#### 7. Upgrade MFE (Micro Frontends)
Pull and build updated MFEs:
```bash
tutor local mfe build
tutor local mfe launch
```
Update MFE URLs in `config.yml` if using custom MFE hosting.

#### 8. Rebuild Custom Docker Images
Rebuild your custom Docker image after the upgrade:
```bash
tutor images build openedx
```

#### 9. Re-enable Plugins and Packages
Enable plugins one-by-one after each upgrade and verify their functionality:
```bash
tutor plugins enable <plugin_name>
```
After each plugin enablement, run:
```bash
tutor local init
tutor local launch
```

<br>

---

## Part 2: Migration from Sumac.2 to Ulmo.1 

We recently updated from Sumac to Ulmo. Here are the explicit steps required to step up the environment safely for `edx-platform`.

### 1. Preparation
1. **Take a database backup** (Optional for development):
   ```bash
   tutor dev do mysql-backup
   ```
2. **Stop the server:**
   ```bash
   tutor dev stop
   ```
3. **Unset the custom theme:**
   ```bash
   tutor dev do settheme default
   ```
4. **Disable non-essential plugins** (make sure to keep `mfe` enabled):
   ```bash
   # For example:
   tutor plugins disable cairn forum s3 wordpress
   # Disable our custom plugins momentarily to prevent conflicts
   tutor plugins disable <plugin_name>
   ```
5. **Remove all mounts EXCEPT edx-platform**:
   ```bash
   tutor mounts list
   # Use 'tutor mounts remove <path>' for everything EXCEPT edx-platform
   ```

### 2. Phase 1: Upgrade to Teak (Tutor 20)
Before we can reach Ulmo, we must upgrade through Teak.

1. **Install Tutor v20:**
   ```bash
   pip install --upgrade "tutor[full]>=20.0.0,<21.0.0"
   tutor config save
   ```
2. **Update the `edx-platform` code to the Teak branch:**
   ```bash
   git fetch origin sherab-teak-dev
   git checkout sherab-teak-dev
   ```
3. **Rebuild the image with the new code:**
   ```bash
   tutor images build openedx --no-cache # this will take some time
   ```
4. **Run the upgrade:**
   ```bash
   tutor dev upgrade --from=sumac
   ```
5. **Launch and Verify:**
   ```bash
   tutor dev launch # this will take some time
   ```
   *Now test that you can access the LMS and are not getting any errors.*

### 3. Phase 2: Upgrade to Ulmo (Tutor 21)
Once Teak is confirmed working, move to Ulmo.

1. **Install Tutor v21:**
   ```bash
   pip install --upgrade "tutor[full]>=21.0.0,<22.0.0"
   tutor config save
   ```
2. **Update the `edx-platform` code to the Ulmo branch:**
   ```bash
   git fetch origin sherab-ulmo1-dev
   git checkout sherab-ulmo1-dev
   ```
3. **Rebuild the image with the Ulmo code:**
   ```bash
   tutor images build openedx --no-cache # this will take some time
   ```
4. **Run the upgrade:**
   ```bash
   tutor dev upgrade --from=teak
   ```
5. **Launch and Verify:**
   ```bash
   tutor dev launch # this will take some time
   ```
   *Now test that you can access the LMS and are not getting any errors.*

<br>

---

## Post-Upgrade Cleanup & Testing

### 10. Test Thoroughly
Test essential functionalities:
- Registration/login (SSO)
- Course access
- Studio course creation
- MFE behavior
- Mobile app API compatibility

Monitor logs:
```bash
docker-compose logs -f
```

### 🧹 Optional: Clean Up
Remove unused plugins:
```bash
tutor plugins disable <plugin>
```
Remove old Docker images:
```bash
docker image prune
```

## 📚 Useful References
- **Tutor Changelog**: [GitHub Changelog](https://github.com/overhangio/tutor/blob/release/CHANGELOG.md)
- **Official Tutor Upgrade Docs**: [Tutor Install Guide](https://docs.tutor.edly.io/install.html#upgrading-to-a-new-open-edx-release)
- **Community Discussions**:
  - [Tutor Upgradation Discussion](https://discuss.openedx.org/t/tutor-upgradation/8360)
  - [Upgrading Tutor to Palm](https://discuss.openedx.org/t/upgrading-tutor-to-palm/10194/6)
  - [Open edX Upgrade Guide](https://discuss.openedx.org/t/how-to-update-open-edx-from-nutmeg-to-last-version/)
