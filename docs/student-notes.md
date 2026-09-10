# Deployment Governance Audit

Fork: https://github.com/karansinghgurjar212001/Deployment-Governance-Lab
Initial main commit: d6f3d4271819e5c1b657d6c5e69ebefd8e5e3ab2
Audit date: 2026-09-10

## Initial findings

1. **Broad deployment triggers.** `deploy.yml` uses `push.branches: ['*']` and unrestricted manual dispatch. The wildcard matches branch names without slashes; manual dispatch also lacks a branch guard. Unreviewed code can enter the production simulation. Restrict push and the job itself to `refs/heads/main`.
2. **No environment gate.** The deploy job has no `environment` target. The fork's Environments API returned zero environments, so no reviewer approval, wait timer, or environment deployment branch policy exists. Configure a protected `production` environment and reference it from the job.
3. **Unsafe credential handling.** The original deploy step references `DEPLOY_KEY`, `DATABASE_URL`, and `API_TOKEN`, and prints the database URL and credential prefixes. The fork's repository secrets API returned zero secrets; there are no existing credentials to migrate. The deployment is an echo simulation and needs no credentials. Remove unused secret access and all secret logging. Any future real deployment must use production environment secrets after approval, never log values, and audit repository/organization fallback scopes.
4. **Unnecessary token permission.** The original job grants `id-token: write`, but performs no OIDC authentication. Remove it; retain only `contents: read`, with checkout credential persistence disabled.
5. **Unprotected main.** The classic protection API returned `Branch not protected`; the rulesets API returned an empty list. No existing protection was altered. Branch protection and environment deployment restrictions are separate controls.
6. **Obsolete CI action.** Actionlint 1.7.12 rejects the existing `actions/setup-node@v3` runner. Update to v6 with Node.js 22 so PR validation can run.

The source repository intentionally omits governance; these gaps allow deployments without controlled approval and would expose credentials if real values were added.

## Workflow changes

- Production pushes restricted to `main`.
- Job guard also rejects manual dispatch from other branches or tags.
- Deploy job targets `environment: production` and has a 10-minute execution timeout.
- Only read access to repository contents is granted; no OIDC write permission.
- Checkout does not persist its token.
- Existing build and simulated deployment behavior retained; no cloud infrastructure or credentials added.
- Secret references and secret-value logging removed.

## Environment configuration and behavioral validation

Pending. The browser needs an authenticated session to capture the required initial settings screenshot before configuration changes. No approval-gate success is claimed until a real main-branch run pauses, is captured, is approved, and completes.

Intended configuration: required reviewer `karansinghgurjar212001` (verified repository owner/admin), selected deployment branch `main`, one-minute wait timer, and administrator bypass disabled. Self-review is needed for this explicitly authorized single-account lab demonstration.

GitHub's native environment reviewer list requires approval from **one** listed reviewer, not every listed reviewer. The starter README's claim that listing two users requires two approvals is inaccurate. A genuine two-approval policy needs an additional control and a second authorized reviewer. See https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#required-reviewers.

## Local validation

- YAML parsed with PyYAML; main-only push, exact-ref job condition, environment target, least-privilege permissions, and absence of secret references asserted.
- Actionlint 1.7.12: both workflows pass after the CI action update. Official release archive checksum verified.
- `npm test`: passed, but is only an echo placeholder, not application test coverage.
- `npm run build`: passed, but is only an echo placeholder, not a compilation step.
- GitHub workflow execution, approval, and final configuration verification: pending.

Screenshots and the submission PDF are kept outside this repository.
