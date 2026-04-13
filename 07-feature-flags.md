# 07. Open edX Feature Flags

## What are Feature Flags?
Open edX has hundreds of features. Instead of deleting code, features are simply toggled on `True` or off `False` using configurations called "Feature Flags". 

In our project, we set these flags in the `configuration_plugin` you created during setup. Here is a dictionary of what each flag actually does.

---

## LMS Flags (The Learner Facing Site)

- `CUSTOM_COURSES_EDX` (CCX): Allows instructors to create custom sub-sections of a course for specific groups.
- `ENABLE_CHANGE_USER_PASSWORD_ADMIN`: Staff can change users' passwords in the Django admin directly.
- `ENABLE_SPECIAL_EXAMS`: Enables proctored/timed exams.
- `MILESTONES_APP`: Allows prerequisites! A student must complete Course A to unlock Course B.
- `ENABLE_PREREQUISITE_COURSES`: Goes hand-in-hand with Milestones.
- `ENABLE_DISCUSSION_HOME_PANEL`: Adds an announcements panel to the student discussion board.
- `ENABLE_DISCUSSION_EMAIL_DIGEST`: Sends emails to students summarizing forum activity.
- `ENABLE_ACCOUNT_DELETION`: We set this to **False**. Users cannot delete their accounts from the UI.
- `CUSTOM_CERTIFICATE_TEMPLATES_ENABLED`: Allows us to design our own HTML certificates instead of the default edX ones.
- `SHOW_HEADER_LANGUAGE_SELECTOR`: Puts the language dropdown at the top navigation bar.
- `SHOW_FOOTER_LANGUAGE_SELECTOR`: We set this to **False** to remove it from the bottom.
- `ORGANIZATIONS_APP`: Groups courses under distinct organizations (crucial for our mobile app filter).
- `ENABLE_COURSE_DISCOVERY`: Allows unauthenticated (logged out) users to search the course catalog.
- `ALWAYS_REDIRECT_HOMEPAGE_TO_DASHBOARD_FOR_AUTHENTICATED_USER`: We set this to **False**. Logged in users can still visit the marketing homepage.
- `ENABLE_COMBINED_LOGIN_REGISTRATION`: Uses a modern single page for login/signup.
- `ENABLE_THIRD_PARTY_AUTH`: Allows Google, Facebook, Apple login.
- `ENABLE_EXTENDED_COURSE_DETAILS`: Unlocks extra metadata fields in Studio.

---

## CMS Flags (Studio - The Authoring Site)
- `ENABLE_SPECIAL_EXAMS`: Lets course authors configure those timed/proctored exams in Studio.
- `MILESTONES_APP` / `ENABLE_PREREQUISITE_COURSES`: Lets authors set the prerequisites.
- `ALLOW_PUBLIC_ACCOUNT_CREATION`: We set this to **False**. Random people cannot sign up to be Course Authors. An admin must create their account.

---
**Next Step:** See [08-transifex-setup-and-management.md](08-transifex-setup-and-management.md) to learn how translations are handled.
