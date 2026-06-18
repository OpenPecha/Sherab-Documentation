# 17. MFE Header Versioning (Post-Ulmo)

## What Is This About?

We are maintaining two different versions of the header component for our Micro-Frontends (MFEs). Different MFEs require specific versions, and using the wrong version will cause the MFE to either fail to build or display a broken header.

This guide documents which header version goes with which MFE.

---

## Header version for MFEs

Here is the exact mapping of which header version each MFE uses. **Pay close attention** — using the wrong version will break the build.

| MFE | Footer Branch | Header Branch |
| --- | ------------- | ------------- |
| **Profile** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Learning** | `wbc-ulmo1-stage` | `wbc-stage-v8.0.0` |
| **Communications** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Gradebook** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **ORA Grading** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Learner Dashboard** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |
| **Authoring** | `wbc-ulmo1-stage` | `wbc-stage-v8.0.0` |
| **Discussions** | `wbc-ulmo1-stage` | `wbc-stage-v6.6.0` |

> [!NOTE]
> **Communications, Discussions, Gradebook, and ORA Grading** do not have custom OpenPecha forks — they use the default Open edX MFE code.

> [!NOTE]
> If a new MFE is added to the platform in the future, check with the team lead which header version it needs.



---

## Troubleshooting

- **MFE header looks broken or misaligned?** Check the version mapping table above. You likely installed the wrong header version for that MFE.
- **Build fails with npm errors?** Make sure the branch names (`wbc-ulmo1-stage`, `wbc-stage-v6.6.0`, `wbc-stage-v8.0.0`) still exist in the GitHub repos. Your team lead may have renamed them.
- **Changes not showing up?** You **must** rebuild with `--no-cache`. Tutor caches Docker layers aggressively, so a regular rebuild may reuse stale layers.

---

**Next Step:** Head back to [README.md](README.md) for an overview of all documentation.
