# Feature-Flag Deployment Theory

## What It Is

Feature-flag deployment separates code deployment from feature release.

The code is deployed to production, but the feature can be turned on or off at runtime through a flag.

## How It Works

1. Developers add new code behind a flag.
2. The code is deployed while the flag is off.
3. The team enables the flag for internal users, beta users, or a percentage of users.
4. Metrics and feedback are monitored.
5. The flag is expanded, disabled, or removed.

## Why It Exists

Without feature flags, deployment and release happen together. That means every deployed change is immediately visible or active.

Feature flags allow teams to deploy safely before releasing. They support gradual rollout, kill switches, experiments, and trunk-based development.

## Types of Feature Flags

Common flag types:

- Release flags: hide incomplete features.
- Experiment flags: support A/B testing.
- Operational flags: disable risky or expensive behavior.
- Permission flags: enable behavior for plans, roles, or tenants.
- Migration flags: switch between old and new implementations.
- Kill switches: quickly disable failing functionality.

## When To Use

Use feature-flag deployment for:

- Continuous Delivery.
- Trunk-based development.
- Risky features.
- User-targeted rollout.
- A/B testing.
- Emergency disable switches.
- Complex migrations.

## When To Avoid

Avoid or be careful when:

- The flag will never be cleaned up.
- The feature changes database schema destructively.
- The flag controls security-sensitive behavior only on the client.
- Too many flags interact in unclear ways.
- Testing all combinations becomes impossible.

## Flag Lifecycle

Healthy flag lifecycle:

1. Create flag with owner and purpose.
2. Deploy code behind the flag.
3. Enable gradually.
4. Observe behavior.
5. Make final decision.
6. Remove dead branch and delete flag.

Temporary release flags should not live forever.

## Testing Strategy

At minimum, test:

- Flag off path.
- Flag on path.
- Permission boundary.
- Rollback or disable behavior.

For important flags, add automated tests for both enabled and disabled states.

## Example

A team builds a new search ranking algorithm. The code is deployed with `new_search_ranking=false`. Internal users test it first. Then 5 percent of users get it. If search latency increases, the flag is disabled immediately without redeploying.

## Pros

- Separates deploy from release.
- Enables gradual rollout.
- Provides kill switches.
- Supports experiments.
- Helps teams merge work earlier.

## Cons

- Creates flag debt.
- Adds branching logic.
- More test combinations.
- Requires governance and ownership.
- Poorly secured flags can expose behavior.

## Short Summary

Feature-flag deployment lets teams deploy code before releasing functionality. It gives control and safety, but flags must be owned, tested, and removed when finished.

## Deep Technical Notes

### Flag Evaluation Flow

A feature flag system usually has:

- Flag definitions.
- Targeting rules.
- Evaluation context.
- SDK or evaluator.
- Default values.
- Audit log.
- Admin UI or config store.

At runtime, code asks: "for this context, what value should this flag return?"

The result may be boolean, string, number, JSON config, or variant name.

### Default Values and Failure Modes

Flag systems can fail. The app must know what to do if the flag provider is unavailable.

Failure scenarios:

- SDK cannot reach flag service.
- Local cache is stale.
- Flag key is missing.
- Evaluation context is incomplete.
- Network timeout.
- Client receives malformed config.

Every flag should have a safe default. For release flags, default is often off. For operational kill switches, the safe default depends on the feature.

### Flag Storage Models

Common models:

- Remote SaaS flag provider.
- Self-hosted flag service.
- Static config file.
- Environment variables.
- Database-backed rules.
- Edge config.

Remote providers give dynamic control, but local caching matters. A flag check should not add high latency to every request.

### Flag Categories by Lifetime

Short-lived flags:

- Release flags.
- Migration flags.
- Experiment flags.

Long-lived flags:

- Permission flags.
- Product entitlement flags.
- Operational controls.

Short-lived flags should be removed after rollout. Long-lived flags require stronger naming and ownership because they become product logic.

### Flag Debt

Flag debt happens when old flags remain in code after they are no longer needed.

Problems caused by flag debt:

- Dead code.
- Confusing behavior.
- Harder testing.
- Hidden dependencies.
- More branches in business logic.
- Risk of accidentally disabling old paths.

Good governance:

- Owner.
- Creation date.
- Purpose.
- Expected removal date.
- Cleanup task.
- Dashboard of stale flags.

### Testing With Flags

At minimum, test the default path and enabled path.

For critical flags:

- Unit tests for both branches.
- Integration tests for flag on/off.
- Permission tests.
- Rollback tests.
- Production smoke checks for selected cohorts.

Avoid creating many interacting flags where the number of combinations becomes impossible to test.

### Deployment vs Release

Feature flags separate deployment and release:

- Deployment: code is shipped to production.
- Release: users can access the behavior.

This allows trunk-based development. Teams can merge incomplete work behind disabled flags, reducing long-lived branches.

### Migration Flags

Migration flags are useful for switching between old and new implementations.

Example:

- `use_new_payment_provider=false`
- Deploy code supporting old and new providers.
- Enable for internal users.
- Enable for a small cohort.
- Monitor.
- Enable all.
- Remove old provider code later.

The dangerous part is side effects. If old and new providers both charge users, the flag design is broken.

### Practical Lab Ideas

- Build a boolean flag evaluator with safe defaults.
- Add percentage rollout using stable hashing.
- Add audit logging for flag changes.
- Create a stale flag cleanup checklist.
- Write tests for flag on and flag off paths.
