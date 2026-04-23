# 06. Partner-Organization Mapping Guide

## What is this?
In the Sherab mobile app, we want to allow users to filter courses by specific "Partners" or "Organizations". This guide explains how to link a Partner to an Organization in the backend so it properly displays in the mobile app UI.

---

## 1. Access the Django Admin
1. Make sure your local Tutor environment is running.
2. Go to `http://local.openedx.io:8000/admin`.
3. Log in with your staff/superuser account.

---

## 2. Navigate to Mappings
1. Scroll down until you find the **Course Partnerships** section.
2. Click on **Partner-Organization Mappings**.

---

## 3. Add a New Mapping
1. Click the **Add Partner-Organization Mapping** button in the top right.
2. Fill out the form:
   - **Partner**: Select an existing Partner from the dropdown.
   - **Organization**: Select the Organization to link with this partner.
   - **Show in mobile app**: ✅ Check this box! If it's unchecked, it won't appear on the mobile app.
   - **Display name (Optional)**: If you want the partner called something different in the app (e.g., "Sherab Academy" instead of just "Sherab"), type it here.

3. Click **Save**.
*(Note: A partner and an organization can only be mapped together once. If you try to map the same pair again, you will get an error).*

---

## How It Appears on Mobile
Once saved, the mobile app calls an API (`GET /api/partners/`). It will receive data looking like this:
```json
[
  {
    "partner_name": "Name of the partner",
    "logo": "https://yourdomain.com/.../partner_logo.png",
    "organization": "organization_short_name"
  }
]
```
The app uses this to dynamically build the filter UI!

---
**Next Step:** Learn about Feature Flags in [07-feature-flags.md](07-feature-flags.md).
