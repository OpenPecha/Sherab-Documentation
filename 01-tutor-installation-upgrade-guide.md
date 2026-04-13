# 01. Tutor Installation and Database Restoration Guide

In this step, we will install Tutor, which is the official tool used to manage and run an Open edX instance.

---

## 2. Setup a Python Virtual Environment
Always use a virtual environment! It keeps Python packages isolated from the rest of your system. 
```bash
# Create the environment
python3 -m venv tutor-env

# Activate it inside the virtual environment(you MUST do this every time you open a new terminal for this project)
source tutor-env/bin/activate
```
*(When activated, your terminal prompt should have `(tutor-env)` at the beginning.)*

---

## 3. Install Tutor
With your environment active, install Tutor version 16.1.8. 
```bash
pip install "tutor[full]==16.1.8"
```

---

## 4. First Launch (Development Mode)
We need to launch Tutor once so it generates the necessary configuration files and containers.
```bash
tutor dev launch
```
- This will take several minutes to download the massive Docker images. Grab a coffee!
- **What this does:** It spins up MySQL, MongoDB, Redis, and the actual Open edX (LMS + CMS) application locally on your machine.

---

## 5. Restore the Databases
We have pre-existing data (courses, users, etc.) that you need to load into your local setup. You will need the **SQL dump** and **MongoDB backup**. Ask the Tech Lead if you don't have these.

### A. Restore MySQL
1. Open a new terminal tab (leave Tutor running).
2. Find the ID of your running MySQL container by listing all active containers:
   ```bash
   docker ps
   ```
3. Look for a container name ending in `mysql`. Copy its Container ID (a random string like `a1b2c3d4e5f6`).
4. Run the restore command. Replace `<sql_container_id>`, `'your_sql_root_password'`, and the path to your dump file:
   ```bash
   cat path/to/sql/dump.sql | docker exec -i <sql_container_id> /usr/bin/mysql -u root --password='your_sql_root_password' mysql
   ```

### B. Restore MongoDB
1. Copy your MongoDB backup folder into Tutor's internal data directory:
   ```bash
   # Use the following to find where Tutor stores its data:
   echo $(tutor config printroot)
   
   # Copy it into that directory under data/mongodb
   cp -r /path/to/your/db_backup.mongodb $(tutor config printroot)/data/mongodb/
   ```
2. Open a direct shell inside pointing to the active MongoDB container:
   ```bash
   tutor dev exec mongodb bash
   ```
3. Inside that shell, run the restore tool:
   ```bash
   mongorestore /data/db/db_name.mongodb/
   ```
4. Type `exit` to leave the shell once it completes.

---

## 6. Update Configuration
We need to tell Tutor the database password. 
1. Open the file `config.yml` located in your Tutor root directory (find the folder by running `tutor config printroot`).
2. Open it in a text editor (like VS Code) and add this line:
   ```yaml
   MYSQL_ROOT_PASSWORD: your_sql_root_password
   ```

---

## 7. Rebuild the Main Image
Since configurations changed, we rebuild the Open edX docker image. This skips caching (--no-cache) to ensure fresh settings.
```bash
tutor images build openedx --no-cache
```

---

## 8. Restart and Verify
Restart everything so your changes apply.
```bash
tutor dev restart
```
- Go to `http://local.openedx.io:8000` in your web browser. You should see the standard Open edX homepage!
- Once verified, stop it so we can upgrade.
```bash
tutor dev stop
```

---

## 9. Upgrade Tutor (Gradual Version Upgrades)
Open edX requires you to upgrade version by version. We must go from Palm (16) -> Quince (17) -> Redwood (18) -> Sumac (19).

*(Make sure your python virtual environment `tutor-env` is still activated!)*

### Step 1: Upgrade to Quince (Tutor 17)
```bash
pip install --upgrade "tutor[full]>=17.0.0,<18.0.0"
tutor config save
tutor dev upgrade --from=palm
```

### Step 2: Upgrade to Redwood (Tutor 18)
```bash
pip install --upgrade "tutor[full]>=18.0.0,<19.0.0"
tutor config save
tutor dev upgrade --from=quince
```

### Step 3: Upgrade to Sumac (Tutor 19) - Current Targeted Server
```bash
pip install --upgrade "tutor[full]>=19.0.0,<20.0.0"
tutor config save
tutor dev upgrade --from=redwood
```

---

## 10. Final Launch
You are fully upgraded! Launch it!
```bash
tutor dev launch
```

> [!WARNING]
> You may see a warning: "It is recommended to upgrade your character set and collation of the MySQL database after upgrading to Sumac." You can safely ignore this for local development unless actively working on database migrations.

---
**Next Step:** Your base software is running, but it's not our project code yet. Move on to [02-tutor-setup-customization-guide.md](02-tutor-setup-customization-guide.md) to apply the Sherab custom theme and plugins.
