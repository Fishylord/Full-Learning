# CI/CD Topics Guide

CI/CD is the practice of moving code from development to production through automated, repeatable, tested, and observable workflows.

- CI means Continuous Integration: frequently merging code and automatically validating it.
- CD can mean Continuous Delivery: code is always releasable, but production release may require approval.
- CD can also mean Continuous Deployment: every change that passes checks is automatically deployed to production.

Each topic folder uses:

- `Theory`: concepts, diagrams, notes, tradeoffs, and questions.
- `Practical/Code`: pipeline files, scripts, configs, test examples, deployment manifests, and lab exercises.

## 1. Core CI/CD Fundamentals

### Continuous Integration

Continuous Integration means developers merge code often and every change is validated automatically. A CI pipeline usually runs linting, formatting checks, unit tests, build verification, dependency checks, and sometimes integration tests.

Key idea: find problems early while the change is still small.

Common tools: GitHub Actions, GitLab CI, Jenkins, CircleCI, Azure DevOps, Bitbucket Pipelines, Buildkite, TeamCity.

Practical work:

- Create a pipeline that runs on every pull request.
- Install dependencies.
- Run linting and unit tests.
- Fail the pipeline if tests fail.

### Continuous Delivery

Continuous Delivery means the main branch is always in a deployable state. The pipeline builds, tests, packages, and prepares a release artifact. A human approval may still be required before production deployment.

Key idea: release should be a business decision, not a technical emergency.

Practical work:

- Build an artifact on merge to main.
- Deploy automatically to staging.
- Require manual approval before production.

### Continuous Deployment

Continuous Deployment means every change that passes automated checks is deployed to production without manual approval.

Key idea: automation must be strong enough that production deployment is routine.

Use it when test coverage, monitoring, rollback, and release controls are mature. Avoid it when changes are risky, regulated, or hard to roll back.

Practical work:

- Automatically deploy main branch to production after all checks pass.
- Add rollback or roll-forward automation.

### Pipeline as Code

