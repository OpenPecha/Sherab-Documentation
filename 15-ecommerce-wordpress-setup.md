# 15. Ecommerce WordPress Setup

This guide describes how to set up ecommerce functionality for Sherab using WordPress and Tutor.

---

## Development Setup

### Step 1: Install the WordPress Plugin

Install the Tutor WordPress plugin:

```bash
pip install -U tutor-contrib-wordpress
```

Enable the WordPress plugin:

```bash
tutor plugins enable wordpress
```

### Step 2: Launch Tutor in Development Mode

```bash
tutor dev launch
```

### Step 3: Configure the Development Domain

Add the following to your Tutor development configuration (`config.yml`):

```yaml
WORDPRESS_HOST: store.local.overhang.io
```

Then save the configuration:

```bash
tutor config save
```

Access the development WordPress site at: [http://store.local.overhang.io](http://store.local.overhang.io)

---

## Production Setup

### Step 1: Install the WordPress Plugin

Install the Tutor WordPress plugin:

```bash
pip install -U tutor-contrib-wordpress
```

Enable the WordPress plugin:

```bash
tutor plugins enable wordpress
```

Launch Tutor with the plugin:

```bash
tutor local launch
```

### Step 2: Domain Registration

Register the store domain on **Cloudflare**:

```
store.sherab.org
```

Add the URL to the Tutor root configuration (`config.yml`):

```yaml
WORDPRESS_HOST: store.sherab.org
```

Then save and apply the configuration:

```bash
tutor config save
tutor local launch
```

### Step 3: Launch and Set Up WordPress

1. Install and activate the **OpenEdX commerce plugins** in the WordPress site.
2. Configure the necessary WordPress settings for ecommerce integration.

### Step 4: Create an OAuth Application in Sherab Admin

1. Go to your Sherab Admin panel.
2. Navigate to **OAuth Applications** and create a new application with the following settings:
   - **Client type:** Confidential
   - **Authorization grant type:** Client credentials
3. Copy the **Client ID** and **Client Secret**.
4. In your WordPress site, add the Client ID and Secret, then generate the **JWT tokens** for authentication.

---

**Next Step:** Head back to [README.md](README.md) for an overview, or continue with [16-mfe-localization-translations.md](16-mfe-localization-translations.md).
