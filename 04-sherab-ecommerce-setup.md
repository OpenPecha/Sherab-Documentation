# 04. Complete Guide to Setting Up E-commerce for Sherab

## What Is This Integration?
We use a **WordPress/WooCommerce** setup to act as the primary storefront for the Sherab project. 
- **WordPress** is the website catalog where users browse.
- **WooCommerce** handles the shopping cart and payments.
- When a user buys a course, the system uses OAuth2 (a secure way APIs talk to each other) to automatically enroll them into the actual course on **Open edX**.

## Prerequisites
Before you begin, you must have your Tutor development server running (`tutor dev start` or `tutor dev launch`).

---

## 1. Install the WordPress Plugin into Tutor
You don't need to manually configure WordPress or database migrations. Tutor handles this via a plugin!

Open your terminal and run:
```bash
pip install -U tutor-contrib-wordpress
tutor plugins enable wordpress
tutor dev launch
```
This automatically sets up:
1. Complete WordPress installation
2. WooCommerce e-commerce plugin
3. Sherab Commerce plugin (the bridge between Open edX and WordPress)

---

## 2. Generate Plugin Configurations
To connect the two systems, we need specific configuration URLs. Tutor will generate these for you.

Run this command to print the required data:
```bash
tutor wordpress config printroot    
```
*Keep this terminal window open or copy the text, you will need it shortly.*

---

## Phase 1: Create OAuth2 Application (In Sherab / Open edX)
We must create a secure "key" in Open edX so WordPress is allowed to talk to it.

1. In your browser, go to the Django Admin panel: `http://local.openedx.io:8000/admin`
2. Log in using a staff superuser account.
3. Scroll down to **OAuth2 Provider** -> **Applications** and click **Add Application**.
4. Fill out the form exactly as follows:
   - **User:** Select your staff user account
   - **Client type:** Confidential
   - **Authorization grant type:** Client credentials
   - **Name:** "WordPress Commerce"
5. Click **Save**.
6. The system will give you a **Client ID** and a **Client Secret**. **Copy and save these immediately.**

---

## Phase 2: Configure WordPress
Now we give WordPress the key we just generated.

1. Access WordPress by going to `http://local.openedx.io:8080/wp-admin` in your browser.
2. Look at the left sidebar menu. Go to **WooCommerce** -> **Settings**.
3. Click on the **Integration** tab at the top.
4. Click on **OpenEdX**.
5. You will see a form. Enter the following:
   - **LMS Domain:** `http://local.openedx.io:8000`
   - **Client ID:** (Paste the ID from Phase 1)
   - **Client Secret:** (Paste the Secret from Phase 1)
6. Scroll down and click **Test Connection (Generate JWT Token)**. If it turns green, you are successfully connected!

---

## Phase 3: Creating a Course Product
Now let's create something to sell!

1. In the WordPress admin sidebar, go to **Products** -> **Add New**.
2. Give your product a name, price, description, and images.
3. Look for the checkbox labelled **OpenEdX Course** and tick it.
4. Two new fields will appear:
   - **Course ID:** The exact ID of the course in Studio (e.g., `course-v1:MySchool+Math101+2024`)
   - **Course Mode:** The type of enrollment (see table below).
5. Click **Publish** on the right side. 

---

### Understanding Course Modes

| Mode | Description | Certificate | Payment Required |
| ---- | ----------- | ----------- | ---------------- |
| `audit` | Free access, no certificate | No | No |
| `honor` | Free access with certificate | Yes | No |
| `verified` | Paid with verified certificate | Yes | Yes |
| `no-id-professional` | Professional, no ID verification | Yes | Yes |

---

## How It Works (The Student Experience)
1. Student browses courses in WordPress.
2. They purchase via WooCommerce handling the credit card transaction.
3. Our plugin detects the completed order.
4. It uses the OAuth API key to instantly tell Open edX to enroll the student's email address in the mapped `Course ID`.
5. The student logs into Open edX and their course is waiting for them!

---

## Common Issues / Troubleshooting

- **JWT Token Fails:** Check that you copied the domain, Client ID, and Client Secret accurately. Ensure the User you selected for the Application was a Staff user.
- **Enrollments Not Working:** Confirm that the `Course ID` in the WordPress Product perfectly matches the one in Open edX Studio. Ensure the student's email address on WordPress exactly matches their Open edX account.

---
**Next Step:** Are you developing the mobile app? Proceed to [05-android-app-development-environment-setup.md](05-android-app-development-environment-setup.md).
