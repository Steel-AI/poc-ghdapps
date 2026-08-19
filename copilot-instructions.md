# GHD Apps POC: context for GitHub Copilot

You are helping build a proof of concept for GHD (an engineering firm on Microsoft 365). Read this before suggesting or changing anything.

## Purpose

Publish applications built in Microsoft 365 Copilot Cowork (self-contained HTML, CSS and JavaScript bundles) to a permanent GHD URL behind Entra ID sign-in. GitHub is the source of truth and the approval layer. Azure is the runtime. Every change ships as a pull request with a preview, an approval and a rollback path. The POC must become a reusable template a second team can use without bespoke work.

Sponsor: David McLaren (Leader, AI). Owner: Steel MacDonald (AI Enablement Specialist). Timebox: spike this week, then four weeks from 31 August 2026.

## Architecture (decided)

- Runtime, first choice: Azure Static Web Apps, Standard plan, one per app. Custom authentication: Entra ID (single GHD tenant) via `staticwebapp.config.json`. All routes require the `authenticated` role.
- Runtime, fallback: one Azure App Service with Easy Auth on a GHD domain, serving many apps by path (`/appname/`) and previews under `/preview/`. Used only if Static Web Apps preview environments cannot enforce Entra sign-in (known open issue Azure/static-web-apps #1438).
- Control layer: GitHub. Pull request per change; PR preview environment; GitHub environment `production` with required reviewers gates the main-branch deploy; rollback is a revert PR.
- Identity: one single-tenant Entra app registration for the POC, `poc-ghdapps-swa`, no admin-consent permissions. Secrets only in Azure app settings, never in the repo.
- URL: production on a GHD custom domain (decision pending with IT); previews on the Azure default hostname.

## Repository layout and the one rule that matters

- `app/`: the application. Replaced wholesale by each publish from Cowork. Never hand-edit content here to "fix" something; fixes go back to the source. `app/staticwebapp.config.json` is the exception: it is wrapper configuration that has to live beside the app because Azure reads it from the deployed folder.
- `platform/`: everything engineering owns: runbook, notes, scripts, infrastructure-as-code later.
- `.github/`: CODEOWNERS, the Azure-generated deploy workflow, PR template, this file.

## Publishing contract (enforce in the pipeline, not in review meetings)

- Self-contained bundle, relative paths only.
- No external scripts, styles or fonts; dependencies vendored into the bundle.
- No secrets, keys or connection strings (secret scanning blocks the PR).
- Content Security Policy: `default-src 'self'`; inline script and style allowed for single-file artifacts; no external origins. Tighten later with hashes.
- Data ceiling: GHD Internal. Nothing Confidential.

## Current state (17 August 2026)

- Spike repo: `Steel-AI/poc-ghdapps` (personal account, public, placeholder page only, no GHD content). The real repo in a GHD GitHub org is requested (Hive ticket RITM0194897).
- Done: `app/index.html` placeholder, `platform/README.md`, `.github/CODEOWNERS`, ruleset `protect-main` (PR required, 1 approval, block force pushes, Copilot code review auto-requested), environment `production` with required reviewer and deployments from `main` only.
- Blocked: no Contributor rights in any GHD Azure subscription yet; asked Collaboration Applications and Platform Systems who owns landing-zone requests. Static Web App and Entra app registration not yet created.
- In progress: PR adding `app/staticwebapp.config.json` (tenant ID left as a placeholder while the repo is public); `platform/RUNBOOK.md`; `.github/pull_request_template.md`.

## What to build next, in order

1. `.github/pull_request_template.md` with the publishing-contract checklist.
2. `platform/RUNBOOK.md`: step-by-step with timings (see the project runbook; keep it in sync).
3. When the Static Web App exists: Azure generates `.github/workflows/azure-static-web-apps-*.yml`. Edit it so the job that deploys on push to `main` declares `environment: production`; PR jobs must not. Do not create a workflow file by hand before Azure generates it, or there will be two.
4. Add pipeline gates as workflow steps or status checks: secret scan, a check that `app/` references no external origins (grep for `http://`, `https://`, `//` in `src=` and `href=` for scripts and styles), and a CSP header presence check.
5. `platform/scripts/`: `az` CLI or Bicep to create a Static Web App, app settings and custom domain for a new app, so app number two is a script run, not a portal session.
6. Later (phase two): a Cowork plugin tool (remote MCP server) that takes a file from the Cowork session, opens a branch and PR, and returns the preview URL. Not in the spike.

## Guardrails

- Do not add GHD-Internal content, tenant IDs, client IDs or secrets to this public spike repo. Placeholders only until the repo moves into a GHD org.
- Do not change `app/index.html` except to replace it wholesale.
- Keep changes small and one-purpose per PR; the PR trail is part of the demo.
- Documentation style: British spelling, sentence-case headings, no em dashes (use a comma, colon or parentheses), "and" not "&".
- Prefer Microsoft 365 and Azure-native tools in any recommendation.

## References

- Azure Static Web Apps custom authentication: https://learn.microsoft.com/en-us/azure/static-web-apps/authentication-custom
- Preview environments: https://learn.microsoft.com/en-us/azure/static-web-apps/preview-environments
- PR preview environments (GitHub Actions): https://learn.microsoft.com/en-us/azure/static-web-apps/review-publish-pull-requests
- Known issue, custom auth on preview environments: https://github.com/Azure/static-web-apps/issues/1438
- GitHub environments and required reviewers: https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments
