# 05. Android App Development Environment Setup for Open edX

This guide walks you through setting up Android Studio and configuring a local development environment to run the Open edX Android app. The mobile app lets users consume Sherab courses directly on their phones.

---

## 1. Install Android Studio
Android Studio is the official program used to write and run Android apps.

### Step 1: Download
Visit the [official Android Studio website](https://developer.android.com/studio) and download the installer for your OS.

### Step 2: Install
- **Windows/macOS:** Run the downloaded installer.
- **Linux:**
  ```bash
  tar -xzf android-studio-*.tar.gz
  sudo mv android-studio /opt/
  cd /opt/android-studio/bin
  ./studio.sh
  ```
- Follow the **Setup Wizard** to install required SDK components. Accept the default options. 

*(Note for Linux 64-bit users only: You may need extra libraries: `sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 libbz2-1.0:i386`)*

---

## 2. Enable Emulator Acceleration (Crucial for Speed!)
If you skip this, running the emulated Android phone on your computer will be incredibly slow.

### Step 1: Check Virtualization Support
You must have Virtualization enabled on your actual computer's motherboard. 
- Reboot your computer and enter BIOS/UEFI (press `DEL`, `F2`, or `Esc` during boot).
- Find the virtualization setting and enable it:
  - **Intel CPUs:** Look for `Intel VT-x`
  - **AMD CPUs:** Look for `AMD-V` or `SVM`
- Save changes (usually `F10`) and boot back to your OS.

### Step 2: OS-Specific Acceleration Setup

#### macOS & Windows
- Open Android Studio. Go to the SDK Manager (Tools -> SDK Manager).
- Install **Intel HAXM** (for Intel CPUs) or enable **hardware acceleration for ARM images**.
*(Android Studio will usually prompt you if HAXM isn't available).*

#### Linux
1. Check if KVM is supported:
   ```bash
   sudo apt-get install cpu-checker
   sudo kvm-ok
   ```
   *Expected output: `KVM acceleration can be used`*
2. Install KVM dependencies:
   ```bash
   sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
   ```

---

## 3. Local Development Environment Setup

> [!CAUTION]
> **Important Note**: The following configurations are intended *only* for local development. We are intentionally relaxing security rules to allow an emulator to talk to your local backend. **Never commit these changes to GitHub.**

### Step 1: Configure Development Settings
1. Open the project in **Android Studio**.
2. Locate the development configuration file at: `default_config/dev/config.yaml`
3. Update the following keys:
   ```yaml
   API_HOST_URL: 'http://10.0.2.2:8000'
   OAUTH_CLIENT_ID: '<your-oauth-client-id>'
   ```

**Wait, why 10.0.2.2?** 
When running the app in an Android emulator, it has its own private network. If you type `localhost` or `127.0.0.1`, it will try to find a server inside the emulator phone itself! `10.0.2.2` is a special alias that perfectly bridges out of the emulator and connects to your computer's localhost.

### Step 2: Allow HTTP (Cleartext) Traffic
Modern apps block HTTP (they demand secure HTTPS). Our local environment runs on HTTP, so we have to disable this block temporarily.

1. Open `app/src/main/AndroidManifest.xml`
2. Add the `networkSecurityConfig` attribute inside the `<application>` tag:
   ```xml
   android:networkSecurityConfig="@xml/network_security_config"
   ```
3. Create a new XML file exactly here: `app/src/main/res/xml/network_security_config.xml`
4. Paste exactly this into the file:
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
      <domain-config cleartextTrafficPermitted="true">
          <domain includeSubdomains="true">10.0.2.2</domain>
          <domain includeSubdomains="true">192.168.1.6</domain>
      </domain-config>
   </network-security-config>
   ```

## ✅ You're All Set!
Click the green "Play" button in Android Studio to build and run the Open edX Android app locally. It should successfully talk to your Tutor backend.

---
**Next Step:** See [06-parters-organization-mapping-management.md](06-partners-organization-mapping-management.md) to manage in-app mappings.
