# 16. MFE Localization & Translation Setup

This guide describes how to set up MFE localization and pull translations for any MFE in Sherab.

> **Note:** Throughout this guide, replace `<mfe-name>` with the actual MFE name you are configuring, such as `account`, `learning`, `profile`, `discussions`, `authoring`, etc.

---

## Step 1: Enter the MFE Container and Confirm the Repository Path

```bash
tutor dev exec <mfe-name> sh -lc 'ls -la /openedx/app | head -n 30'
```

This confirms the MFE container is running and shows the repository structure.

---

## Step 2: Check if the Makefile Has a `pull_translations` Target

```bash
tutor dev exec <mfe-name> sh -lc '
  cd /openedx/app &&
  make -qp 2>/dev/null \
  | awk -F: "/^[a-zA-Z0-9][^$#\/\t=]*:([^=]|$)/ {print \$1}" \
  | sort -u \
  | grep -i -E "trans|i18n|atlas|pull|locale|msg"
'
```

If `pull_translations` appears in the output, you can proceed to Step 5. If not, you need to install Atlas CLI first.

---

## Step 3: Check if Atlas CLI is Installed

```bash
tutor dev exec <mfe-name> sh -lc 'which atlas || atlas --version || echo "atlas missing"'
```

If the output shows `atlas missing`, proceed to Step 4. Otherwise, skip to Step 5.

---

## Step 4: Install Atlas CLI (if Missing)

```bash
tutor dev exec <mfe-name> sh -lc '
  apt-get update &&
  apt-get install -y python3-pip git &&
  pip3 install --no-cache-dir openedx-atlas &&
  atlas --version
'
```

Once the install completes successfully, confirm the version is printed before continuing.

---

## Step 5: Pull Translations for the MFE

```bash
tutor dev exec <mfe-name> sh -lc '
  cd /openedx/app &&
  make ATLAS_OPTIONS="--repository=OpenPecha/openedx-translations --revision=sherab-dev" pull_translations
'
```

This pulls the Sherab-specific translations from the `OpenPecha/openedx-translations` repository on the `sherab-dev` branch.

---

## Common MFE Names Reference

| MFE Name | Description |
|---|---|
| `account` | Account settings MFE |
| `learning` | Course learning experience MFE |
| `profile` | User profile MFE |
| `discussions` | Course discussions MFE |
| `authoring` | Course authoring MFE |
| `authn` | Authentication MFE |
| `learner-dashboard` | Learner dashboard MFE |

---

**Next Step:** Head back to [README.md](README.md) for an overview, or review [11-advanced-plugins-and-translations.md](11-advanced-plugins-and-translations.md) for broader translation management.
