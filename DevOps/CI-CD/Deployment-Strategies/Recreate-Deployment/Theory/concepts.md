# Recreate Deployment Theory

## What It Is

Recreate deployment is the simplest deployment strategy: stop the old version, then start the new version.

Only one version of the application runs at a time. The old version is removed first, so there is usually downtime between shutdown and startup.

In Kubernetes, this maps to the `Recreate` deployment strategy. Existing pods are terminated before replacement pods are created.

## How It Works

1. Current version is serving traffic.
2. Deployment starts.
3. Old application instances are stopped.
4. New application instances are started.
5. Health checks confirm the new version is available.
6. Traffic resumes.

## Why It Exists

Some applications cannot safely run two versions at the same time. For example, they may use a shared file lock, exclusive database migration, single worker process, or legacy stateful runtime.

Recreate avoids mixed-version behavior because the old and new versions do not overlap.

## When To Use

Use recreate deployment for:

- Development and testing environments.
- Internal tools where downtime is acceptable.
- Small applications with low availability requirements.
- Batch jobs or workers that can be paused.
- Systems that cannot safely run old and new versions together.
- Major maintenance windows where downtime is already planned.

## When To Avoid

Avoid recreate deployment for:

- User-facing production applications.
- High-availability services.
- APIs expected to stay online during releases.
- Systems with strict uptime SLAs.
- Apps where startup takes a long time.

## Requirements

Recreate deployment does not need much infrastructure, but it still needs basic safety:

- Health check after startup.
- Rollback command or previous artifact.
- Maintenance window if downtime matters.
- Clear user communication for planned outage.
- Database backup before risky migrations.

## Main Risks

The main risk is downtime. If the new version fails to start, the system remains unavailable until rollback or manual repair.

Other risks:

- Users lose active sessions or in-flight requests.
- Startup errors are discovered only after old version is gone.
- Rollback may be slow.
- Database migrations may make rollback impossible.

## Example

A small internal admin app runs as one container. During deployment, the old container is stopped and a new container starts from the latest image.

This is acceptable if the team can tolerate a few minutes of downtime and the app is not business critical.

## Pros

- Very simple.
- Easy to understand.
- No old/new version overlap.
- Lower infrastructure cost.
- Useful for legacy or single-instance systems.

## Cons

- Causes downtime.
- Risky if the new version fails to start.
- Poor fit for public production systems.
- No gradual rollout.
- No production traffic testing before full cutover.

## Short Summary

Recreate deployment is stop old, start new. It is simple and avoids version overlap, but it causes downtime and is usually not suitable for high-availability production systems.

## Deep Technical Notes

### Runtime Mechanics

In recreate deployment, the deployment controller intentionally reduces current capacity to zero before creating the new capacity. This means there is a hard cutover in process state. Open HTTP requests, WebSocket connections, background jobs, and in-memory sessions are interrupted unless the application has explicit shutdown handling.

For a server process, the sequence should ideally be:

1. Stop accepting new traffic.
2. Drain existing requests for a bounded grace period.
3. Stop workers or consumers.
4. Flush logs and metrics.
5. Shut down the old process.
6. Start the new process.
7. Run readiness checks.
8. Reattach traffic.

If the platform simply kills the old process without graceful termination, recreate deployment becomes more disruptive than necessary.

### Graceful Shutdown

Even though recreate causes downtime, the old version should still shut down cleanly.

Important shutdown behavior:

- Stop accepting new requests.
- Finish in-flight requests if possible.
- Stop queue polling before terminating workers.
- Commit or roll back open transactions.
- Flush telemetry.
- Close database connections.
- Release distributed locks.

In Kubernetes, this is controlled through `terminationGracePeriodSeconds`, preStop hooks, signal handling, and readiness probes. In systemd or VM deployments, the service process must handle `SIGTERM` or equivalent stop signals.

### Database Migration Concerns

Recreate is sometimes chosen because a database migration cannot support old and new versions at the same time. This does not automatically make the release safe.

Risky migration examples:

- Renaming a column used by the old app.
- Dropping a table.
- Changing enum values.
- Changing stored procedure signatures.
- Rebuilding indexes with long locks.

If the migration runs before the new app starts and the new app fails, rollback may require database rollback, not only app rollback.

Safer recreate deployment with database changes:

1. Take backup or snapshot.
2. Stop old app.
3. Run migration with timeout and verification.
4. Start new app.
5. Run smoke tests.
6. Keep rollback script ready if migration is reversible.

### Traffic Handling

For public traffic, users should receive a controlled maintenance response instead of random connection failures.

Options:

- Put the app behind a maintenance page.
- Return HTTP 503 with clear retry behavior.
- Disable queue consumers but keep read-only endpoints up if possible.
- Use a load balancer rule to route users to a static maintenance page.

If DNS is changed for maintenance mode, remember DNS caching can make behavior inconsistent.

### Stateful Systems

Recreate is more common for stateful or single-writer systems, but state makes rollback harder.

Examples:

- One scheduler process.
- One background consumer group.
- Legacy app with local disk state.
- Monolith requiring exclusive migration.
- App with in-memory session state.

For local state, verify whether the new version can read old files, locks, cache entries, and temporary data.

### Rollback Mechanics

Rollback for recreate deployment means stopping the new version and starting the old version. The dangerous part is not process rollback; it is data compatibility.

Rollback is safe only if:

- The old artifact is available.
- Old config is still available.
- Database schema still supports old code.
- External API contracts did not change irreversibly.
- Queued messages created by the new version can be consumed by the old version.

### Production Hardening Checklist

- Announce or schedule downtime if user-facing.
- Confirm backups before destructive migration.
- Confirm old artifact can be redeployed.
- Confirm health checks detect startup failure.
- Confirm rollback script works.
- Set maintenance response.
- Capture deployment logs.
- Verify after startup with smoke tests.
- Monitor error rate and user traffic after recovery.

### Practical Lab Ideas

- Deploy a single-instance app with recreate strategy.
- Add a slow startup and observe downtime.
- Add graceful shutdown and compare request loss.
- Add a migration that breaks rollback, then redesign it using expand-and-contract.
- Create a maintenance page in front of the app during cutover.
