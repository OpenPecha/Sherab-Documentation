# 17. MFE Header Versioning (Post-Ulmo)

## What Is This About?

We are maintaining two different versions of the header component for our Micro-Frontends (MFEs). Different MFEs require specific versions, and using the wrong version will cause the MFE to either fail to build or display a broken header.

This guide explains:
1. Which header version goes with which MFE.
2. How the Tutor plugin overrides the default versions automatically during Docker image builds.

> [!IMPORTANT]
> This is separate from the local development override described in [14-header-footer-brand-override.md](14-header-footer-brand-override.md). That guide is for **local development** (mounting repos). This guide is for **building Docker images** for staging and production.

---

## Header version for MFEs

Here is the exact mapping of which header version each MFE uses. **Pay close attention** — using the wrong version will break the build.

| MFE | Footer Branch | Header Branch |
| --- | ------------- | ------------- |
| **Profile** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Learning** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Communications** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Gradebook** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **ORA Grading** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Learner Dashboard** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Authoring** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Discussions** | `wbc-ulmo1-stage` | `wbc-stage-v8.0.0` |

> [!NOTE]
> **Communications, Discussions, Gradebook, and ORA Grading** do not have custom OpenPecha forks — they use the default Open edX MFE code. However, they still receive our custom header and footer via this plugin during the Docker image build.

> [!NOTE]
> If a new MFE is added to the platform in the future, check with the team lead which header version it needs before adding it to the plugin.



---

## How Does This Work Under the Hood?

When Tutor builds the Docker images for the Micro-Frontends, it normally pulls the standard Open edX header and footer packages.

To apply our custom branding and versioning, our Tutor plugin intervenes during this build process. Right after the default components are installed, the plugin instructs Docker to download our custom OpenPecha forks directly from GitHub and install them over the defaults.

By doing this dynamically during the build:
- The footer is always overridden with the `wbc-ulmo1-stage` branch for every MFE.
- The header is dynamically overridden with either the `wbc-stage-v6.6.0` or `wbc-stage-v8.0.0` branch, depending on which specific MFE is currently compiling.

---

## Troubleshooting

- **MFE header looks broken or misaligned?** Check the version mapping table above. You likely installed the wrong header version for that MFE.
- **Build fails with npm errors?** Make sure the branch names (`wbc-ulmo1-stage`, `wbc-stage-v6.6.0`, `wbc-stage-v8.0.0`) still exist in the GitHub repos. Your team lead may have renamed them.
- **Changes not showing up?** You **must** rebuild with `--no-cache`. Tutor caches Docker layers aggressively, so a regular rebuild may skip the npm install steps.

---

**Next Step:** Head back to [README.md](README.md) for an overview of all documentation.
