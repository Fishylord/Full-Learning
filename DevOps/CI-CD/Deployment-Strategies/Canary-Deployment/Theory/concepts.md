# Canary Deployment Theory

## What It Is

Canary deployment releases a new version to a small portion of production traffic first. If the new version behaves well, more traffic is gradually shifted to it.

The goal is to detect production issues with limited blast radius.

## How It Works

1. Current version serves most traffic.
2. New version receives a small percentage of traffic.
3. Metrics and logs are observed.
4. If healthy, traffic share increases.
5. If unhealthy, traffic returns to the old version.
6. The rollout continues until the new version receives all traffic.

## Common Rollout Steps

Example:

- 1 percent for 10 minutes.
- 5 percent for 20 minutes.
- 25 percent for 30 minutes.
- 50 percent for 30 minutes.
- 100 percent after metrics stay healthy.

The exact percentages depend on traffic volume, risk, and confidence.

## Why It Exists

Some bugs only appear under real production traffic, real users, real integrations, or real scale. Canary deployment allows teams to learn from production without exposing everyone at once.

## When To Use

Use canary deployment for:

- High-traffic services.
- Risky backend changes.
- Performance-sensitive releases.
- New infrastructure paths.
- API behavior changes.
- Systems with strong observability.

## When To Avoid

Avoid or be careful with canary when:

- Traffic volume is too low to produce useful signal.
- You cannot split traffic reliably.
- Metrics are weak or delayed.
- The new version causes irreversible side effects.
- User experience must be identical for all users at the same time.

## Canary Signals

Good canary decisions require health signals:

- Error rate.
- Latency p95 and p99.
- CPU and memory.
- Queue depth.
- Database load.
- Exception count.
- Business transaction success.
- Payment failures.
- Login failures.

Do not rely only on "service is up." A service can be up and still broken.

## Traffic Splitting Methods

Canary can be controlled by:

- Load balancer weights.
- Service mesh routing.
- Kubernetes rollout controllers.
- API gateway routing.
- Feature flags.
- Region or tenant selection.
- Header-based routing.

## Example

A payment API deploys version 2 to 2 percent of traffic. The pipeline watches error rate, payment success rate, and p99 latency. If metrics remain healthy for 15 minutes, it increases to 10 percent. If payment failures rise, it rolls back to version 1.

## Pros

- Limits blast radius.
- Tests with real production traffic.
- Supports automated metric-based rollout.
- Good for high-risk changes.
- Can catch performance problems early.

## Cons

- Requires traffic splitting.
- Requires strong observability.
- Low traffic may not produce useful signal.
- Debugging mixed traffic can be harder.
- Some bugs appear only after larger rollout.

## Short Summary

Canary deployment exposes a new version to a small amount of production traffic first. It is excellent for risk reduction when traffic splitting and observability are strong.

## Deep Technical Notes

### Control Plane vs Data Plane

Canary deployment has two parts:

- Control plane: decides rollout steps, timing, and promotion.
- Data plane: actually routes traffic between old and new versions.

Control plane examples:

- Argo Rollouts.
- Flagger.
- Spinnaker.
- CodeDeploy.
- Custom CI/CD pipeline.

Data plane examples:

- Load balancer weights.
- Service mesh.
- Ingress controller.
- API gateway.
- Feature flag evaluation.

Separating these concepts helps debugging. A rollout controller may say "10 percent canary," but the routing layer must actually enforce it.

### Traffic Percentage Is Not User Percentage

Ten percent traffic does not always mean ten percent users.

If routing is per request, one user may hit both old and new versions. That can be bad for user experience.

If routing is per user or session, the same users consistently receive the canary.

Choose based on release type:

- Backend performance change: request-level traffic split may be fine.
- UI or workflow change: user-sticky assignment is better.
- Tenant-level SaaS change: tenant-sticky assignment may be required.

### Metric-Based Promotion

A serious canary rollout should define promotion and abort rules before deployment.

Example abort rules:

- Error rate is more than 2x baseline.
- p99 latency increases by more than 20 percent.
- Payment success drops below threshold.
- Critical log errors exceed threshold.
- CPU saturation exceeds threshold.

Example promotion rule:

- Canary runs for 15 minutes.
- Minimum request count reached.
- Error and latency within threshold.
- No critical alerts triggered.

### Statistical Signal

Canary decisions need enough traffic to be meaningful.

If a service receives 100 requests per hour, a 1 percent canary receives about 1 request per hour. That is not enough to detect most problems.

Low-traffic systems may need:

- Longer canary windows.
- Synthetic traffic.
- Larger initial percentage.
- Staging validation.
- Blue-green instead of canary.

### Baseline Comparison

Compare canary against the stable version at the same time when possible. Absolute thresholds can be misleading.

Example:

- All versions have high latency because the database is slow.
- Canary should not be blamed unless it is worse than baseline.

Useful comparison:

- Canary p95 latency vs stable p95 latency.
- Canary error rate vs stable error rate.
- Canary business conversion vs stable conversion.

### Dependency and Regional Bias

A canary routed to one region may hide bugs that only appear elsewhere. A canary routed to one customer segment may not represent all traffic.

Design canary cohorts carefully:

- Include realistic traffic types.
- Avoid only internal users for performance validation.
- Avoid only one region if dependencies vary by region.
- Segment metrics by canary and stable version.

### Database and Side Effects

Canary does not isolate database writes. If the new version writes bad data, the old version may read it too.

Risk examples:

- New version writes invalid enum.
- New version changes customer status incorrectly.
- New version publishes malformed events.
- New version corrupts cache entries.

Mitigations:

- Idempotency keys.
- Schema validation.
- Backward-compatible writes.
- Limited write access for high-risk canaries.
- Kill switch.

### Rollback and Abort

Canary rollback usually means route all traffic back to stable. But also consider cleanup:

- Stop new workers.
- Stop new event producers.
- Clear incompatible cache keys.
- Disable feature flags.
- Quarantine bad data.
- Drain canary pods.

### Practical Lab Ideas

- Use Nginx weighted upstreams to route 95/5 traffic.
- Use Kubernetes plus Argo Rollouts for staged canary steps.
- Build a script that promotes only when p95 latency and error rate are healthy.
- Simulate low traffic and show why 1 percent canary gives no useful signal.
