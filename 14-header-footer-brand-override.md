# 14. Header, Footer & Brand Component Overrides

This guide describes how to override the header, footer, and brand components with custom implementations for Sherab (WebBuddhist Course).

---

## 1. Clone the Required Repositories

Clone the header and footer components **inside each MFE directory** where you want to override them.

**Header component:**
```bash
git clone https://github.com/OpenPecha/frontend-component-header.git
```

**Footer component:**
```bash
git clone https://github.com/openedx/frontend-component-footer.git
```

---

## 2. Clone the Brand Component

Clone the brand component in the **root directory** of your project:

```bash
git clone https://github.com/OpenPecha/brand-openedx.git
```

---

## 3. Mount the Brand Component

Mount the brand component to all the MFEs you want to override. Update the path to match your local setup:

```bash
tutor mounts add "authn,account,learner-dashboard,learning,profile,discussions,authoring:/path/to/brand-openedx:/openedx/brand-openedx"
```

> **Note:** Replace `/path/to/brand-openedx` with the actual absolute path to your cloned `brand-openedx` directory.

---

## 4. Install Dependencies

Run the following command inside **each component directory** (header, footer, and brand):

```bash
npm install
```

---

## 5. Create a `module.config.js` File

Create a `module.config.js` file inside **each MFE directory** and add the following configuration:

```js
module.exports = {
  localModules: [
    // Brand overrides
    {
      moduleName: '@edx/brand',
      dir: '../brand-openedx',
    },

    // Footer overrides
    {
      moduleName: '@edx/frontend-component-footer/dist',
      dir: 'frontend-component-footer',
      dist: 'src',
    },
    {
      moduleName: '@edx/frontend-component-footer',
      dir: 'frontend-component-footer',
      dist: 'src',
    },

    // Header overrides
    {
      moduleName: '@edx/frontend-component-header/dist',
      dir: 'frontend-component-header',
      dist: 'src',
    },
    {
      moduleName: '@edx/frontend-component-header',
      dir: 'frontend-component-header',
      dist: 'src',
    },
  ],
};
```

---

## 6. Rebuild the MFEs

Rebuild all Micro-Frontend applications that have been modified:

```bash
tutor images build mfe --no-cache
```

---

## 7. Restart the Containers

```bash
tutor dev stop
tutor dev start -d
```

For production:

```bash
tutor local stop
tutor local start -d
```

---

## 8. Build and Deploy Changes

After making any further changes to the components, run:

```bash
npm run build
```

Then push the `dist` directory to your deployment target.

---

**Next Step:** Head back to [README.md](README.md) for an overview, or continue with [15. MFE Localization & Translation Setup](15-mfe-localization-translations.md).
