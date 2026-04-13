# 08. Transifex Setup and Management

## What is Transifex?
Open edX supports many languages out of the box. Translating terms (like changing "Submit" to "ཞུགས་པ།" in Tibetan) is managed by a third-party service called **Transifex**.

All translation files (mostly `.po` and `.json` files) are tracked in a repository.

---

## How Sherab Handles Translations
We use **GitHub Integration** with Transifex. This is the easiest and most automated way.
- When someone translates a string on the Transifex website, it automatically opens a Pull Request to our GitHub repository.
- When we merge that Pull Request, the translation becomes live on our platform!

### 📚 Further Reading for Advanced Setup:
If you need to manually fix translation keys or generate new tokens, reference our deep-dive resources:
- [Transifex Integration with GitHub](./docs-resources/transifex/transifex-github-integration.md)
- [Generating a Transifex API Token](./docs-resources/transifex/transifex-token-generation.md)
- [User Roles and Access in Transifex](./docs-resources/transifex/user-roles-access-for-transifex.md)
- [Fixing Transifex Resource Names for Open edX](./docs-resources/transifex/fix-transifex-resource-names-for-openedx.md)

---
**Next Step:** See the server migration steps in [09-tutor-migration-production.md](09-tutor-migration-production.md).
