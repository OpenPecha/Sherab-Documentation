# 03. Mounting Frontend React Components (MFE) in Tutor

## What is an MFE?
MFE stands for **Micro Frontend**. Instead of having one massive frontend layout, Open edX breaks down the UI into smaller, independent React applications (like `frontend-app-learning` for the course player, or `frontend-app-account` for user settings). 

If you need to change how the platform looks or behaves for a specific feature, you will "mount" the MFE locally. This means telling the Docker container to use the code on your computer instead of its internal code.

---

## Prerequisites
You must have **Node.js** and **npm** installed (see [00-getting-started-prerequisites.md](00-getting-started-prerequisites.md)).

---

## 1. Clone the Target React Component
First, we need to download the React code from the OpenPecha organization on GitHub. 
Clone the mfes inside the tutor root directory
```bash
   cd $(tutor config printroot)
```
*Example: Let's assume we want to modify the Learning app.*

```bash
git clone https://github.com/OpenPecha/frontend-app-learning.git
```

## 2. Create a Feature Branch
We never work directly on the `main` branch. Create a feature branch for your specific task:

```bash
cd frontend-app-learning
git checkout -b feature/my-custom-feature
```

## 3. Install Node Dependencies
Before modifying code or running it, you must install all the required Javascript packages.

```bash
npm install
```
*(If this fails, ensure you are using Node versions 16 or 18. Newer versions of Node often break MFE builds in Open edX! Use `nvm use 18` and try again.)*

## 4. Push Your Branch to GitHub
It is good practice to push your branch up so your work is backed up remotely.

```bash
# Wait until you actually make code changes before running this:
git add .
git commit -m "Initial commit for custom feature"
git push origin feature/my-custom-feature
```

## 5. Mount the Component in Tutor
Now we tell Tutor to use your local folder.

For most MFEs (like `frontend-app-learning`), you should clone them into your Tutor root directory, which you can locate with `tutor config printroot`.

1. Navigate to the Tutor root:
   ```bash
   cd $(tutor config printroot)
   ```
2. Mount it using `./`:
   ```bash
   tutor mounts add ./frontend-app-learning
   ```

> [!WARNING]
> **Exception for `frontend-app-profile` (macOS Users)**
> Your default Tutor root on macOS is usually inside `~/Library/Application Support/tutor`. The space in "Application Support" causes `pip` and `npm` errors *specifically* for the Profile MFE. 
> 
> If you are mounting `frontend-app-profile`, **do not move it into the Tutor root**. Keep it in a workspace directory without spaces (e.g., `~/sherab-mfe/`) and mount it using the absolute path:
> ```bash
> # Do this ONLY for frontend-app-profile!
> tutor mounts add ~/sherab-mfe/frontend-app-profile
> ```

4. Verify it was successfully mounted:
   ```bash
   tutor mounts list
   ```

## 6. Rebuild the MFE Image
Because we changed which code the frontend uses, we must rebuild the Docker image specifically for MFEs.

```bash
tutor images build mfe --no-cache
```

## 7. Restart Tutor
Apply everything by restarting your development server.

```bash
tutor dev restart
```

Now, any changes you make in `frontend-app-learning/src/` on your computer will show up live on your local Sherab instance!

---
**Next Step:** E-Commerce configuration is next in [04-sherab-ecommerce-setup.md](04-sherab-ecommerce-setup.md).
