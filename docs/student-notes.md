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

Completed on 2026-09-10. Screenshot 2 captured the empty environment settings before hardening. Screenshot 3 captured the saved production protections. PR #1 was merged as e3582b53a58f061881a111f4defcfe79b2f1da23, triggering the deployment from main.

Verified configuration: required reviewer `karansinghgurjar212001` (verified repository owner/admin), selected deployment branch `main`, one-minute wait timer, and administrator bypass disabled. Self-review is needed for this explicitly authorized single-account lab demonstration.

GitHub's native environment reviewer list requires approval from **one** listed reviewer, not every listed reviewer. The starter README's claim that listing two users requires two approvals is inaccurate. A genuine two-approval policy needs an additional control and a second authorized reviewer. See https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#required-reviewers.

## Local validation

- YAML parsed with PyYAML; main-only push, exact-ref job condition, environment target, least-privilege permissions, and absence of secret references asserted.
- Actionlint 1.7.12: both workflows pass after the CI action update. Official release archive checksum verified.
- `npm test`: passed, but is only an echo placeholder, not application test coverage.
- `npm run build`: passed, but is only an echo placeholder, not a compilation step.
- GitHub workflow execution, approval, and final configuration verification: passed. Run https://github.com/karansinghgurjar212001/Deployment-Governance-Lab/actions/runs/34473138039 reached status waiting, with production waiting for review. Screenshot 4 was captured before approval. The pending-deployments API confirmed the configured reviewer could approve. The approval audit records karansinghgurjar212001 approving, and github-actions[bot] satisfying the one-minute wait timer. The deploy job completed successfully at 2026-09-10T11:48:50Z; logs include Simulating deployment to production and Application deployed successfully. Screenshot 5 captures the unchanged protections after success. This was a simulation, not a cloud deployment.

Screenshots and the submission PDF are kept outside this repository.

## Final evidence

Exactly five screenshot files are stored outside the repository, ordered: initial workflow, initial settings, corrected protections, real approval pause, final protections. Screenshot 1 is preserved unchanged. No credentials exist in the repository or production environment, and none were invented. The final PDF requires the student's actual roll number, which was not present in the supplied request.
