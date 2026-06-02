# Progressive Delivery Theory

## What It Is

Progressive delivery is a release approach where changes are exposed gradually and promoted based on evidence.

It combines techniques such as canary deployment, feature flags, user targeting, ring deployment, traffic shifting, automated analysis, and rollback.

## How It Works

1. Deploy a new version or feature in a limited way.
2. Observe technical and business metrics.
3. Promote to more users or traffic if healthy.
4. Pause, roll back, or disable if unhealthy.
5. Continue until full rollout.

## Why It Exists

Traditional deployment often treats release as a single event. Progressive delivery treats release as a controlled process.

The theory is that confidence should increase through real signals, not assumptions.

## Core Ingredients

Progressive delivery needs:

- Traffic control.
- Feature flags or targeting.
- Observability.
- Automated health checks.
- Clear promotion rules.
- Rollback or disable mechanism.
- Small release increments.

## Common Promotion Signals

Promotion can depend on:

- Error rate.
- Latency.
- Saturation.
- Exception count.
- Business conversion.
- Payment success.
- Queue depth.
- Customer support alerts.

## When To Use

Use progressive delivery for:

- Mature production systems.
- High-traffic services.
- Risky releases.
- SaaS platforms.
- Systems with strong metrics.
- Teams deploying frequently.

## When To Avoid

Avoid or be careful when:

- Observability is weak.
- Traffic is too low for signal.
- Rollout automation is immature.
- Changes cannot be partially released.
- Metrics are delayed or unreliable.

## Difference From Canary

Canary deployment is one technique. Progressive delivery is the broader discipline.

A progressive rollout may use canary traffic, feature flags, user targeting, ring deployment, and automated metric checks together.

## Example

A team releases a new checkout service. It starts with internal users, then 1 percent public traffic, then 10 percent, then selected regions, then full rollout. At each step, the system checks payment success, error rate, latency, and support alerts.

## Pros

- Reduces release risk.
- Supports automated safety decisions.
- Combines product and technical rollout.
- Limits blast radius.
- Works well with frequent deployment.

## Cons

- Requires strong observability.
- More complex than simple deployment.
- Needs reliable traffic control.
- Requires clear ownership of metrics.
- Can be overkill for simple apps.

## Short Summary

Progressive delivery releases changes gradually and promotes them based on evidence. It is the mature combination of canary, feature flags, targeting, metrics, and rollback.

## Deep Technical Notes

### Progressive Delivery as a Control Loop

Progressive delivery is a feedback control loop:

1. Expose a small part of the system to change.
2. Measure the result.
3. Compare against policy.
4. Promote, pause, or roll back.
5. Repeat.

This is different from a pipeline that blindly waits for a timer and then continues. Mature progressive delivery uses health evidence.

### Components

A progressive delivery system usually includes:

- Deployment controller.
- Traffic router.
- Metrics provider.
- Analysis engine.
- Feature flag or targeting system.
- Rollback mechanism.
- Notification or approval system.

Example stack:

- Argo Rollouts as controller.
- Istio or Nginx for traffic routing.
- Prometheus for metrics.
- Analysis templates for success criteria.
- Slack or PagerDuty for notifications.

### Automated Analysis

Automated analysis decides whether to continue.

Metrics might include:

- HTTP 5xx rate.
- p95 and p99 latency.
- Request success rate.
- Queue lag.
- CPU saturation.
- Business transaction rate.
- Custom application health.

Analysis should compare canary and baseline where possible. Absolute metrics alone can mislead when the whole system is unhealthy.

### Rollout Units

Progression can happen by:

- Traffic percentage.
- User cohort.
- Tenant ring.
- Region.
- Availability zone.
- Device channel.
- Feature flag segment.

The rollout unit should match the failure mode. For a regional dependency risk, progress by region. For a SaaS workflow risk, progress by tenant or user cohort.

### Pause Points

Pause points give time for metrics and human observation.

Types:

- Fixed duration pause.
- Manual approval pause.
- Metric analysis pause.
- Business-hours pause.
- Customer validation pause.

For high-risk production systems, automatic promotion should not outrun the time needed for meaningful signal.

### Rollback vs Disable

Progressive delivery may roll back code or disable a feature.

Rollback code when:

- New runtime version is unhealthy.
- Infrastructure change caused failure.
- Performance regression is in deployed code.

Disable flag when:

- Feature behavior is wrong.
- A specific path is failing.
- The code is otherwise stable.

Often both are needed.

### SLO-Aware Delivery

Progressive delivery works best with service-level objectives.

Example:

- Availability SLO: 99.9 percent.
- Latency SLO: 95 percent under 300 ms.
- Error budget remaining: 20 percent.

If error budget is low, rollout should be more conservative or blocked.

### Failure Modes

Common failures:

- Metrics delayed, so bad release promotes too quickly.
- Canary traffic too small to detect issue.
- Metrics not segmented by version.
- Rollout controller cannot route traffic as expected.
- Rollback only changes traffic but not bad data.
- Business metrics are ignored.

### Practical Lab Ideas

- Build a staged rollout that pauses for metric checks.
- Add automatic abort on error rate increase.
- Compare absolute threshold vs baseline comparison.
- Combine feature flag rollout with canary traffic.
- Design progressive rollout by tenant rings for a SaaS app.
