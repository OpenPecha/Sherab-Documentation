# 17. Database & Media Restoration Guide

This document describes how to restore missing partner logos and course images in your local Open edX environment after a Docker volume wipe (e.g. accidental `docker system prune`).

> [!NOTE]
> This guide covers the **local development** environment (`tutor dev`). A separate section at the bottom covers what to change for **production** (`tutor local`).

---

## 🩺 Understanding the Problem

When Docker volumes are pruned, the MySQL database is wiped and re-initialised from scratch by Tutor. Two things break as a result:

| What's broken | Why |
|---|---|
| **Partner logos not showing** | `organizations_organization.logo` is empty — the file paths were in MySQL, which was wiped. The actual image files survive on disk. |
| **Course images not showing** | `course_overviews_courseoverviewimageset` table is empty — these processed thumbnail records were also in MySQL. |

The physical image files are **not lost**. They live in a host-mounted directory and survive the prune:
```
~/Library/Application Support/tutor/data/openedx-media/partner/
```

---

## ✅ Pre-Requisites

1. Your Tutor environment is running:
   ```bash
   source ~/openedx/tutor-venv/bin/activate
   tutor dev start -d
   ```
2. Confirm MySQL is healthy:
   ```bash
   docker ps | grep mysql
   ```
3. Confirm the partner logo files exist on disk:
   ```bash
   ls ~/Library/Application\ Support/tutor/data/openedx-media/partner/
   ```
   You should see files like `BDRC_Logo.png`, `Esukhia_logo.png`, etc.

---

## Part 1: Restore Partner Logos

### Step 1 — Open a MySQL session

```bash
docker exec -it tutor_dev-mysql-1 mysql -u root -pfzTIpoTg openedx
```

You are now inside the MySQL shell. Your prompt will look like `mysql>`.

### Step 2 — Verify logos are currently empty

Run this to confirm the problem before making any changes:
```sql
SELECT id, short_name, logo FROM organizations_organization;
```
All rows in the `logo` column should be empty — that confirms the issue.

### Step 3 — Update the logo paths

Run these `UPDATE` statements one by one. They restore the logo file path for each organisation that has a known logo on disk.

> [!IMPORTANT]
> The path format must be exactly `partner/filename.ext` — this is the relative path Django uses to serve files from the `openedx-media` storage root.

```sql
-- Organisations with known logo files on disk
UPDATE organizations_organization SET logo='partner/BDRC_Logo.png'          WHERE short_name='BDRC';
UPDATE organizations_organization SET logo='partner/Esukhia_logo.png'       WHERE short_name='Esukhia';
UPDATE organizations_organization SET logo='partner/Kumarajiva_logo.png'    WHERE short_name='Kumarajiva';
UPDATE organizations_organization SET logo='partner/sherab_logo.jpeg'       WHERE short_name='Sherab';
UPDATE organizations_organization SET logo='partner/Sarah_college_logo.png' WHERE short_name='Sarah';
UPDATE organizations_organization SET logo='partner/Monlam_logo.png'        WHERE short_name='Jampa';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungKajangMalaysia';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungSangyelingFrance';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungFrance';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungChangchubDargyelingUK';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungSamphelCholingSpain';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungHongKong';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungDharmaChakra';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungEurope';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungYesheCholing';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungYesheGatshalFinland';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungFinland';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungChokorLingIreland';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungSherablingTaiwan';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungYesheCholingSlovenia';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungThigsumChokyiGhatsalAustralia';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='PalpungLungtokChoelingUS';
UPDATE organizations_organization SET logo='partner/Palpung_logo_g8hgck6.png' WHERE short_name='KarmalingBirminghamEngland';
```

> [!NOTE]
> `Khyentse_Foundation`, `Demo`, and `test` do not have logo files on disk. Leave them with an empty logo for now. You can upload logos for these later via the Django Admin if needed.

### Step 4 — Verify the update worked

```sql
SELECT id, short_name, logo FROM organizations_organization;
```
You should now see `partner/filename.ext` in the `logo` column for the updated rows.

### Step 5 — Exit MySQL

```sql
exit
```

---

## Part 2: Regenerate Course Image Sets

The `course_overviews_courseoverviewimageset` table stores processed thumbnail metadata. It is currently empty (0 rows). Django has a built-in management command to regenerate it from the existing course data.

### Step 1 — Run the management command

```bash
docker exec tutor_dev-lms-1 bash -c \
  "./manage.py lms --settings=tutor.development generate_course_overview --all-courses"
```

This will scan all 90 courses in `course_overviews_courseoverview`, read their `course_image_url`, and write the corresponding imageset records back into the database.

> ⏳ This may take a few minutes to complete. Wait for the command to finish before proceeding.

### Step 2 — Verify the table is populated

```bash
docker exec tutor_dev-mysql-1 mysql -u root -pfzTIpoTg openedx \
  -e "SELECT COUNT(*) FROM course_overviews_courseoverviewimageset;" 2>/dev/null
```

The count should now be greater than 0 (ideally around 90).

---

## Part 3: Verify in Browser

1. Open the LMS: [http://local.openedx.io:8000](http://local.openedx.io:8000)
2. Browse the course catalogue — course thumbnails should now appear.
3. Navigate to any page that shows partner/organisation logos — they should now load.

If images still appear broken (showing broken image icon):
- Hard refresh your browser (`Cmd + Shift + R`)
- Check the browser console for the exact URL it is trying to load
- Confirm that URL resolves to a file in `openedx-media/partner/` on disk

---

## 🏭 Production Notes

> [!WARNING]
> Before touching any data in production, **always take a backup first**:
> ```bash
> tutor local do mysql-backup
> ```

The same process applies in production with two differences:

| Step | Local (`tutor dev`) | Production (`tutor local`) |
|---|---|---|
| Start environment | `tutor dev start -d` | `tutor local start -d` |
| Management command | `docker exec tutor_dev-lms-1 bash -c "..."` | `tutor local run lms bash -c "..."` |
| Media storage | Local filesystem `openedx-media/` | S3 bucket (logo paths are still `partner/filename.ext` as S3 keys) |
| MySQL container | `tutor_dev-mysql-1` | `tutor_local-mysql-1` |

The SQL `UPDATE` statements are **identical** in both environments.

---

**Previous:** [16. Development, Testing & Deployment Workflow](16-development-testing-deployment-workflow.md)