Pipeline as Code means CI/CD workflows are defined in version-controlled files, such as `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, or `azure-pipelines.yml`.

Benefits:

- Reviewable changes.
- Reusable templates.
- Version history.
- Easier disaster recovery.

Practical work:

- Create reusable workflow templates for build, test, scan, and deploy.

### Branching Strategies

Branching strategy controls how code moves through collaboration and release.

Common models:

- Trunk-based development: small frequent changes into main.
- Feature branches: short-lived branches merged through pull requests.
- GitFlow: long-running branches such as develop, release, and hotfix.
- Release branches: branch from main when stabilizing a release.

CI/CD works best with short-lived branches and frequent integration.

Practical work:

- Compare trunk-based and GitFlow pipelines.
- Add branch-specific pipeline behavior for pull requests, main, and release branches.

### Artifact Management

Artifacts are outputs from the pipeline: binaries, Docker images, packages, compiled files, build reports, SBOMs, or deployment bundles.

Good artifact practice:

- Build once, promote the same artifact across environments.
- Version artifacts clearly.
- Store artifacts in a registry.
- Do not rebuild separately for staging and production.

Common registries: Docker registry, ECR, GitHub Packages, GitLab Package Registry, Nexus, Artifactory, CodeArtifact.

### Environment Promotion

Environment promotion means moving the same version from dev to test to staging to production.

Good promotion includes:

- Same artifact across environments.
- Environment-specific config outside the artifact.
- Approval gates where needed.
- Smoke tests after deployment.

## 2. Pipeline Stages

### Source Checkout

The pipeline starts by checking out source code from Git. It should know the commit SHA, branch, tag, author, and pull request metadata.

Practical detail: stamp the commit SHA into the build so production can identify exactly what code is running.

### Build

The build stage compiles or prepares the application. For Docker apps, this usually creates an image. For frontend apps, it may produce static assets. For backend apps, it may compile code, install dependencies, or package binaries.

Best practice: builds should be deterministic and repeatable.

### Unit Testing

Unit tests validate small pieces of code in isolation. They should be fast and run on every pull request.

Pipeline rule: if unit tests fail, stop immediately.

### Integration Testing

Integration tests validate multiple parts working together, such as API plus database, service plus queue, or app plus cache.

Practical work:

- Use Docker Compose or test containers.
- Create temporary test databases.
- Clean up after the test run.

### End-to-End Testing

E2E tests validate real user or system flows through the full stack.

Use them carefully because they are slower and more brittle than unit tests. Run critical E2E tests before production deployment or on release candidates.

### Static Analysis

Static analysis checks code without running it. It includes linting, formatting, type checking, complexity checks, and code quality rules.

Examples:

- ESLint, Prettier, TypeScript.
- Ruff, Black, Mypy.
- Checkstyle, PMD, SpotBugs.
- SonarQube.

### Dependency Scanning

Dependency scanning checks third-party packages for known vulnerabilities and license issues.

Examples: Dependabot, npm audit, pip-audit, Snyk, Trivy, OWASP Dependency-Check.

### Container Scanning

Container scanning checks Docker images for OS and package vulnerabilities, secrets, and misconfiguration.

Examples: Trivy, Grype, Snyk Container, Docker Scout, ECR scan.

### Packaging

Packaging prepares the deployable output. It may create Docker images, JAR files, wheels, npm packages, Lambda zip files, Helm charts, or static asset bundles.

### Publishing Artifacts

Publishing stores artifacts in a registry or artifact repository. The deploy stage should pull from the registry instead of building again.

### Deployment

Deployment updates the runtime environment. This may update Kubernetes, ECS, EC2, Lambda, static hosting, databases, or infrastructure.

Deployment should include:

- Version traceability.
- Health checks.
- Logs and metrics.
- Rollback or roll-forward path.

### Smoke Testing

Smoke tests are quick checks after deployment to confirm the system is basically working.

Examples:

- Health endpoint returns success.
- Login works.
- One critical API call works.
- Database connection works.

### Rollback

Rollback returns the system to a previous known-good version. It must be tested before production incidents happen.

Rollback is harder when database migrations are not backward-compatible.

## 3. Deployment Strategy Types

### Recreate Deployment

Recreate deployment stops the old version, then starts the new version.

Pros:

- Simple.
- Easy to understand.
- Useful for small internal apps.

Cons:

- Causes downtime.
- Risky for user-facing production systems.

Use for: dev environments, simple tools, low-traffic services where downtime is acceptable.

### Rolling Deployment

Rolling deployment replaces old instances gradually with new instances.

Example: if you have 10 app instances, replace 2 at a time until all run the new version.

Pros:

- No full downtime if capacity is sufficient.
- Common default in Kubernetes and many platforms.
- Simple operational model.

Cons:

- Old and new versions run at the same time.
- Requires backward-compatible APIs and database changes.
- Rollback may take time.

Use for: most normal web services and APIs.

Important checks:

- Readiness checks.
- Liveness checks.
- Backward-compatible database migrations.
- Load balancer health checks.

### Blue-Green Deployment

Blue-green deployment keeps two environments:

- Blue: current production.
- Green: new version.

Traffic is switched from blue to green after green is verified.

Pros:

- Fast rollback by switching traffic back.
- New version can be tested before receiving traffic.
- Clear separation between old and new.

Cons:

- Requires duplicate infrastructure.
- More expensive.
- Database compatibility can still be difficult.

Use for: high-value production services where fast rollback matters.

Practical flow:

1. Blue serves production traffic.
2. Deploy new version to green.
3. Run smoke and integration checks on green.
4. Switch load balancer or DNS traffic to green.
5. Keep blue temporarily for rollback.
6. Remove blue after confidence period.

### Canary Deployment

Canary deployment sends a small percentage of traffic to the new version first, then increases gradually.

Example:

- 1 percent traffic to new version.
- Then 5 percent.
- Then 25 percent.
- Then 50 percent.
- Then 100 percent.

Pros:

- Limits blast radius.
- Good for detecting production-only problems.
- Works well with metrics-based promotion.

Cons:

- Requires traffic splitting.
- Requires strong observability.
- Some bugs only appear at higher traffic.

Use for: user-facing apps, APIs, high-risk changes, performance-sensitive services.

Promotion signals:

- Error rate.
- Latency p95 and p99.
- Business metrics.
- Logs.
- User complaints.
- Infrastructure saturation.

### User-ID Targeted Deployment

User-ID targeted deployment releases a new version or feature to specific users or user groups. This is often done through feature flags, routing rules, or experiment platforms.

Examples:

- Enable feature only for user ID `123`.
- Enable for internal employees.
- Enable for beta testers.
- Enable for 5 percent of users based on hash of user ID.
- Enable for users in one region or tenant.

Pros:

- Very controlled rollout.
- Great for beta testing.
- Allows quick disable without redeploy.
- Can avoid exposing unfinished features to everyone.

Cons:

- Requires feature flag discipline.
- Adds conditional paths in code.
- Old flags must be cleaned up.
- User experience can become inconsistent if not designed carefully.

Use for: product features, experiments, risky UI changes, tenant-by-tenant SaaS rollout, internal dogfooding.

Important implementation detail:

Use a stable hash of user ID for percentage rollout. Random rollout per request can create inconsistent behavior.

### Feature-Flag Deployment

Feature-flag deployment separates code deployment from feature release. The code is deployed, but the feature is controlled by a flag.

Types of flags:

- Release flags.
- Experiment flags.
- Permission flags.
- Operational kill switches.
- Migration flags.

Pros:

- Release without redeploy.
- Kill switch for broken features.
- Enables user-targeted rollout.

Cons:

- Flag debt.
- More test combinations.
- Must secure server-side decisions.

Best practice: every flag should have an owner and removal date.

### Dark Launch

Dark launch deploys functionality without making it visible to users. The system may process real traffic internally, but users do not see the output.

Example: compute a new recommendation model in parallel but do not show results yet.

Use for: performance validation, data collection, backend readiness, hidden product capabilities.

### Shadow Deployment

Shadow deployment sends a copy of real traffic to the new version while the old version still handles real responses. The new version's response is ignored or compared.

Pros:

- Tests new version with real traffic.
- No user-facing risk if isolated correctly.

Cons:

- Must prevent side effects.
- Can double load.
- Requires careful data isolation.

Use for: API rewrites, performance testing, migration validation.

### A/B Testing Deployment

A/B testing sends different users to different variants to compare outcomes.

This is product experimentation more than pure deployment, but it often uses CI/CD and feature flag systems.

Use for: conversion changes, onboarding flows, recommendation logic, pricing pages, UI experiments.

Important: define success metrics before rollout.

### Ring Deployment

Ring deployment rolls out to groups in ordered rings.

Example:

- Ring 0: developers.
- Ring 1: internal employees.
- Ring 2: beta customers.
- Ring 3: 5 percent public users.
- Ring 4: all users.

Use for: large organizations, SaaS products, enterprise customers, platform changes.

### Immutable Deployment

Immutable deployment creates new infrastructure for each release instead of modifying existing servers in place.

Pros:

- Repeatable.
- Reduces configuration drift.
- Easier rollback.

Cons:

- Requires infrastructure automation.
- May take longer to provision.

Use for: autoscaling groups, AMI-based releases, containerized platforms.

### Progressive Delivery

Progressive delivery is controlled, automated rollout based on health signals. Canary, feature flags, user targeting, and metric-based promotion are part of progressive delivery.

Goal: release gradually and automatically stop or roll back if metrics become unhealthy.

## 4. Environments

### Development

Used by developers for active work. It may be local or shared. It is not production-like enough for release confidence.

### Testing

Used for automated integration, QA, and validation. Data should be fake or sanitized.

### Staging

Staging should mirror production closely. It is used for final validation before production.

Key point: staging is only useful when configuration, deployment process, and dependencies are close to production.

### Production

Production serves real users. It requires strict access control, monitoring, backups, incident response, and release discipline.

### Preview Environments

Preview environments are temporary environments created per pull request or branch.

Use for: UI review, QA, stakeholder review, integration checks.

### Ephemeral Environments

Ephemeral environments are temporary environments created automatically and destroyed after use.

Use for: tests, demos, pull requests, isolated experiments.

## 5. Infrastructure and Deployment Tooling

### Infrastructure as Code

IaC defines infrastructure in code. Examples include Terraform, CloudFormation, CDK, Pulumi, and Bicep.

CI/CD should validate and apply infrastructure changes safely.

Pipeline stages:

- Format.
- Validate.
- Plan.
- Security scan.
- Approval.
- Apply.

### GitOps

GitOps uses Git as the source of truth for infrastructure and deployments. A controller watches Git and reconciles the environment.

Examples: Argo CD, Flux.

Use for: Kubernetes-heavy systems, auditable deployments, declarative operations.

### Kubernetes Deployments

Kubernetes supports rolling updates, readiness probes, liveness probes, rollback, replica management, and deployment history.

CI/CD should produce images and update manifests or Helm values.

### Helm

Helm packages Kubernetes manifests into charts. It supports values, templates, releases, and rollbacks.

Use for: repeatable Kubernetes app deployment.

### Database Migrations

Database migrations are one of the riskiest parts of deployment.

Safe pattern:

1. Expand: add backward-compatible schema changes.
2. Deploy app that uses both old and new safely.
3. Migrate data.
4. Contract: remove old schema after all old code is gone.

Avoid deployments where new code requires a schema change that breaks old code during rollback.

### Configuration Management

Config should be external to artifacts. The same artifact should run in staging and production with different config.

Examples:

- Environment variables.
- Config files from secret stores.
- Parameter Store.
- Kubernetes ConfigMaps.

### Secrets Management

Secrets must not be hardcoded in source code or CI logs.

Use secret managers:

- AWS Secrets Manager.
- AWS Systems Manager Parameter Store.
- HashiCorp Vault.
- GitHub Actions secrets.
- GitLab CI variables.

## 6. DevSecOps Topics

### Secret Scanning

Secret scanning detects leaked credentials, tokens, private keys, and passwords.

Run it on pull requests and main branches.

### SAST

Static Application Security Testing scans source code for security issues.

Examples: CodeQL, Semgrep, SonarQube, Checkmarx.

### DAST

Dynamic Application Security Testing tests a running application from the outside.

Examples: OWASP ZAP, Burp Suite automation.

### SBOM

Software Bill of Materials lists dependencies and components inside software.

Use for supply chain visibility and vulnerability response.

### Artifact Signing

Artifact signing proves an artifact came from a trusted build process and was not modified.

Examples: Cosign, Sigstore, Notary.

### Policy as Code

Policy as Code enforces rules automatically.

Examples:

- Block public S3 buckets.
- Require image scanning.
- Require approved base images.
- Require Kubernetes resource limits.

Tools: OPA, Conftest, Kyverno, Checkov.

## 7. Operations and Release Control

### Health Checks

Health checks tell deployment systems whether a version is safe to receive traffic.

Types:

- Liveness: should the process be restarted?
- Readiness: can this instance receive traffic?
- Startup: has the app finished booting?

### Observability

Production deployment needs logs, metrics, and traces.

Useful release metrics:

- Error rate.
- Latency p95 and p99.
- CPU and memory.
- Queue depth.
- Database load.
- Business transaction success.

### Release Approvals

Approvals are manual gates before sensitive deployments.

Use for:

- Production deployment.
- Infrastructure changes.
- Database migrations.
- Regulated systems.

### Change Management

Change management tracks what changed, who approved it, when it deployed, and how it can be rolled back.

Useful for audits and incident investigation.

### Incident Rollback

Rollback should be a practiced operation, not a guess during an outage.

Rollback requirements:

- Previous artifact available.
- Compatible database state.
- Traffic switch mechanism.
- Communication plan.

### Roll Forward

Roll forward means fixing production by deploying a new correction instead of returning to an old version.

Use when rollback is unsafe, especially after database changes.

## 8. Metrics

### DORA Metrics

DORA metrics measure software delivery performance:

- Deployment frequency.
- Lead time for changes.
- Change failure rate.
- Mean time to recovery.

### Lead Time

Lead time measures how long it takes for a change to go from commit to production.

Short lead time usually means the team can respond quickly.

### Deployment Frequency

Deployment frequency measures how often production deployments happen.

High frequency is good only when quality remains high.

### Change Failure Rate

Change failure rate measures how often deployments cause incidents, rollbacks, or hotfixes.

### MTTR

MTTR means Mean Time To Recovery. It measures how quickly the team restores service after failure.

## 9. Common CI/CD Pipeline Templates

### Basic Pull Request CI

```yaml
stages:
  - checkout
  - install
  - lint
  - unit_test
  - build
