# GHD Apps runbook

Repeatable steps for publishing a Cowork-built app to a GHD URL behind Entra sign-in. Each step records what to click, what to set and how long it took. This file becomes the standard operating procedure for the next team and the onboarding pack for new team members. Findings and roadblocks are kept in the project log, not here.

Conventions: resources named `poc-ghdapps-*`; tags owner, purpose, expiry on every Azure resource; nothing GHD-Internal in any repo outside a GHD GitHub org.

## Phase 0: before you touch anything

1. Confirm the GitHub org and repo owner (Hive request if needed; the POC used RITM0194897).
2. Confirm the Azure subscription and resource group with Contributor rights (landing-zone request; group to be confirmed).
3. Heads-up to Collaboration Applications (Kaine Sherwood) and Platform Systems (Dan Jones) before creating resources. Not strictly required for a spike; kept for no-surprises.

## Phase 1: repository (about 20 minutes)

1. New repository in the GHD org: name `<app>-ghdapps`, private or internal, README on, no template, no .gitignore, no licence, no Copilot jumpstart. Description names owner, sponsor and tear-down or review date.
2. `app/index.html`: the app bundle from Cowork (the spike used a placeholder page with links to `/.auth/me` and `/.auth/logout`).
3. `platform/README.md`: engineering owns the wrapper; `app/` is replaced wholesale by each publish; never hand-edit it.
4. `.github/CODEOWNERS`: `/app/` owner and deputy; `/platform/` and `/.github/` engineering.
5. `.github/copilot-instructions.md` and `.github/PULL_REQUEST_TEMPLATE.md` from the template.
6. Settings, Rules, Rulesets, New branch ruleset `protect-main`: Active; target default branch; Require a pull request before merging (1 approval); Block force pushes; Automatically request Copilot code review. In the GHD org, also Require review from the owning team.
7. Settings, Code security: Secret scanning and Push protection on.
8. Settings, Environments, New environment `production`: Required reviewers = owner and deputy; Deployment branches and tags = Selected, add `main`.
9. Check: editing `app/index.html` directly on `main` is refused and GitHub offers a branch and PR.

Spike notes (17 Aug 2026): personal Free-plan account enforces rules only on public repos; repo made public with placeholder content. Ruleset UI offers "Require review from specific teams" rather than Code Owners; CODEOWNERS still auto-requests reviewers.

## Phase 2: Static Web App (about 15 minutes plus first build)

1. Azure portal, Create a resource, Static Web App. Subscription and resource group per Phase 0. Name `poc-ghdapps-<app>`. Plan type Standard. Region nearest users. Deployment source GitHub; sign in; org, repo, branch `main`. Build presets Custom; app location `app`; API location blank; output location blank. Tags. Review + create.
2. Azure commits `.github/workflows/azure-static-web-apps-<id>.yml` and stores the deployment token as a repo secret. Watch the Actions run; open the default hostname; confirm the app renders.
3. Record: minutes from Create to live.

## Phase 3: Entra sign-in (about 30 minutes)

1. Entra admin centre, App registrations, New registration: name `poc-ghdapps-swa`; single tenant; platform Web; redirect URI `https://<default-host>/.auth/login/aad/callback`. Certificates and secrets, New client secret; copy it once. Note tenant ID and client ID. No API permissions beyond the defaults (no admin consent).
2. Static Web App, Settings, Environment variables (production): `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`.
3. Merge the PR that adds `app/staticwebapp.config.json` with the tenant ID filled in (see template). Routes lock `/*` to `authenticated`; 401 redirects to `/.auth/login/aad`; CSP header set.
4. Test in a private window: GHD account signs in and sees the app; `/.auth/me` shows claims; a personal Microsoft account is refused at the Microsoft sign-in page.

## Phase 4: preview, approval, rollback (about 90 minutes)

1. Branch, visible change to `app/index.html`, open PR. The workflow creates a staging environment at `<default-host>-<PR number>.<region>.azurestaticapps.net`.
2. Open the preview in a private window and record the result: redirect URI mismatch (add the preview callback URL to the app registration and retry), works, or token or issuer failure. Note whether the preview needs its own environment variables (portal, Environments, select the PR environment).
3. Paste the preview link in a Teams chat; record whether Teams blocks it.
4. Merge; confirm production changed on the same URL; close the PR; confirm the staging environment is deleted.
5. Edit the Azure-generated workflow so the job that deploys on push to `main` declares `environment: production`; PR jobs do not. Push a change through a PR: preview builds without approval; production deploy pauses until a reviewer approves.
6. Open the merged PR, Revert, approve, merge. Record minutes from click to the old version live.

## Phase 5: custom domain (after the DNS decision)

1. Static Web App, Custom domains, Add: `<app>.apps.ghd.com` (or the agreed name). Add the CNAME or TXT record in GHD DNS per the portal instructions. Wait for validation; free managed certificate is issued.
2. Update the app registration redirect URI to the custom domain. Re-test sign-in.

## Phase 6: write-up and hand-over

1. Fill in timings and screenshots in this file. Update the project log with findings.
2. Delete spike resources or record them with a keep-until date.
3. For a new app: copy this repo from the template, run Phases 1 to 5, and record anything that needed a person to intervene. That list is the backlog for automation.

## Timings recorded

| Step | Date | Minutes | Notes |
|---|---|---|---|
| Phase 1 repository | 17 Aug 2026 | ~20 | Spike on personal account |
| Phase 2 create to live | | | |
| Phase 3 sign-in working | | | |
| Phase 4 change to preview | | | |
| Phase 4 approval to production | | | |
| Phase 4 rollback | | | |
