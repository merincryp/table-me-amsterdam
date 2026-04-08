---
name: deploy
description: Auto-invoked deployment workflow for Vercel
trigger: When user requests deploy or push to production
---

# Deploy Workflow

## Pre-deploy Checks
1. All changes committed to git
2. `vercel.json` valid and up to date
3. No console errors in browser
4. Responsive design verified (mobile + desktop)
5. All external links working

## Deploy Steps
1. `git add` and `git commit` with descriptive message
2. `git push origin main`
3. Vercel auto-deploys from main branch
4. Verify deployment at production URL

## Post-deploy
- Check production URL loads correctly
- Test mobile viewport on real device
- Verify all interactive elements work
