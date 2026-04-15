# 13. S3 Plugin Setup

This plugin allows Sherab (WebBuddhist Course) to be configured with custom S3 settings for file storage, profile images, and partner logos.

---

## 1. Install the Plugin

**For staging:**
```bash
pip install git+https://github.com/OpenPecha/tutor-contrib-s3.git@sherab-dev
```

**For production:**
```bash
pip install git+https://github.com/OpenPecha/tutor-contrib-s3.git@sherab-prod
```

---

## 2. Enable the Plugin

```bash
tutor plugins enable s3
```

---

## 3. Add the Tutor Plugin Patch

Create a plugin file and add the following content:

```python
from tutor import hooks

hooks.Filters.ENV_PATCHES.add_item(
    (
        "openedx-lms-development-settings",
        """
# Profile image S3 backend configuration (from plugin)
# Ensure base_url exists for DEBUG profile-image static() URLs
try:
    PROFILE_IMAGE_BACKEND["options"]["base_url"]
except KeyError:
    # Match the original default behavior from lms.envs.common
    PROFILE_IMAGE_BACKEND["options"]["base_url"] = os.path.join(MEDIA_URL, "profile-images/")
""",
    )
)
```

Enable the plugin after creating it:

```bash
tutor plugins enable <your-plugin-file-name>
```

---

## 4. Add Configuration to the Tutor Root

Add the following key-value pairs to your Tutor root configuration (`config.yml`):

```yaml
OPENEDX_AWS_ACCESS_KEY: <your-aws-access-key>
OPENEDX_AWS_SECRET_ACCESS_KEY: <your-aws-secret-access-key>
S3_ADDRESSING_STYLE: virtual
S3_FILE_UPLOAD_BUCKET: <private-assets-bucket>
S3_PARTNER_LOGO_BUCKET: <public-assets-bucket>
S3_PARTNER_LOGO_CUSTOM_DOMAIN: <public-assets-domain>
S3_PROFILE_IMAGE_BUCKET: <public-assets-bucket>
S3_PROFILE_IMAGE_CUSTOM_DOMAIN: <public-assets-domain>
S3_REGION: <aws-region>
S3_STORAGE_BUCKET: <private-assets-bucket>
S3_USE_SSL: true
```

> **Note:** Replace all `<placeholder>` values with your actual AWS credentials and S3 bucket names.

---

## 5. Rebuild the Tutor Image and Restart the Services

```bash
tutor images build openedx --no-cache
tutor dev stop
tutor dev start -d
```

For production, replace `tutor dev` with `tutor local`:

```bash
tutor images build openedx --no-cache
tutor local stop
tutor local start -d
```

---

**Next Step:** Head back to [README.md](README.md) for an overview, or review [12-custom-ora-component-setup.md](12-custom-ora-component-setup.md) for ORA component configuration.
