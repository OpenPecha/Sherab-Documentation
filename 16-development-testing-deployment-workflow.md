# 16. Development, Testing & Deployment Workflow

This guide explains how we develop new features and bug fixes, how they are tested, and how they get deployed to staging and production. If you are new to the team, read this carefully before starting your first task.

---

## Overview

Every change goes through the following stages before reaching production:

```
Your Feature Branch
       ↓  (Pull Request)
 wbc-ulmo1-stage   →   Staging Environment   →   Product Owner Testing
       ↓  (Pull Request, on sprint end)
 wbc-ulmo1-prod    →   Production Environment
```

---

## Stage 1: Development (On Your Machine)

When you pick up a new ticket (feature or bug fix):

1. **Create a new branch** from `wbc-ulmo1-stage`:
   ```bash
   git checkout wbc-ulmo1-stage
   git pull
   git checkout -b your-feature-branch-name
   ```

2. **Work on your changes** on this new branch.

3. **Test everything locally** on your own machine before raising a PR. Make sure it works as expected in your local development environment (`tutor dev`).

---

## Stage 2: Staging Deployment

Once development is done and locally tested:

1. **Create a Pull Request (PR)** from your feature branch → `wbc-ulmo1-stage`.

2. **Wait for a code review** from another developer on the team. You have to assign a reviewer. Address any feedback.

3. Once the PR is **reviewed and merged**, deploy the changes to the **staging environment**.
   - Staging uses the `wbc-ulmo1-stage` branch.
   - Follow the standard deployment steps for staging.

4. **Product Owner** will then test the feature on staging.

---

## Stage 3: Production Deployment

Once the ticket has been tested on staging and approved:

1. The ticket is marked as **"Ready for Prod"** by the product owner.

2. Production deployments happen **together on a specific day** — typically at the **end of the sprint**, when all "Ready for Prod" tickets are batched and deployed together.

3. When it is time to deploy to production:
   - Create a **PR from `wbc-ulmo1-stage` → `wbc-ulmo1-prod`**.
   - After review and merge, **deploy the changes to production**.
   - Production uses the `wbc-ulmo1-prod` branch.

---

## Summary Table

| Stage | Branch | Who is Responsible | Environment |
|---|---|---|---|
| Development | `your-feature-branch` | Developer | Local (`tutor dev`) |
| Staging | `wbc-ulmo1-stage` | Developer + Reviewer | Staging server |
| Testing | `wbc-ulmo1-stage` | Product Owner | Staging server |
| Production | `wbc-ulmo1-prod` | Developer | Production server |

---

> **Important:** Never push directly to `wbc-ulmo1-stage` or `wbc-ulmo1-prod`. Always go through a Pull Request so that changes are reviewed before they reach any shared environment.

---

**Next Step:** Head back to [README.md](README.md) for the full documentation index.
