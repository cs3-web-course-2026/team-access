# team-access

Self-service GitHub team access for `cs3-web-course-2026`. A student opens
an issue picking a team; a bot PR is created; the request is merged (and
team membership granted) only after review/approval from `@iamredl-lab`.

## How it works

1. Student opens an issue via **New issue → Запрос доступа к команде** and
   picks a team (`cs-31` or `cs-32`).
2. `.github/workflows/request-access.yml` opens a PR encoding the request
   as `team:<slug>` / `user:<login>` labels.
3. `.github/CODEOWNERS` requires `@iamredl-lab`'s review on anything under
   `requests/`, so the PR can't merge unreviewed.
4. On `@iamredl-lab`'s approval, `.github/workflows/grant-access.yml` adds
   the student to the GitHub team and merges the PR.

Repository access itself comes from whatever permissions the team already
has on the target repos — configure that separately, in advance, per team.

## One-time setup (repo owner)

1. **Branch protection on `main`**: Settings → Branches → add rule for
   `main` → enable "Require a pull request before merging" and "Require
   review from Code Owners".
2. **Org admin token**: create a classic PAT scoped to `admin:org` from an
   org-owner account, then add it as an **organization secret** named
   `ORG_ADMIN_TOKEN` (Settings → Secrets and variables → Actions, at the
   org level, restricted to this repo). The default `GITHUB_TOKEN` cannot
   manage org team membership, so this step is required for
   `grant-access.yml` to work.
3. **Team permissions**: for each team in the issue-form dropdown
   (`cs-31`, `cs-32`), make sure it already has the right permission level
   on the repos students need — this flow only adds team *membership*, not
   repo permissions.

## Verifying the flow end-to-end

1. Open an issue via the **Запрос доступа к команде** form as a test
   account, picking a team.
2. Confirm a PR appears within a minute or two, labeled `access-request`,
   `team:<slug>`, `user:<test-account>`, with `@iamredl-lab` requested as
   reviewer, and containing `requests/<test-account>--<slug>.md`.
3. Confirm the PR cannot be merged before it's approved (branch protection
   working).
4. Approve the PR as `@iamredl-lab`. Confirm: the test account is added to
   the team (Settings → People, or `gh api
   orgs/cs3-web-course-2026/teams/<slug>/memberships/<test-account>`), and
   the PR auto-merges.
5. As a sanity check on the security guard, have a *different* reviewer
   (not `@iamredl-lab`) approve a fresh request PR and confirm no grant
   happens — only `@iamredl-lab`'s approval triggers `grant-access.yml`.
