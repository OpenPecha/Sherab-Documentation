# 12. Custom ORA Component Setup

This guide describes how to set up custom ORA (Open Response Assessment) components for Sherab (WebBuddhist Course).

There are two approaches depending on the environment:

- **Local development:** mount the local `ora2` repository.
- **Staging and production:** create a Tutor plugin instead of using mounts.

---

## 1. Local Development Setup

### Step 1: Clone the ORA Components Repository

```bash
git clone https://github.com/OpenPecha/edx-ora2.git
```

### Step 2: Mount the ORA Components Repository

Go to the `ora2` directory and run:

```bash
tutor mounts add ./ora2
```

### Step 3: Save the Tutor Configuration

```bash
tutor config save
```

### Step 4: Rebuild and Restart

Rebuild the images and restart the service:

```bash
tutor images build openedx --no-cache
tutor dev stop
tutor dev start -d
```

---

## 2. Staging and Production Setup

For staging and production, **do not use mounts**. Instead, create a Tutor plugin.

### Step 1: Create the Plugin File

Create a file named `sherab_ora.py` and add the following content:

```python
from tutor import hooks

hooks.Filters.ENV_PATCHES.add_items([
    (
        "openedx-dockerfile-post-python-requirements",
        """
RUN pip uninstall -y ora2
RUN pip install --no-cache-dir git+https://github.com/OpenPecha/edx-ora2.git@sherab-dev
"""
    ),
    (
        "openedx-dev-dockerfile-post-python-requirements",
        """
RUN pip uninstall -y ora2
RUN pip install --no-cache-dir git+https://github.com/OpenPecha/edx-ora2.git@sherab-dev
"""
    )
])
```

> **Note:** Update the branch name (e.g. `sherab-dev`) to match the target environment — use `sherab-staging` for staging or `sherab-prod` for production.

### Step 2: Enable the Plugin

```bash
tutor plugins enable sherab_ora
```

### Step 3: Rebuild and Restart

```bash
tutor images build openedx --no-cache
tutor local stop
tutor local start -d
```

---

**Next Step:** Head back to [11-advanced-plugins-and-translations.md](11-advanced-plugins-and-translations.md) or continue with [13-s3-plugin-setup.md](13-s3-plugin-setup.md).
