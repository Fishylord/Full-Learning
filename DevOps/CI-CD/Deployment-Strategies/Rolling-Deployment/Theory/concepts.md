# Rolling Deployment Theory

## What It Is

Rolling deployment gradually replaces old application instances with new ones.

Instead of stopping everything at once, the platform updates a few instances at a time. Healthy old instances keep serving traffic while new instances start and become ready.

In Kubernetes, rolling update is the default strategy for Deployments.

## How It Works

1. Version 1 is running across multiple instances.
2. Deployment starts version 2 on a small number of instances.
3. Readiness checks confirm the new instances can receive traffic.
4. Some version 1 instances are removed.
5. The process repeats until all instances run version 2.

## Key Concepts

Rolling deployments depend on capacity and health checks.

Important Kubernetes settings:

- `maxUnavailable`: how many old instances can be unavailable during rollout.
- `maxSurge`: how many extra new instances can temporarily exist during rollout.
- `readinessProbe`: whether an instance can receive traffic.
- `livenessProbe`: whether an instance should be restarted.

## Why It Exists

Rolling deployment reduces downtime without requiring a full duplicate environment. It is a practical default for replicated stateless services.

## When To Use

Use rolling deployment for:

- Normal web APIs.
- Stateless services.
- Kubernetes applications.
- ECS services.
- Apps with multiple replicas.
- Low to medium risk releases.

## When To Avoid

Avoid or be careful with rolling deployment when:

- The app has only one instance.
- Old and new versions cannot run together.
- Database migrations are not backward-compatible.
- Message formats change incompatibly.
- Sessions are stored only in local memory.
- The app has long startup time and limited capacity.

## Version Compatibility Problem

Rolling deployment runs old and new versions at the same time. This is its biggest theory point.

During rollout:

- Some users hit old version.
- Some users hit new version.
- Old code and new code may both read/write the same database.
- Old workers and new workers may consume the same queue.
- Old clients may call new APIs or new clients may call old APIs.

Therefore, changes must be backward-compatible.

## Safe Database Pattern

For rolling deployments, use expand-and-contract migrations:

1. Add new schema without breaking old code.
2. Deploy code that supports both old and new schema.
3. Backfill data.
4. Switch application behavior.
5. Remove old schema only after old code is gone.

## Example

A service has 10 replicas. The deployment allows 2 extra replicas and 1 unavailable replica.

The platform starts 2 new replicas. Once they are ready, it terminates old replicas. It repeats this process until all replicas are updated.

## Pros

- Usually no full downtime.
- Efficient infrastructure usage.
- Common default in container platforms.
- Works well for stateless services.
- Easy to automate.

## Cons

- Old and new versions overlap.
- Requires backward compatibility.
- Rollback is gradual, not instant.
- Partial rollout failures can be confusing.
- Needs good readiness checks.

## Short Summary

Rolling deployment updates instances gradually. It is a strong default for replicated services, but it requires old and new versions to safely run together.

## Deep Technical Notes

### Runtime Mechanics

A rolling deployment is a controlled replacement algorithm. The platform creates a limited number of new instances, waits for them to become ready, then removes a limited number of old instances. This repeats until the desired state is reached.

The strategy depends on three things:

- Enough replicas to keep serving traffic.
- Correct readiness detection.
- Compatibility between versions.

If any of these are weak, rolling deployment can produce partial outage, inconsistent behavior, or failed rollout.

### Kubernetes Rolling Update Details

In Kubernetes, rolling behavior is controlled mainly by:

- `replicas`: desired number of pods.
- `maxSurge`: extra pods allowed above desired count.
- `maxUnavailable`: pods allowed to be unavailable during rollout.
- `readinessProbe`: decides whether a pod receives service traffic.
- `progressDeadlineSeconds`: rollout timeout before it is considered failed.
- `minReadySeconds`: how long a pod must stay ready before counted available.

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 0
```

This allows extra capacity during rollout while avoiding intentional capacity loss.

### Readiness Is Not Liveness

A common mistake is using a shallow health check for readiness.

Bad readiness check:

- Process is running.

Better readiness check:

- HTTP server is accepting traffic.
- Required config loaded.
- Database pool initialized.
- Required migrations compatible.
- Cache or queue dependency reachable if critical.
- App can serve the main request path.

Liveness should answer "should this process be restarted?" Readiness should answer "can this process receive user traffic now?"

### Connection Draining

When old instances are removed, the load balancer should stop sending new traffic and allow in-flight requests to complete.

Important settings:

- Kubernetes termination grace period.
- Load balancer deregistration delay.
- Application signal handling.
- WebSocket close behavior.
- Long-running request timeout.

If termination is too aggressive, users see dropped requests even when the rolling rollout appears healthy.

### Version Compatibility

Rolling deployment creates a mixed-version window. During that window:

- Old and new app versions run simultaneously.
- Requests may hit either version.
- Workers may consume messages from either version.
- New code may write data old code later reads.
- Old clients may talk to new servers.

This requires backward-compatible APIs, database schemas, events, and cache formats.

### Message Queue Compatibility

Queues are often overlooked. If new producers create messages with a new schema, old consumers may fail.

Safe patterns:

- Add fields instead of renaming or deleting.
- Make consumers ignore unknown fields.
- Version message schemas.
- Deploy consumers before producers when adding new required data.
- Use dead-letter queues to catch incompatible messages.

### Cache Compatibility

Cache keys and values can break rolling deployments.

Examples:

- New version stores JSON shape old version cannot parse.
- Cache key changes cause stampede.
- Old and new versions overwrite each other's cached values.

Safer patterns:

- Version cache keys.
- Use tolerant parsers.
- Keep TTLs short during rollout.
- Clear or isolate cache only when necessary.

### Autoscaling Interaction

Rolling deployment and autoscaling can interact badly. During rollout, new pods may use more CPU because of startup work. Autoscalers may scale up unexpectedly, then scale down while rollout is still happening.

Watch:

- HPA behavior.
- Cluster capacity.
- Pod scheduling failures.
- Resource requests and limits.
- Startup CPU spikes.

### Rollback

Rollback in a rolling deployment is another rolling deployment back to the previous version. It is not instant.

Rollback is safe only if the newer version did not create incompatible data or irreversible side effects.

Important rollback requirements:

- Previous image digest available.
- Previous config available.
- Database remains compatible.
- New messages can be handled by old consumers or drained separately.

### Observability During Rolling Deployment

Track metrics by version, not only total service metrics.

Useful labels:

- `version`
- `commit_sha`
- `deployment_id`
- `pod_template_hash`
- `region`
- `availability_zone`

Without per-version metrics, a bad new version can be hidden by healthy old instances.

### Practical Lab Ideas

- Configure `maxSurge` and `maxUnavailable` differently and observe behavior.
- Add a bad readiness probe and see traffic route too early.
- Deploy incompatible cache value changes and fix them with versioned keys.
- Roll back after a message schema change and observe consumer failures.
