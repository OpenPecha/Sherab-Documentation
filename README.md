# Sherab Project Documentation

Welcome to the **Sherab Project**. This repository contains the central documentation, technical setup instructions, and operational procedures for our customized Open edX deployment.

---

## 📚 Start Here!
If you are new to the team, please read these files in strict order. They are designed to hold your hand through the entire setup of our platform.

- 📘 [00. Getting Started: Global Prerequisites](00-getting-started-prerequisites.md)
- ⬆️ [01. Tutor Installation & Upgrade Guide](01-tutor-installation-upgrade-guide.md)
- ⚙️ [02. Tutor Setup & Customization Guide](02-tutor-setup-customization-guide.md)
- 🧩 [03. Mounting Frontend MFEs](03-mounting-frontend-mfe.md)
- 🛒 [04. Setting Up E-commerce for Sherab](04-sherab-ecommerce-setup.md)
- 📱 [05. Android App Development Environment Setup](05-android-app-development-environment-setup.md)

## 🧩 Advanced Features & Admin
Once you're set up, refer to these documents for specific administrative tasks and deployments:

- 🤝 [06. Partner-Organization Mapping](06-partners-organization-mapping-management.md)
- 🚩 [07. Open edX Feature Flags](07-feature-flags.md)
- 🌐 [08. Transifex Setup & Management](08-transifex-setup-and-management.md)
- 🔄 [09. Tutor Server Migration Guide](09-tutor-migration-production.md)
- 🚑 [10. Troubleshooting Common Issues](10-troubleshooting.md)
- 🚀 [11. Advanced Plugins & Translations](11-advanced-plugins-and-translations.md)

---

# 📦 Core Repositories

These are our core repositories maintained under the Sherab project by [OpenPecha](https://github.com/OpenPecha).

### 1. [Sherab Theme](https://github.com/OpenPecha/Sherab-theme)
A custom theme for Open edX tailored for the Sherab learning platform, including branding, styles, and logos.

### 2. [edx-platform](https://github.com/OpenPecha/edx-platform.git)
Our custom fork of the Open edX core codebase.

### 3. [Sherab Custom Plugin](https://github.com/OpenPecha/sherab-custom-plugin)
A Django plugin extending core functionality with Sherab-specific backend logic and custom app APIs.

### 4. Micro-Frontends (MFEs)
- [Learner Dashboard](https://github.com/OpenPecha/frontend-app-learner-dashboard)
- [Frontend Component - Footer](https://github.com/OpenPecha/frontend-component-footer)
- [Frontend Component - Header](https://github.com/OpenPecha/frontend-component-header)

We also utilize the default enabled MFEs from Open edX (Authn, Authoring, Account, Communications, Discussions, Gradebook, Learning, ORA Grading, Profile).

---

# 🤝 How to Contribute

We welcome contributions from the community! Here's how you can get involved:

### 🐞 Bug Reports

If you encounter a bug or unexpected behavior:

- Open an issue in the respective repository
- Describe the problem clearly and include steps to reproduce it if possible
- This helps us track and resolve issues efficiently

### 🛠 Code Contributions

Want to contribute code? Follow these steps:

1. **Create a feature branch** off the `sherab-dev` branch in the respective repository
2. Make your changes and commit them.
3. **Open a Pull Request (PR)** to the `sherab-dev` branch.
4. One of the Sherab developers will review and test your contribution.
5. After approval, the changes will be merged into production.

> 🔒 **Note:** All main branches are protected and require code review before merging.

### 💬 Join the Conversation

For discussions, questions, or collaboration:

- **[Join our Discord community](https://discord.gg/8ENNuW95wx)**
- **[Visit the Sherab Community Forum](https://forum.openpecha.org/)**

We're excited to work together and improve the Sherab platform with your help!
