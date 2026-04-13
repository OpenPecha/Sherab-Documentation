# 10. Troubleshooting Common Issues

Welcome to the troubleshooting guide. Open edX is a massive system, and you will likely see errors. Don't panic! Check here first.

---

## 1. `NoReverseMatch` in Django Admin
**Symptom:** You try to load a page in the Django Admin and see a yellow error screen complaining about `NoReverseMatch`.
**Cause:** Usually means a route (URL) is missing, often caused by a plugin failing to load or a missing feature flag.
**Fix:**
- Verify you enabled your custom plugins (`tutor plugins list`).
- Ensure `sherab-custom-plugin` is mounted and installed in the LMS container via `pip`.
- Restart the server completely `tutor dev stop` followed by `tutor dev start`.

## 2. Platform Extremely Slow or Crashing
**Symptom:** Docker containers keep shutting down randomly or your computer sounds like a jet engine.
**Cause:** Docker is out of RAM. Open edX requires massive amounts of memory.
**Fix:** Open Docker Desktop -> Settings -> Resources. Increase Memory to **at least 8GB** (ideally 12GB+ if you have it), and increase Swap to 2GB. Restart Docker.

## 3. MFE Changes Not Showing Up
**Symptom:** You edited React code in `frontend-app-learning`, but the browser still shows the old version.
**Cause:** Tutor might not be tracking the mount correctly or you didn't rebuild the MFE.
**Fix:**
- Run `tutor mounts list` and ensure the path is exactly correct.
- If it wasn't mounted when you started Tutor, run `tutor dev restart`.
- Try rebuilding the image: `tutor images build mfe` (this updates the container's understanding of dependencies).

## 4. `npm install` Errors on MFEs
**Symptom:** Installing dependencies throws a hundred red lines of text about missing Python versions, C++ compilers, or incompatible node-gyp.
**Cause:** Your version of Node.js is too new.
**Fix:**
- Open edX MFEs are notoriously picky about Node versions. Install `nvm`.
- Run `nvm install 16` and `nvm use 16` (or 18, depending on the Ulmo release). 
- Delete `node_modules` entirely, and run `npm install` again.

## 5. Website Doesn't Load on Phone Emulator
**Symptom:** Android emulator says "Server cannot be reached".
**Cause:** Emulator is trying to connect to its own localhost.
**Fix:** Ensure your `config.yaml` points to `10.0.2.2:8000` as specified in the Android guide, and Network Security cleartext is enabled.