```

Use for every pull request.

### Standard Web App CI/CD

```yaml
stages:
  - checkout
  - install
  - lint
  - unit_test
  - integration_test
  - build
  - container_scan
  - publish_image
  - deploy_staging
  - smoke_test_staging
  - approval
  - deploy_production
  - smoke_test_production
```

Use for normal production applications.

### Progressive Delivery Pipeline

```yaml
stages:
  - build
  - test
  - publish
  - deploy_canary_1_percent
  - observe_metrics
  - deploy_canary_10_percent
  - observe_metrics
  - deploy_canary_50_percent
  - observe_metrics
  - deploy_100_percent
```

Use for high-value systems with strong monitoring.

### Infrastructure CI/CD

```yaml
stages:
  - format
  - validate
  - security_scan
  - plan
  - approval
  - apply
  - drift_check
```

Use for Terraform, CloudFormation, CDK, Pulumi, or similar infrastructure changes.

## 10. Learning Order

1. Continuous Integration, pull request checks, unit tests, and build artifacts.
2. Continuous Delivery, staging deployment, approvals, and smoke tests.
3. Rolling deployment and blue-green deployment.
4. Canary deployment, user-ID targeted deployment, feature flags, and progressive delivery.
5. Docker image build, registry publishing, Kubernetes or ECS deployment.
6. Infrastructure as Code, GitOps, and database migrations.
7. DevSecOps: secret scanning, SAST, dependency scanning, SBOM, and artifact signing.
8. Observability, rollback, roll-forward, DORA metrics, and incident response.

## 11. Key Interview Questions

- What is the difference between Continuous Delivery and Continuous Deployment?
- Why should you build once and promote the same artifact?
- What can go wrong during rolling deployment?
- How does blue-green deployment make rollback easier?
- When is canary deployment better than blue-green?
- How do you roll out a feature to only specific user IDs?
- Why are database migrations risky in CI/CD?
- What checks should run before production deployment?
- What metrics tell you a deployment is unhealthy?
- What is the difference between rollback and roll-forward?

## 12. Full Theory Notes by Topic

This section expands the theory for the CI/CD folders in this directory. The goal is not to memorize every tool name. The goal is to understand what problem each topic solves, what can go wrong, and what a mature pipeline should do about it.

### Fundamentals: Continuous Integration

Continuous Integration is the habit and system of integrating code changes into a shared branch frequently. In a weak process, developers work separately for days or weeks, then integration becomes a large painful event. CI reduces that risk by forcing small changes to meet the shared codebase constantly.

A good CI system answers one question quickly: is this change safe enough to merge? It does that by running fast checks automatically. Typical checks are formatting, linting, type checking, unit tests, build verification, dependency installation, and sometimes lightweight security scanning. The faster these checks run, the more useful CI becomes. If CI takes too long, developers avoid using it or batch too much work together.

CI is not only a tool. GitHub Actions, GitLab CI, Jenkins, and CircleCI can run CI jobs, but the engineering practice is smaller commits, frequent merges, reliable tests, and a team culture that keeps the main branch healthy. A broken main branch blocks everyone, so fixing it should usually take priority over starting new feature work.

Good CI characteristics:

- Runs on every pull request and important branch push.
- Produces deterministic results.
- Fails clearly with useful logs.
- Finishes fast enough to support developer flow.
- Blocks merges when required quality gates fail.
- Uses clean environments instead of relying on a developer machine.

Common mistakes:

- Tests are flaky, so people rerun until green.
- CI only runs after merge, so broken code reaches main.
- CI builds different artifacts for each environment.
- Required checks are too slow or too broad.
- Secrets are exposed in logs.

### Fundamentals: Continuous Delivery

Continuous Delivery means the codebase is always kept in a releasable state. Every change that passes the pipeline should produce a deployable artifact. Production release can still require a human approval, but the technical work needed to release has already been automated.

The important idea is that delivery is not the same as deployment. Delivery prepares software for release. Deployment puts it into an environment. A team practicing Continuous Delivery may automatically deploy to staging and then wait for approval before production.

Continuous Delivery depends on discipline around testing, artifacts, configuration, and environment promotion. The same artifact should move from staging to production. If staging builds one artifact and production builds another, then staging did not really test what production will run.

Useful gates in Continuous Delivery:

- Unit tests and integration tests.
- Security and dependency checks.
- Build artifact creation.
- Staging deployment.
- Smoke tests.
- Manual production approval.
- Release notes or change record.

### Fundamentals: Continuous Deployment

Continuous Deployment is the stronger form of CD where every change that passes the pipeline is automatically deployed to production. There is no manual approval gate for normal changes.

This is powerful, but it requires maturity. The team needs strong automated tests, observability, fast rollback or roll-forward, small changes, feature flags for risky features, and production metrics that can catch issues quickly. Continuous Deployment is unsafe if production failures are hard to detect or hard to reverse.

Continuous Deployment works best when changes are small and routine. Large releases are harder to validate, harder to roll back, and harder to diagnose. A small change that fails is easier to understand.

Use Continuous Deployment when:

- Tests are reliable.
- Changes are small.
- The system is observable.
- Rollback or roll-forward is practiced.
- Product features can be disabled independently from code deployment.

Avoid it when:

- Releases require legal or regulatory approval.
- Database changes are frequently destructive.
- Monitoring is weak.
- Manual QA is the only reliable safety net.

### Fundamentals: Pipeline as Code

Pipeline as Code means pipeline behavior is declared in files stored with the code or in a central pipeline repository. Examples include GitHub Actions workflow YAML, GitLab CI YAML, Jenkinsfile, Azure Pipelines YAML, Buildkite pipeline files, and reusable templates.

This matters because a pipeline is production infrastructure. It decides what is tested, what is released, and who can deploy. If pipeline configuration lives only in a UI, changes are harder to review, reproduce, and audit.

Good Pipeline as Code uses reusable pieces without hiding important behavior. A team should be able to answer: what triggers the pipeline, what jobs run, what secrets are used, what artifacts are produced, and what can deploy to production?

Practical design points:

- Keep common jobs reusable.
- Pin third-party actions or images where possible.
- Keep environment secrets out of the repository.
- Use branch and environment protection.
- Make production deployment gates explicit.

### Fundamentals: Branching Strategies

Branching strategy affects CI/CD more than many teams expect. CI/CD is easiest when changes are small and merged often. It becomes harder when branches live for a long time and drift far away from main.

Trunk-based development keeps most work close to main. Developers use short-lived branches or commit directly through strict checks. It pairs well with feature flags because unfinished work can be merged without being exposed to users.

Feature branch workflows are common and practical. The risk is long-lived feature branches. A feature branch should usually be small enough to review and merge quickly.

GitFlow uses long-running branches like `develop`, `release`, and `hotfix`. It can help teams with scheduled releases, but it adds merge complexity and can slow integration.

Release branches are useful when production needs stabilization while main continues moving. The cost is that fixes may need to be cherry-picked or merged across branches.

Theory rule: the longer code waits before integration, the more expensive integration becomes.

### Fundamentals: Artifact Management

An artifact is the deployable output of a pipeline. It may be a Docker image, compiled binary, package, frontend bundle, Helm chart, Lambda zip, SBOM, or release archive.

The strongest artifact rule is build once, promote many times. The artifact tested in staging should be the same artifact deployed to production. Environment-specific values should come from configuration, not from rebuilding.

Artifact management gives traceability. When production has a problem, the team should know the exact commit, build number, dependency set, container digest, and deployment time.

Important concepts:

- Version tags identify human-readable releases.
- Immutable digests identify exact artifacts.
- Registries store and distribute artifacts.
- Retention policies keep useful history without infinite storage growth.
- Signing and checksums help verify integrity.

### Fundamentals: Environment Promotion

Environment promotion is the movement of one release candidate through environments such as dev, test, staging, and production. Promotion is useful only when environments are close enough that passing one environment gives confidence in the next.

The artifact should stay the same. Configuration should change. For example, staging and production may use different database URLs, API keys, scaling settings, and domains, but they should run the same application image.

Promotion gates should match risk:

- Dev may deploy automatically.
- Test may require automated tests.
- Staging may require smoke and integration tests.
- Production may require approval, change window, or progressive rollout.

### Pipeline Stages: Source Checkout

Source checkout fetches the exact code revision the pipeline will validate. It sounds simple, but traceability starts here. The pipeline should record commit SHA, branch, tag, pull request ID, author, and trigger event.

For secure pipelines, checkout must avoid untrusted code gaining access to privileged secrets. Pull requests from forks, for example, should not automatically receive production credentials.

### Pipeline Stages: Build

The build stage turns source code into runnable software. For compiled languages this means compilation. For frontend apps it may mean bundling static files. For containers it means producing an image. For serverless it may mean packaging a zip file.

A good build is repeatable. Given the same source, dependency lockfile, base image, and build inputs, it should produce the same result or an equivalent result. Builds should avoid pulling floating dependency versions without lockfiles because a build can break even when your code did not change.

Build output should include metadata: commit SHA, build number, version, build time, and source repository.

### Pipeline Stages: Unit Testing

Unit tests verify small pieces of logic in isolation. They are the first serious safety net in CI because they are fast and precise. A unit test failure usually points directly to a small part of the code.

Unit tests should run on every pull request. If they are slow, unreliable, or dependent on external services, they stop being useful as unit tests.

Good unit tests:

- Run quickly.
- Are deterministic.
- Do not require network access.
- Test behavior instead of implementation details.
- Make failures easy to understand.

### Pipeline Stages: Integration Testing

Integration tests verify that components work together. They catch problems that unit tests cannot: database queries, migrations, queue behavior, API contracts, cache integration, authentication configuration, and service boundaries.

Integration tests are slower than unit tests, so teams often split them into fast integration tests for pull requests and broader integration tests for staging or nightly runs.

Modern integration testing often uses disposable dependencies: Docker Compose, Testcontainers, temporary databases, local cloud emulators, or ephemeral namespaces.

### Pipeline Stages: End-to-End Testing

End-to-end tests simulate user or system workflows through the full stack. They are valuable because they test what users actually experience, but they are expensive because they can be slow and flaky.

Good E2E strategy focuses on critical paths:

- Sign up or login.
- Create or update the main business object.
- Checkout or payment flow.
- Main admin operation.
- One read path and one write path.

Do not try to cover every edge case with E2E tests. Use unit and integration tests for most logic, then use E2E tests to confirm the system is assembled correctly.

### Pipeline Stages: Static Analysis

Static analysis checks code without running the application. It includes linting, formatting, type checking, complexity checks, dead code detection, and code quality rules.

Static analysis gives fast feedback and catches classes of errors before tests run. Type checking can catch wrong function calls, missing fields, invalid imports, and unreachable logic. Linters catch style and correctness issues.

Good static analysis rules should be strict enough to help but not so noisy that developers ignore them.

### Pipeline Stages: Dependency Scanning

Dependency scanning checks third-party libraries for known vulnerabilities and licensing issues. This is important because modern applications depend on many packages maintained outside the team.

Dependency scanning should understand severity and exploitability. Not every vulnerability is equally urgent. A critical vulnerability in a runtime dependency exposed to internet input is more urgent than a low-severity issue in a development-only tool.

Useful practices:

- Keep lockfiles committed.
- Automate dependency update pull requests.
- Fail builds only for agreed severity thresholds.
- Track exceptions with expiration dates.

### Pipeline Stages: Container Scanning

Container scanning checks Docker or OCI images for vulnerable OS packages, language dependencies, secrets, and insecure configuration. It matters because a container image includes more than application code.

Scanning should happen after the image is built and before it is deployed. Base images should be updated regularly. Smaller images reduce attack surface and scan noise.

Good container image practice:

- Use trusted base images.
- Avoid running as root.
- Do not bake secrets into images.
- Pin image digests for critical base images.
- Rebuild images when base image vulnerabilities are fixed.

### Pipeline Stages: Packaging

Packaging creates the final deployable unit. It should gather compiled code, runtime dependencies, static assets, migrations, metadata, and configuration templates as needed.

Packaging should not include environment secrets. A package should be portable between environments, with runtime configuration supplied separately.

### Pipeline Stages: Publishing Artifacts

Publishing uploads the artifact to a registry or repository. This creates a stable source for deployment. Deployments should pull from the artifact store, not rebuild from source.

Publishing should be protected. A malicious or accidental overwrite of a production artifact can be a serious supply chain problem. Prefer immutable tags or deploy by digest for containers.

### Pipeline Stages: Deployment

Deployment changes the running environment to a new desired version. Deployment may update Kubernetes Deployments, ECS services, EC2 Auto Scaling Groups, Lambda functions, static sites, databases, or infrastructure stacks.

Deployment is not complete when files are copied or pods are created. It is complete when the new version is healthy, reachable, observable, and serving correctly.

Deployment must account for:

- Traffic routing.
- Health checks.
- Config and secrets.
- Database migrations.
- Backward compatibility.
- Rollback or roll-forward.
- Monitoring after release.

### Pipeline Stages: Smoke Testing

Smoke tests are quick post-deployment checks that prove the most basic system behavior works. They should be fast and reliable. Smoke tests are not full regression tests.

Examples:

- `/health` returns healthy.
- Application version matches expected build.
- Login works.
- One important API write succeeds.
- One important API read succeeds.
- Background worker can consume a test message.

### Pipeline Stages: Rollback

Rollback restores a previous working version. It is one of the most important production safety mechanisms, but it only works if the previous version is still compatible with the current environment and data.

Rollback is easy for stateless application code and hard for destructive database changes. If the new release drops a column that the old release needs, rollback may fail. This is why safe migration patterns are central to CI/CD.

Rollback should be automated and tested. A rollback plan that has never been practiced is only a hope.

### Deployment Strategies: Recreate Deployment

Recreate deployment shuts down the old version before starting the new version. In Kubernetes, this maps to the `Recreate` strategy where existing pods are removed before new pods are created.

This is the simplest strategy and the easiest to reason about. Only one version runs at a time, so compatibility between old and new versions is less of a concern. The cost is downtime.

Use recreate when downtime is acceptable or unavoidable:

- Internal admin tools.
- Development environments.
- Single-instance apps.
- Batch systems.
- Systems that cannot safely run two versions at once.

Avoid recreate for public production systems that require high availability.

### Deployment Strategies: Rolling Deployment

Rolling deployment replaces old instances gradually with new instances. Kubernetes uses rolling updates as the default Deployment strategy and controls the rollout with values such as `maxUnavailable` and `maxSurge`.

Rolling deployment works well for stateless replicated services. The load balancer continues sending traffic to healthy instances while old instances are replaced.

The main theory issue is version overlap. During the rollout, old and new versions run at the same time. That means APIs, database schema, message formats, caches, and clients must tolerate both versions.

Example:

- 10 instances run version 1.
- Start 2 instances of version 2.
- Wait until they are ready.
- Remove 2 instances of version 1.
- Repeat until all instances run version 2.

Rolling deployment requires:

- More than one instance for zero downtime.
- Readiness checks.
- Backward-compatible changes.
- Enough capacity during rollout.
- Monitoring for partial failure.

### Deployment Strategies: Blue-Green Deployment

Blue-green deployment uses two production-capable environments. Blue is usually the current live environment. Green receives the new version. After testing green, traffic switches from blue to green.

The key advantage is fast rollback: switch traffic back to blue if green fails. The key cost is duplicate infrastructure. Blue-green can also be difficult when there is one shared database because both environments must remain compatible with the same data.

Blue-green flow:

1. Blue serves users.
2. Green is deployed separately.
3. Green receives smoke tests and synthetic checks.
4. Traffic moves to green through a load balancer, service selector, DNS, or routing layer.
5. Blue stays warm for a short rollback window.
6. Blue is removed or becomes the next idle environment.

Important caution: not every routing mechanism switches traffic atomically. Some controllers and load balancers have propagation delays. Keep the old environment alive long enough for traffic routing to settle.

### Deployment Strategies: Canary Deployment

Canary deployment exposes the new version to a small amount of production traffic before a full rollout. The name comes from using a small early signal to detect danger.

The canary can be based on traffic percentage, request headers, regions, users, tenants, or service instances. Argo Rollouts, service meshes, load balancers, and feature flag systems can all support canary behavior.

Common rollout plan:

- 1 percent for 10 minutes.
- 5 percent for 20 minutes.
- 25 percent for 30 minutes.
- 50 percent for 30 minutes.
- 100 percent after metrics remain healthy.

Canary requires clear success and failure signals:

- Error rate.
- Latency p95 and p99.
- Saturation.
- Business transaction success.
- Queue depth.
- Logs and exception counts.

Canary is weaker if traffic is low. A 1 percent canary on a low-traffic service may not produce enough signal to detect failures.

### Deployment Strategies: User-ID Targeted Deployment

User-ID targeted deployment releases a version or feature to specific users based on identity. It is usually implemented with feature flags or request routing. The important idea is deterministic targeting: the same user should consistently see the same behavior.

Common targeting methods:

- Exact user IDs.
- Employee accounts.
- Beta tester group.
- Tenant ID.
- Region.
- Plan type.
- Stable hash of user ID for percentage rollout.

Stable hashing matters. If a system randomly assigns users on every request, users can bounce between old and new experiences. A hash of user ID gives a consistent cohort.

Use user-targeted rollout for:

- Beta launches.
- Internal dogfooding.
- SaaS tenant-by-tenant rollout.
- Risky UI changes.
- Features that require customer enablement.

Risks:

- More code paths must be tested.
- User support becomes harder if users see different behavior.
- Feature flags can become permanent clutter.
- Client-side flags can expose hidden features if sensitive checks are not enforced server-side.

### Deployment Strategies: Feature-Flag Deployment

Feature flags separate deployment from release. Code can be deployed to production while a feature remains disabled. Later, the feature can be enabled without redeploying.

Feature flags are one of the most important techniques for Continuous Delivery because they let teams merge incomplete or risky work safely. They also provide kill switches for production issues.

Types of feature flags:

- Release flags: hide incomplete features.
- Experiment flags: support A/B tests.
- Operational flags: disable expensive or risky behavior.
- Permission flags: enable features for specific plans or users.
- Migration flags: switch between old and new implementations.

Flag discipline is essential. Every temporary flag needs an owner and removal date. Otherwise the codebase accumulates stale branches and test combinations.

### Deployment Strategies: Dark Launch

Dark launch means deploying code that is active internally but invisible to users. The system may execute new logic, populate new data structures, warm caches, or call a new service, but the user-facing result does not change.

Use dark launch to validate:

- Performance of new backend logic.
- Data correctness.
- Infrastructure capacity.
- Internal event flows.
- New search or recommendation systems.

Dark launch is not risk-free. Hidden code still consumes resources and can cause side effects. It should have limits, monitoring, and a kill switch.

### Deployment Strategies: Shadow Deployment

Shadow deployment copies real production traffic to a new version while the old version still handles the real response. The new version's response is ignored or compared offline.

This is useful for rewrites and high-risk backend changes. It tests the new system against real traffic patterns without exposing users to its responses.

Shadow deployment must prevent side effects. If the shadow service processes payments, sends emails, writes to production databases, or publishes real events, it can cause serious damage. Shadow systems should use read-only mode, isolated data stores, or side-effect blocking.

### Deployment Strategies: A/B Testing Deployment

A/B testing compares variants to measure product or business outcomes. It is different from canary deployment. Canary asks, is this safe? A/B testing asks, which version performs better?

Good A/B tests require:

- Defined hypothesis.
- Stable user assignment.
- Clear success metric.
- Guardrail metrics.
- Enough sample size.
- Avoiding biased cohorts.

A/B testing often uses feature flags, but the release decision should be based on experiment results rather than only technical health.

### Deployment Strategies: Ring Deployment

Ring deployment releases changes to ordered groups. Each ring has more users or higher risk than the previous one.

Example:

- Ring 0: developers.
- Ring 1: internal employees.
- Ring 2: trusted beta customers.
- Ring 3: 5 percent of public users.
- Ring 4: all users.

Ring deployment is useful for large products, enterprise SaaS, operating systems, mobile apps, and platforms with many customer segments. It combines product rollout discipline with operational monitoring.

### Deployment Strategies: Immutable Deployment

Immutable deployment creates new infrastructure or runtime units instead of modifying existing ones. Containers, AMIs, and new Auto Scaling Group launch templates are common examples.

The theory is simple: do not patch production servers by hand. Build a new version, launch it, shift traffic, and remove the old version. This reduces configuration drift and makes environments more reproducible.

Immutable deployment works well with blue-green and rolling strategies. It requires strong automation and artifact management.

### Deployment Strategies: Progressive Delivery

Progressive delivery is the controlled release of changes in stages, using automation and telemetry to decide whether to continue. It combines canary, feature flags, user targeting, traffic shifting, automated analysis, and rollback.

Progressive delivery is not just "deploy slowly." It is "deploy gradually based on evidence." The system should check metrics and stop promotion if the release becomes unhealthy.

Typical progressive delivery loop:

1. Deploy a small slice.
2. Observe technical and business metrics.
3. Promote if healthy.
4. Pause or roll back if unhealthy.
5. Continue until full release.

### Environments: Development

Development environments are for active engineering work. They can be local, shared, or cloud-hosted. They favor speed and flexibility over production parity.

The main risk is assuming dev success means production safety. Development often has smaller data, looser security, different dependencies, and different traffic patterns.

### Environments: Testing

Testing environments run automated or manual verification. They should use safe test data. They may be rebuilt frequently to avoid stale state.

The theory goal is isolation. Tests should not depend on hidden state from previous runs. A test environment that cannot be reset becomes unreliable.

### Environments: Staging

Staging is the final rehearsal before production. It should be as production-like as practical: same deployment mechanism, similar configuration, similar infrastructure shape, realistic data patterns, and production-like integrations where safe.

Staging is useful for catching release, integration, and configuration problems. It is not a perfect predictor of production because production has real users, real traffic, real scale, and real external behavior.

### Environments: Production

Production is where real users and business operations happen. CI/CD for production needs stricter controls than lower environments.

Production requirements:

- Access control.
- Auditing.
- Monitoring.
- Backup and restore.
- Incident response.
- Deployment traceability.
- Change approval when needed.
- Rollback or roll-forward path.

### Environments: Preview Environments

Preview environments are created for a pull request or branch so reviewers can test a change before merge. They are very useful for frontend, API, and product review.

Preview environments should be cheap, automatic, and disposable. They should not contain production secrets or unsafe data.

### Environments: Ephemeral Environments

Ephemeral environments are temporary environments created on demand and destroyed after use. Preview environments are one type. Integration test environments are another.

The theory benefit is clean state. Instead of fighting a shared test environment, each pipeline run gets an isolated environment.

The cost is infrastructure automation complexity and cloud spend if cleanup fails.

### Infrastructure: Infrastructure as Code

Infrastructure as Code defines infrastructure through version-controlled code or templates. Examples include Terraform, CloudFormation, CDK, Pulumi, and Bicep.

IaC brings software engineering practices to infrastructure: review, testing, versioning, reuse, and automated deployment. CI/CD for IaC should validate syntax, run security scans, show a plan, require approval for risky changes, apply changes, and detect drift.

Important IaC concepts:

- Desired state: the code describes what should exist.
- State: the tool tracks what currently exists.
- Plan: proposed changes before applying.
- Drift: real infrastructure differs from code.
- Modules or constructs: reusable infrastructure components.

Common IaC risks:

- Accidentally deleting production resources.
- State file corruption or leakage.
- Overly broad permissions.
- Manual console changes causing drift.
- Reusing modules without understanding defaults.

### Infrastructure: GitOps

GitOps uses Git as the source of truth for deployment and infrastructure state. Instead of a pipeline directly pushing changes into a cluster, a controller watches Git and reconciles the environment to match.

GitOps is common in Kubernetes with tools such as Argo CD and Flux. The repository contains manifests, Helm values, or Kustomize overlays. The controller applies changes and reports drift.

Benefits:

- Clear audit trail.
- Easier rollback through Git history.
- Pull-based deployment model.
- Environment state visible in repositories.

Risks:

- Secrets need careful handling.
- Repository design can become complex.
- Emergency manual changes may be overwritten by reconciliation.

### Infrastructure: Kubernetes Deployments

Kubernetes Deployments manage stateless replicated pods and ReplicaSets. They support declarative updates, rollout status, rollback, scaling, pausing, and rollout history.

For CI/CD, Kubernetes gives a standard deployment target, but the pipeline still needs to handle image publishing, manifest updates, config, secrets, readiness checks, and observability.

Important Kubernetes deployment settings:

- `replicas`: desired pod count.
- `strategy.type`: `RollingUpdate` or `Recreate`.
- `maxUnavailable`: how much capacity can be unavailable.
- `maxSurge`: how much temporary extra capacity is allowed.
- `readinessProbe`: whether a pod can receive traffic.
- `livenessProbe`: whether a pod should be restarted.
- `revisionHistoryLimit`: rollback history.

### Infrastructure: Helm

Helm packages Kubernetes resources into charts. A chart contains templates and values. Different environments can use the same chart with different values files.

Helm is useful when raw Kubernetes YAML becomes repetitive. It can package services, deployments, ingress, config maps, service accounts, autoscaling, and other resources.

Good Helm practice:

- Keep chart values understandable.
- Avoid excessive template magic.
- Version charts.
- Validate rendered manifests.
- Use separate values per environment.

### Infrastructure: Database Migrations

Database migrations change schema or data. They are often the hardest part of CI/CD because application rollback and database rollback are not symmetric.

The safest approach is expand and contract:

1. Expand: add new tables, columns, indexes, or nullable fields without breaking old code.
2. Deploy code that can work with both old and new structures.
3. Backfill or migrate data.
4. Switch reads and writes.
5. Contract: remove old columns or tables only after no running code needs them.

Avoid:

- Dropping columns in the same deployment that stops using them.
- Renaming columns without compatibility handling.
- Long locking migrations during peak traffic.
- Migrations that cannot be resumed safely.

### Infrastructure: Configuration Management

Configuration controls how the same artifact behaves in each environment. Examples include database URL, service endpoint, log level, feature flags, region, queue names, and scaling settings.

Configuration should be external to the artifact. If a production URL is compiled into a build, the artifact cannot safely move across environments.

Good config principles:

- Separate config from code.
- Validate required config at startup.
- Keep secrets in secret stores, not normal config.
- Track config changes where possible.
- Avoid undocumented manual config edits.

### Infrastructure: Secrets Management

Secrets include passwords, API keys, private keys, tokens, signing keys, and certificates. CI/CD systems often need secrets to fetch dependencies, publish artifacts, deploy infrastructure, and call cloud APIs.

Secrets are high-risk because CI/CD pipelines execute code. If untrusted code can read deployment secrets, the pipeline becomes an attack path.

Good practices:

- Use short-lived credentials where possible.
- Use role-based access instead of long-lived static keys.
- Mask secrets in logs.
- Restrict secrets by environment.
- Do not expose production secrets to pull requests from forks.
- Rotate secrets.

### Security: Secret Scanning

Secret scanning detects accidental commits of credentials. It should run on pull requests and main branches, but prevention is better than detection. Developers should use environment variables, local secret stores, or cloud secret managers rather than copying secrets into files.

When a secret is leaked, deleting it from Git is not enough. Assume it was exposed and rotate it.

### Security: SAST

Static Application Security Testing scans source code for security weaknesses. It can catch injection risks, unsafe deserialization, hardcoded secrets, path traversal, insecure cryptography, and framework misuse.

SAST should be tuned. Too many false positives cause alert fatigue. High-confidence issues in sensitive code should block merges.

### Security: DAST

Dynamic Application Security Testing scans a running application. It behaves more like an attacker or external tester. DAST can find issues related to authentication, headers, injection, exposed endpoints, and runtime behavior.

DAST is usually slower than SAST and often runs against staging or scheduled environments instead of every small pull request.

### Security: SBOM

A Software Bill of Materials lists the components inside a software artifact. It helps teams answer: are we affected by this newly disclosed vulnerability?

An SBOM is most useful when it is tied to a specific artifact version and stored with release metadata. It should include direct and transitive dependencies where possible.

### Security: Artifact Signing

Artifact signing proves that an artifact came from a trusted build process and has not been modified. This is part of software supply chain security.

Signing is especially important when artifacts move through registries, multiple environments, or third-party distribution. Verification should happen before deployment, not only during publishing.

### Security: Policy as Code

Policy as Code expresses rules in machine-enforceable form. Instead of relying on humans to remember every rule, the pipeline checks policies automatically.

Examples:

- Containers must not run as root.
- Kubernetes workloads must set CPU and memory limits.
- Terraform must not create public storage buckets.
- Production deployments require approval.
- Images must come from approved registries.
- Critical vulnerabilities block release.

Policy as Code should be transparent. Developers need to know which rule failed and how to fix it.

### Operations: Health Checks

Health checks allow deployment systems to know whether a new version can receive traffic.

Common types:

- Startup check: has the app finished booting?
- Liveness check: should the process be restarted?
- Readiness check: can this instance receive production traffic?
- Deep health check: can critical dependencies be reached?

Readiness checks are essential for rolling deployments. Without them, a load balancer may send traffic to an instance before it is ready.

### Operations: Observability

Observability is the ability to understand system behavior from outputs such as logs, metrics, and traces. CI/CD without observability is dangerous because the team cannot tell whether a deployment is healthy.

Deployment observability should answer:

- What version is running?
- When was it deployed?
- Which commit introduced it?
- Did error rate change?
- Did latency change?
- Are dependencies failing?
- Are business transactions succeeding?

Metrics should be compared before and after deployment, especially during canary and progressive delivery.

### Operations: Release Approvals

Release approvals are human gates in a pipeline. They are useful when changes are high risk, regulated, customer-visible, or operationally sensitive.

Approval should not replace automation. It should complement automated evidence. A useful approval screen shows version, diff, test results, security scan status, deployment plan, migration risk, and rollback plan.

### Operations: Change Management

Change management records what changed, why it changed, who approved it, when it happened, and how to recover if it fails.

Lightweight change management can be generated from pull requests, pipeline runs, commit SHAs, issue links, and deployment records. Heavy manual change processes often slow teams without improving safety unless they are connected to real risk controls.

### Operations: Incident Rollback

Incident rollback is the emergency version of rollback. The goal is to restore service quickly, not to finish root cause analysis during the outage.

Good incident rollback requires:

- Clear trigger conditions.
- Known previous good version.
- Automated rollback command.
- Database compatibility.
- Communication channel.
- Post-rollback verification.

If rollback is unsafe, use roll-forward.

### Operations: Roll Forward

Roll forward means deploying a new fix instead of returning to the previous version. This is common when the bad release included a database migration or data change that cannot easily be reversed.

Roll-forward works when the team can produce a fix quickly and the deployment pipeline is fast. It is risky if the fix is rushed through weak testing.

### Metrics: DORA Metrics

DORA metrics measure software delivery performance. The classic four are deployment frequency, lead time for changes, change failure rate, and time to restore service. They balance speed and stability.

Do not use DORA metrics to punish teams. They are system-level improvement metrics. If lead time is slow, find the bottleneck. If change failure rate is high, improve testing, rollout safety, and observability.

### Metrics: Lead Time

Lead time for changes measures how long it takes for a committed change to reach production. It includes review time, queue time, build time, test time, approval time, and deployment time.

Long lead time often means large batches, slow review, slow tests, manual release processes, or environment bottlenecks.

### Metrics: Deployment Frequency

Deployment frequency measures how often production receives changes. Higher frequency usually means smaller batches, faster feedback, and lower release anxiety.

High deployment frequency is not automatically good. It must be paired with low change failure rate and fast recovery.

### Metrics: Change Failure Rate

Change failure rate measures the percentage of production changes that cause incidents, degraded service, rollback, hotfix, or urgent remediation.

This is a quality and stability signal. A team that deploys frequently but breaks production often is moving risk faster, not delivering better.

### Metrics: MTTR

MTTR means Mean Time To Recovery or restore service, depending on wording. It measures how long it takes to recover from production failure.

MTTR improves with observability, small changes, clear ownership, rollback automation, incident practice, and simple architecture.

## 13. CI/CD Tooling Map

CI/CD concepts are portable. Tools differ, but the same theory applies.

| Area | Common Tools | What They Do |
| --- | --- | --- |
| Pipeline orchestration | GitHub Actions, GitLab CI, Jenkins, CircleCI, Azure DevOps, Buildkite | Run CI/CD workflows |
| Artifact registry | Docker Hub, ECR, GHCR, Artifactory, Nexus, CodeArtifact | Store build outputs |
| Kubernetes delivery | kubectl, Helm, Kustomize, Argo CD, Flux, Argo Rollouts | Deploy and manage Kubernetes apps |
| Infrastructure as Code | Terraform, CloudFormation, CDK, Pulumi | Provision infrastructure |
| Security scanning | CodeQL, Semgrep, Trivy, Grype, Snyk, OWASP ZAP | Find vulnerabilities and misconfigurations |
| Feature flags | OpenFeature providers, LaunchDarkly, Unleash, Flagsmith, ConfigCat | Control release by user, cohort, or environment |
| Observability | CloudWatch, Datadog, Grafana, Prometheus, OpenTelemetry, New Relic | Measure deployment health |

## 14. Deployment Strategy Pros and Cons

| Strategy | Best For | Pros | Cons |
| --- | --- | --- | --- |
| Recreate | Simple apps, dev, internal tools | Simple; only one version runs | Downtime; weak production safety |
| Rolling | Normal replicated services | Low or zero downtime; efficient infrastructure use | Old and new versions overlap; needs compatibility |
| Blue-green | High-value services needing fast rollback | Test new environment before traffic; quick traffic rollback | Duplicate infrastructure; database compatibility still hard |
| Canary | Risky changes, high-traffic services | Limits blast radius; uses real production signal | Needs traffic splitting and strong metrics |
| User-ID targeted | Beta users, tenants, internal rollout | Precise control; stable cohorts; good for SaaS | More code paths; requires flag discipline |
| Feature flag | Separating deploy from release | Runtime control; kill switch; supports experiments | Flag debt; more testing combinations |
| Dark launch | Hidden backend validation | Tests production behavior before visible launch | Hidden side effects and resource cost |
| Shadow | Rewrites, API migration, performance validation | Tests real traffic without user-facing response | Must block side effects; can double load |
| A/B testing | Product experiments | Measures business outcome | Requires statistical care and stable assignment |
| Ring | Large orgs, enterprise rollout | Structured expansion by risk group | Slower; requires cohort management |
| Immutable | Reproducible infrastructure | Reduces drift; clean rollback model | Requires strong automation |
| Progressive delivery | Mature production systems | Automated gradual rollout based on evidence | Needs reliable telemetry and automation |

## 15. CI/CD Pros and Cons

### Pros

- Faster feedback because every change is tested automatically.
- Smaller releases because teams integrate more frequently.
- Better reliability when deployments are repeatable and observable.
- Easier rollback when artifacts and versions are traceable.
- Less manual release work.
- Better security when scanning and policy checks are automated.
- Better auditability through pipeline history and deployment records.

### Cons

- Bad tests create false confidence.
- Flaky tests slow teams and reduce trust.
- Pipeline maintenance becomes real engineering work.
- Secrets and permissions can become serious security risks.
- Complex deployment strategies require observability and operational skill.
- Feature flags can create long-term code complexity.
- Database changes remain hard even with strong automation.

## 16. Quick Comparison: Delivery vs Deployment

| Concept | Meaning | Production Release |
| --- | --- | --- |
| Continuous Integration | Merge and validate code frequently | Not necessarily deployed |
| Continuous Delivery | Always keep code releasable | Manual approval may release it |
| Continuous Deployment | Automatically release passing changes | Automatic after checks pass |

## 17. Short Version

CI/CD is the system for safely moving code from commit to production. CI checks whether a change is safe to merge. Continuous Delivery prepares a tested artifact so it can be released when the team chooses. Continuous Deployment automatically releases every passing change to production.

A good pipeline usually does this: checkout code, install dependencies, run static checks, run tests, build one artifact, scan it, publish it, deploy it to staging, smoke test it, then deploy to production using a strategy such as rolling, blue-green, canary, or user-targeted rollout.

The safest teams deploy small changes often, build once and promote the same artifact, use feature flags for risky features, keep database changes backward-compatible, watch production metrics after release, and practice rollback or roll-forward before incidents happen.

In one sentence: CI/CD is not just automation; it is the discipline of making software release small, tested, traceable, reversible, and observable.

## 18. References Used

- IBM CI/CD overview: https://www.ibm.com/think/topics/ci-cd
- Kubernetes Deployments and rolling updates: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ and https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/
- Argo Rollouts canary strategy: https://argoproj.github.io/argo-rollouts/features/canary/
- Argo Rollouts blue-green strategy: https://argoproj.github.io/argo-rollouts/features/bluegreen/
- DORA Accelerate State of DevOps report: https://dora.dev/research/2022/dora-report/2022-dora-accelerate-state-of-devops-report.pdf
- OWASP CI/CD Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html
- OpenFeature evaluation context and targeting key: https://openfeature.dev/docs/reference/concepts/evaluation-context
- Martin Fowler feature toggles: https://martinfowler.com/articles/feature-toggles.html

## 19. Final Short Recap

CI/CD is how teams turn code changes into reliable production releases. CI proves a change can be integrated. Continuous Delivery proves a release is ready. Continuous Deployment automatically releases it when checks pass.

Use rolling deployments for normal replicated services, blue-green when fast traffic rollback matters, canary when you want to test with a small amount of real traffic, and user-ID or feature-flag rollout when you want to expose changes to specific users, tenants, or cohorts.

The core rule is simple: build once, test automatically, deploy gradually when risk is high, observe production, and keep rollback or roll-forward ready.
