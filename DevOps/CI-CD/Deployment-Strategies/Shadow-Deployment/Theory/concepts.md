# Shadow Deployment Theory

## What It Is

Shadow deployment sends a copy of real production traffic to a new version while the old version continues serving real users.

The new version processes the copied request, but its response is not returned to the user.

## How It Works

1. Production traffic goes to the current version.
2. A proxy, gateway, or service mesh copies selected requests to the shadow version.
3. The current version returns the real response.
4. The shadow version response is ignored, logged, or compared.
5. Engineers analyze performance, errors, and correctness.

## Why It Exists

Shadow deployment tests a new system with real traffic without exposing users to its behavior.

It is useful when staging traffic is not realistic enough.

## When To Use

Use shadow deployment for:

- API rewrites.
- Framework migrations.
- Service replacement.
- Performance testing.
- New database read paths.
- Request compatibility validation.
- Machine learning inference comparison.

## When To Avoid

Avoid or be careful when:

- Requests cause side effects.
- The new version writes to production systems.
- The system cannot handle duplicate load.
- Sensitive data should not be copied.
- Response comparison is hard or meaningless.

## Side Effect Risk

Side effects are the biggest danger.

Bad shadow behavior:

- Charging a payment twice.
- Sending duplicate emails.
- Creating duplicate orders.
- Publishing duplicate events.
- Writing conflicting database records.

Safer design:

- Make shadow service read-only.
- Use isolated databases.
- Disable outbound side effects.
- Strip mutation requests.
- Use test-mode credentials.

## Comparison Methods

Shadow response comparison can check:

- Status code.
- Response schema.
- Important fields.
- Latency.
- Error count.
- Business calculations.

Do not compare volatile fields such as timestamps, generated IDs, or ordering unless normalized first.

## Example

A team rewrites a legacy API in a new language. Production requests continue going to the legacy API, but copies are sent to the new API. Engineers compare responses and latency until confidence is high enough for canary deployment.

## Pros

- Uses real production traffic.
- No user-facing response risk.
- Good for rewrites and migrations.
- Reveals performance and compatibility issues.
- Can run before canary.

## Cons

- Must prevent side effects.
- Can double traffic load.
- Response comparison can be complex.
- Requires routing infrastructure.
- May expose sensitive data to more systems.

## Short Summary

Shadow deployment tests a new version with copied production traffic while users still receive responses from the old version. It is powerful for migrations, but side effects must be blocked.

## Deep Technical Notes

### Traffic Mirroring Architecture

Shadow deployment usually requires a traffic mirroring component.

Possible components:

- API gateway.
- Reverse proxy.
- Service mesh.
- Load balancer mirror rule.
- Custom middleware.
- Event replay system.

The mirror sends the real request to the primary service and a copied request to the shadow service. The primary response goes to the user. The shadow response is discarded or analyzed.

### Synchronous vs Asynchronous Mirroring

Synchronous mirroring means the shadow request is triggered inside the live request path. This can accidentally affect latency if implemented poorly.

Asynchronous mirroring queues or emits a copy for later processing. This reduces user impact but changes timing and may not perfectly match live dependency state.

For safety, shadow traffic should not block the real user response.

### Mutation Control

The hardest part is preventing side effects.

Dangerous request types:

- POST create order.
- POST charge payment.
- PUT update user.
- DELETE resource.
- Send notification.
- Publish event.

Mitigation strategies:

- Mirror only GET requests.
- Strip or block mutation methods.
- Use shadow credentials with no write permission.
- Route writes to isolated databases.
- Disable outbound integrations.
- Mark shadow requests with headers and enforce no-side-effect behavior.

### Request Identity

Shadow requests should be marked clearly.

Example headers:

- `X-Shadow-Traffic: true`
- `X-Original-Request-Id`
- `X-Shadow-Run-Id`

This helps logs, traces, metrics, and downstream services distinguish shadow behavior.

### Response Comparison

For API migration, compare primary and shadow responses.

Normalize before comparison:

- Remove timestamps.
- Remove generated IDs.
- Sort arrays when order does not matter.
- Ignore fields known to differ.
- Compare schemas separately from exact values.

Track:

- Match rate.
- Schema mismatch rate.
- Status code mismatch.
- Latency difference.
- Error difference.

### Privacy and Compliance

Shadow traffic may copy sensitive production data to another system. This can violate privacy requirements if the shadow environment has weaker controls.

Ensure:

- Same security boundary or approved environment.
- No unnecessary logging of sensitive payloads.
- Data retention policy.
- Access controls.
- Redaction where required.

### Capacity Planning

Shadowing 100 percent of traffic can double backend load. Start smaller.

Options:

- Mirror only 1 percent.
- Mirror specific routes.
- Mirror only read traffic.
- Sample requests.
- Replay captured traffic in a controlled environment.

### Practical Lab Ideas

- Build a proxy that forwards real responses from service A and mirrors to service B.
- Add a header to mark shadow requests.
- Block mutation methods in the shadow path.
- Compare JSON responses after normalizing timestamps.
- Measure load increase from shadow traffic.
