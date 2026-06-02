# Preview Environments Theory

## What It Is

A preview environment is a temporary environment created for a pull request, branch, or feature so reviewers can see and test the change before it is merged.

Preview environments are common for web apps, APIs, documentation sites, design systems, and SaaS features.

## Main Purpose

Preview environments make review more concrete.

They allow:

- Product review.
- UI review.
- QA testing.
- Stakeholder feedback.
- API behavior testing.
- Early integration checks.

Instead of only reading code, reviewers can use the running change.

## How It Works

Typical flow:

1. Developer opens pull request.
2. CI builds the branch.
3. Pipeline deploys an isolated preview environment.
4. Bot comments the preview URL on the pull request.
5. Reviewers test the change.
6. Environment updates on new commits.
7. Environment is destroyed when PR closes or merges.

## Architecture Options

Preview environments can be implemented with:

- Static hosting previews.
- One namespace per pull request in Kubernetes.
- One container service per branch.
- Serverless function preview aliases.
- Temporary cloud stacks.
- Platform features such as Vercel/Netlify previews.

The architecture depends on cost, app complexity, and isolation needs.

## Isolation

Preview environments should be isolated enough that branches do not affect each other.

Isolation dimensions:

- URL.
- Runtime compute.
- Database schema or database.
- Cache namespace.
- Queue names.
- Object storage prefix.
- Secrets.
- External service sandbox.

Full isolation is safer but more expensive.

## Data Strategy

Preview data should be disposable.

Options:

- Seed data on environment creation.
- Use shared read-only test data.
- Create per-preview database.
- Create per-preview schema.
- Use mocked backend for frontend-only previews.

Avoid connecting previews to production databases.

## Secrets and Security

Preview environments often run unmerged code, so secrets must be restricted.

Important rules:

- Do not expose production secrets.
- Be careful with pull requests from forks.
- Use preview-scoped credentials.
- Restrict external side effects.
- Require authentication if preview contains sensitive features.
- Expire environments automatically.

If untrusted contributors can modify code, they may try to print secrets from the pipeline.

## CI/CD Role

Preview environments are created by CI/CD automation.

Pipeline responsibilities:

- Build preview artifact.
- Deploy isolated environment.
- Run basic checks.
- Post preview URL.
- Update preview on new commits.
- Destroy environment after PR closes.

Cleanup is critical. Forgotten previews waste money and may expose old code.

## Review Workflow

Preview environments improve review by allowing multiple roles to test:

- Engineers check behavior.
- Designers check UI.
- Product checks workflow.
- QA checks acceptance criteria.
- Security reviews risky paths.

The preview URL should show version and branch information so reviewers know what they are testing.

## Common Problems

- Preview environments are not cleaned up.
- Preview uses production secrets.
- Shared database causes branch conflicts.
- Preview is slower than production and gives misleading feedback.
- Preview URL changes too often.
- External webhooks point to wrong branch.
- Costs grow with many open PRs.

## Technical Controls

Useful controls:

- Automatic TTL.
- PR-close cleanup job.
- Unique resource names.
- Cost tags.
- Preview-specific secrets.
- Basic authentication.
- Branch/commit label visible in app.
- Resource quotas.
- Fork PR restrictions.

## Practical Lab Ideas

- Create a preview deployment per pull request.
- Comment the preview URL automatically on the PR.
- Add cleanup when PR closes.
- Use per-branch database schema.
- Add visible commit SHA to the preview app.

## Short Summary

Preview environments let teams test a pull request as a running app. They improve review quality, but must be isolated, secure, disposable, and automatically cleaned up.
