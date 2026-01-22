
# Mounting Frontend React Components in Tutor

## 1. Clone the Required React Component

Go to the OpenPecha and clone the MFE repository you want to customise, for example:
- `frontend-app-learning`
- `frontend-app-account`
- `frontend-app-profile`

## 2. Create a Feature Branch

```bash
cd frontend-app-learning
git checkout -b feature/my-custom-feature
```

## 3. Push Your Changes to GitHub

```bash
git add .
git commit -m "Add custom feature to learning MFE"
git push origin feature/my-custom-feature
```

## 4. Clone the Feature Branch into Tutor Root

Navigate to your Tutor root directory (e.g., `~/Library/Application Support/tutor`) and clone the MFE:

```bash
cd /path/to/your/tutor/root
git clone -b feature/my-custom-feature https://github.com/your-username/frontend-app-learning.git
```

## 5. Mount the Component in Tutor

```bash
tutor mounts add ./frontend-app-learning
```

check if its mounted by running:
```bash
tutor mounts list
```
Install the dependencies in the MFE:

```bash
cd frontend-app-learning
```
```bash
npm install
```

## 6. Rebuild the MFE Image (Without Cache)

```bash
tutor images build mfe --no-cache
```

## 7. Restart Tutor (If Required)

```bash
tutor dev restart
```

